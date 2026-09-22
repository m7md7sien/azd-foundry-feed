# Build 37 — verification pass

**Published 2026-09-22. Historical build checklist, not a readiness certificate.**
This build bundles changes intended to address findings from the last bug bash.
There is no blanket claim that 13 bugs are fixed or verified. The job here is
narrower than [Bugbash-Instructions.md](./Bugbash-Instructions.md): confirm the
behavior against a live Foundry project and record the exact installed versions.
Evidence from another source tree or a later candidate does not verify this build.

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

To reproduce this historical build, use its immutable release URL, not the rolling
Latest source (which will eventually resolve a newer candidate). These commands
replace the two installed extensions in the current azd configuration:

```bash
azd extension source add -n foundry-bugbash-37 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-22-37/registry.json
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension install azure.ai.evaluations --source foundry-bugbash-37
azd extension install azure.ai.dataset --source foundry-bugbash-37
```

Confirm you are actually on this build — read `installedVersion`, not `version`:

```bash
azd extension list --installed
azd ai eval version             # must print 1.0.37-beta
azd ai dataset version          # must print 1.0.0-beta.25
```

Skip an uninstall command if that extension is not installed. For the rolling
candidate rather than build 37, use the Latest instructions in the
[README](./README.md). Always name the desired source explicitly.

---

## 2. What is in this build

Three pull requests were **bundled into the feed build**, not asserted to be merged
into upstream `main`. The release description does not identify a single source
commit or a matching CI run for the shipped archives. Do not infer those from a
feed tag's target commit, which identifies this feed repository, not the source
repository. Source-only checks and later baseline tests are separate evidence.

| PR | Carries |
|---|---|
| [Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116) | Conversation simulation, plus five run/generate correctness fixes |
| [Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113) | `init`/`publish` input validation, evaluator catalog metadata, cancellation |
| [Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102) | Usage telemetry from both extensions |

---

## 3. Bugs to verify

Each row is a verification target, not a recorded pass. Confirm **Expected** with
these installed versions and record pass, failure, or a specific service blocker.

### Generate and dataset

| Bug | What to run | Expected |
|---|---|---|
| **5631329** | `azd ai eval generate --dataset --evaluation-level conversation --from prompt --agent-instruction "..." --generation-model <model> --max-samples 15` | Rows are **seeds** (`test_case_description`, optional `desired_num_turns`), not query/response pairs. The job's `options.type` is `simulation_seed`. |
| **5631288** | The same, with `--target <service-key>` where the `azure.yaml` key differs from the deployed agent name | Seeds are generated from the **deployed agent**, not from the local service key |
| **5631478** | Declare an eval with a `simulation:` block over a seed dataset, `azd ai eval create`, then `azd ai eval run start` | A conversation simulation completes and scores its transcript. Merely accepting the request does not verify this. |
| **5631468** | Run an agent-target eval over a **registered** dataset, uncapped and capped separately | An uncapped run uses the service-issued identity when available. A capped run intentionally sends bounded rows inline; `Inline data` is expected in that case. Verify the row count and selected dataset version. |

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
| **5631330** | A service-side fix has merged, but deployment to the test project is unknown. Recheck the actual count; do not mark fixed from merge status alone. |
| **5595119** | No extension change is claimed here. Record the current service outcome separately. |

---

## 4. Historical observations and external gates

The observations below were reported during earlier source/live investigations.
They explain the contract but are not a per-scenario verification record for
every archive in this release or for a future candidate.

**The seed-generation request uses `simulation_seed`.** This has contract and
successful baseline job evidence. The simulation *run* uses
`azure_ai_user_conversation_simulation_preview`, a separate discriminator.
The final GA discriminator remains unconfirmed.

**The request and the dataset tag deliberately use different words.** The service accepts
`options.type: simulation_seed` on the way in, and then writes
`data_generation_type: conversation_simulation` onto the dataset version it produces. Both
are correct in their own place. **If you see `conversation_simulation` in dataset tags,
that is expected, not a regression.**

**Rubric metadata preservation has baseline live evidence.** A rubric edit
published a new version retaining `display_name`, `categories` and
`supported_evaluation_levels`. Repeat the edit/read-back on the candidate before
marking its `5530209` scenario verified.

**Two undocumented service constraints turned up while confirming that:**

| Constraint | Accepted | On violation |
|---|---|---|
| `categories` is a **closed enum** | `quality`, `safety`, `agents`, `business` | HTTP 400 |
| `pass_threshold` is **normalized** | `0.0` to `1.0` | HTTP 400 — e.g. `3` is rejected |

Use these constraints when authoring an evaluator; validate the returned metadata
as well as the status code. Official release readiness also remains gated on
privacy signoff. This unofficial feed is not an official release approval.

---

## 5. Known rough edges to distinguish from regressions

- **Generation count is not a run cap.** An earlier generation request for 15
  returned 14 rows. Generation is service-dependent and may return fewer rows;
  record overshoot rather than assuming a service fix is deployed. A dataset run's
  `--max-samples` bounds the input rows and is a separate behavior.
- **Standalone `--no-wait` prints a state warning.** Run outside an `azd` project you will
  see `cannot record EVAL_FINGERPRINT_DATA_JOB_..._LEVEL: no project exists`. Harmless —
  the level is recovered from the service when you reattach, which is exactly the case
  that recovery exists for.

---

## 6. Reporting back

File findings as bugs, not PR comments: **<https://aka.ms/evalsbug>**

For each bug above, report one of **verified**, **still broken**, or **blocked, and why**.
If something is still broken, include your OS, `azd version`, the exact command,
sanitized output, and the job or run id. Do not post tokens, private prompts,
customer rows, or full raw service responses. Include the release tag, both
extension versions, and the exact source SHA when the release provides it.
