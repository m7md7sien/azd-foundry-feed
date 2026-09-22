# Build 37 — verification pass

**Published 2026-09-22.** This build is installable from the feed and bundles fixes for
13 bugs found in the last bug bash. The job here is narrower than
[Bugbash-Instructions.md](./Bugbash-Instructions.md): confirm each fix actually holds
against a live Foundry project, and record what you find.

Start from the hero scenarios if you want breadth. Use this doc if you want to close out
specific bugs.

| Extension | Version in this build |
|---|---|
| `azure.ai.evaluations` | `1.0.37-beta` |
| `azure.ai.dataset` | `1.0.0-beta.25` |

Release: [extensions-2026-09-22-37](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-22-37)

---

## 1. Install

**Hard prerequisite: `azd >= 1.33.0`.** Both extensions publish
`requiredAzdVersion: >=1.33.0`, so an older `azd` will not resolve this build and reports
that no compatible version was found. That is the constraint working as intended, not a
bug — do not file it.

```bash
azd version                     # must be >= 1.33.0
```

If it is older, reinstall `azd` first — see
[Install the Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd).

The feed source is stable and does not change per build:

```bash
azd extension source add -n foundry-bugbash -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/latest/download/registry.json
azd extension upgrade azure.ai.evaluations
azd extension upgrade azure.ai.dataset
```

Confirm you are actually on this build — read `installedVersion`, not `version`:

```bash
azd extension list --installed
azd ai eval version             # must print 1.0.37-beta
azd ai dataset version          # must print 1.0.0-beta.25
```

**Returning testers:** if you still have sources pinned to dated releases (`bugbash30`,
`bugbash31`, or a `foundry-bugbash` pointing at a dated URL), they will surface older
versions alongside this one. Remove the stale ones with
`azd extension source delete <name>` and keep only the `latest/download` source above.

---

## 2. What is in this build

Three pull requests, merged onto `main` and packed together. All three pass the same gate
as a combined tree — `gofmt`, `go fix`, `go vet`, tests, `golangci-lint`, `cspell`.

| PR | Carries |
|---|---|
| [Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116) | Conversation simulation, plus five run/generate correctness fixes |
| [Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113) | `init`/`publish` input validation, evaluator catalog metadata, cancellation |
| [Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102) | Usage telemetry from both extensions |

---

## 3. Bugs to verify

Each row is a fix this build claims. Reproduce the old behaviour only if you have an
older build handy — otherwise just confirm **Expected**.

### Generate and dataset

| Bug | What to run | Expected |
|---|---|---|
| **5631329** | `azd ai eval generate --dataset --evaluation-level conversation --from prompt --agent-instruction "..." --generation-model <model> --max-samples 15` | Rows are **seeds** (`test_case_description`, optional `desired_num_turns`), not query/response pairs. The job's `options.type` is `simulation_seed`. |
| **5631288** | The same, with `--target <service-key>` where the `azure.yaml` key differs from the deployed agent name | Seeds are generated from the **deployed agent**, not from the local service key |
| **5631478** | Declare an eval with a `simulation:` block over a seed dataset, `azd ai eval create`, then `azd ai eval run start` | A conversation simulation actually runs. Before this build there was no run path at all. |
| **5631468** | Run an agent-target eval over a **registered** dataset | The request references the dataset by id and version. Foundry must **not** show `Inline data`. |

### Run

| Bug | What to run | Expected |
|---|---|---|
| **5631335** | An agent-target eval at conversation level over seed rows, with no `query` column | **Refused before the run starts**, naming the eval and the missing column. Must not invoke the agent with an empty `{{item.query}}`. |
| **5631281** | A conversation eval over traces | `evaluation_level` reaches the service; rows are conversation-shaped, not turn-shaped |
| **5595070** | Let a run fail or error some rows, then read the summary | The summary names failing and errored rows and prints runnable follow-up commands (`run output list --eval ... --run ...`, `run output export ... --output-file ...`) |

### Init and publish

| Bug | What to run | Expected |
|---|---|---|
| **5631311** | `azd ai eval init --dataset ./broken.jsonl` with a malformed line, an empty file, an array row, an empty object, or a bare scalar | **Exit 1**, a precise file-and-line message, and **nothing written** — no `azure.eval.yaml`, no `azure.yaml` edit. Fixing the file and rerunning must succeed. |
| **5631310** | `azd ai eval init --evaluator builtin.does_not_exist` against a **reachable** project | Refused, and the error lists what the project actually offers. If the project is **unreachable**, the reference is left as written and the command exits 0. |
| **5530209** | Publish an evaluator, edit the rubric, then `azd up` again | The new version **keeps** `display_name`, `categories` and `supported_evaluation_levels`. Support must not narrow from `[turn, conversation]` to `[turn]`. |
| **5571804** | `azd ai eval create`, both when something is created and when nothing changed | Output ends with the Portal link |
| **5571322** | `azd ai eval create` or `run show` with two or more evals declared, then **close the picker** | Prints `Cancelled. No eval was selected.` and **exits 0** — no ambiguity error. Under `-o json`, stdout stays parseable, with no prose mixed in. |

### Service-side

| Bug | Note |
|---|---|
| **5631330** | `max_samples` overshoot. Fixed service-side, **not** in this extension build. |
| **5595119** | Needs no change in these extensions. |

---

## 4. Already settled — please do not re-litigate

These were confirmed against a live project while the build was being put together.
Re-test if you want, but the conclusions below are load-bearing, and at least two of them
look like bugs if you do not know the backstory.

**The data-generation discriminator is `simulation_seed`.** Confirmed twice over: in the
published contract (`azure-rest-api-specs`, `data_generation_jobs/models.tsp`,
`DataGenerationJobType`) and by a live job that ran to `succeeded`.
`conversation_simulation` is **not** a member of that enum.

**The request and the dataset tag deliberately use different words.** The service accepts
`options.type: simulation_seed` on the way in, and then writes
`data_generation_type: conversation_simulation` onto the dataset version it produces. Both
are correct in their own place. **If you see `conversation_simulation` in dataset tags,
that is expected, not a regression.**

**The `5530209` wire shape is confirmed.** A genuine rubric edit published **version 2,
HTTP 201**, retaining `display_name`, `categories` and `supported_evaluation_levels`.

**Two undocumented service constraints turned up while confirming that:**

| Constraint | Accepted | On violation |
|---|---|---|
| `categories` is a **closed enum** | `quality`, `safety`, `agents`, `business` | HTTP 400 |
| `pass_threshold` is **normalized** | `0.0` to `1.0` | HTTP 400 — e.g. `3` is rejected |

Nothing user-facing in either extension violates these, but they are easy to trip over
when hand-writing a request.

---

## 5. Known rough edges — expected, not bugs to file

- **`--max-samples` is approximate.** A request for 15 returned **14** rows. The service
  treats it as an upper bound and best effort, not an exact count. Worth a doc-wording
  fix; please do **not** file it as data loss.
- **Standalone `--no-wait` prints a state warning.** Run outside an `azd` project you will
  see `cannot record EVAL_FINGERPRINT_DATA_JOB_..._LEVEL: no project exists`. Harmless —
  the level is recovered from the service when you reattach, which is exactly the case
  that recovery exists for.

---

## 6. Reporting back

File findings as bugs, not PR comments: **<https://aka.ms/evalsbug>**

For each bug above, report one of **verified**, **still broken**, or **blocked, and why**.
If something is still broken, include your OS, `azd version`, the exact command, the full
output, and the job or run id — those are what make a service-side claim checkable. One
live run beats an argument from reading the source.
