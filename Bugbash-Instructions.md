# Bug bash: `azd ai eval` and `azd ai dataset`

Two prerelease `azd` extensions for Foundry evaluations. Everything here runs
against a **real, shared Foundry project** -- these commands create datasets,
evaluators, evals and runs in it, and cost real model calls.

| Extension | Namespace | PR |
|---|---|---|
| `azure.ai.evaluations` | `azd ai eval` | [Azure/azure-dev#9500](https://github.com/Azure/azure-dev/pull/9500) |
| `azure.ai.dataset` | `azd ai dataset` | [Azure/azure-dev#9499](https://github.com/Azure/azure-dev/pull/9499) |

**File findings as bugs, not PR comments: https://aka.ms/evalsbug**
Include your OS, `azd version`, the exact command and its full output.

---

## Quick start

You need `azd` 1.27.1 or later, and `az login` + `azd auth login` done.

Everywhere below, replace `<you>` with your alias. **Names must be unique** --
the project is shared and evals persist, so reusing a name someone else used
leaves you unable to run it. See [Appendix A](#appendix-a-known-issues).

```bash
# 1. install the extensions
azd extension source add -n foundry-bugbash -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-08-24-20/registry.json
azd extension install azure.ai.evaluations --source foundry-bugbash
azd extension install azure.ai.dataset --source foundry-bugbash

# 2. point at the shared project (PowerShell; other shells in Notes)
$env:FOUNDRY_PROJECT_ENDPOINT="https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral"

# 3. make a project to work in
mkdir azd-eval-bugbash
cd azd-eval-bugbash
azd init --minimal --no-prompt -e bugbash

# 4. your first evaluation
azd ai eval init --source traces --target support-agent --judge-model gpt-4.1-nano --name <you>-trace-eval --no-prompt
azd ai eval create
azd ai eval run start
```

That evaluates the traces `support-agent` has already produced. A first run of
around 20 samples usually takes a few minutes -- roughly half a minute per
sample, since every one of them is a model call -- and ends with a table of
evaluators and an overall pass rate. If it finishes in seconds, or is still
going after fifteen minutes, that is worth reporting.

Then read the per-sample results:

```bash
azd ai eval run output list --eval <you>-trace-eval
```

**Check you are current:** `azd extension list --installed` should show
`azure.ai.evaluations` **1.0.24-beta** and `azure.ai.dataset` **1.0.0-beta.17**,
both from `foundry-bugbash`. If not, `azd extension upgrade <id>`.

If you took part in an earlier round, the feed URL above is new. Point the
source at it again -- `azd extension source remove foundry-bugbash` then the
`add` above -- or `azd extension upgrade` will keep offering you the old build.

### Fixed since the last round

Please re-test these and reopen if any is still wrong.

- **Deleting asks first.** `eval delete`, `eval dataset delete`, `eval
  evaluator delete` and `run delete` removed the data the moment you pressed
  enter, while `azd ai dataset delete` in the other extension asked. They all
  ask now, and all take `--force`. **Scripts and `-o json` need `--force`**
  -- a question nobody can answer is refused rather than guessed at, so the
  cleanup commands below have changed.
- **A listing that breaks half way no longer looks like a missing evaluator.**
  If page two of a version listing failed, `create` concluded nothing was
  published and published over the rubric, reporting success. Worth a look if
  you ever found a rubric edit had vanished.
- **`azd up` accepts the `.jsonl` files `dataset create` accepts.** A file
  written by PowerShell (`>` or `Set-Content`) starts with a byte order
  mark. Upload skipped it and validation did not, so one command took the file
  and the other pointed at row 1 and called it broken.
- **`init --max-traces <n>` works without `--source traces`** in a project
  wired for traces. It was refused a line before the default was going to
  choose traces anyway.
- **A rubric you cannot record is refused before the generation job runs**, not
  after it has been billed and written.
- **Answering `no` to a delete exits 0.** It came back as an error, so a
  deliberate `no` looked to any script exactly like a delete that broke.
- **A listing the service cannot finish is an error rather than a short list.**
  A repeated page link used to leave you a partial catalog that looked
  complete -- and those rows are what pick the latest version.
- **Two `generate`s at once no longer lose one another's entry.** A lock that
  could not be taken was reported and the work went ahead anyway, which is
  the lost update the lock exists to stop. It now fails and tells you to
  wait and retry.
- **A dataset whose file sits on a later page of the container is found.**
  The blob listing stopped early and reported success, so a generated
  dataset could resolve to the wrong file, or to none at all.
- **A second `init` in the same project works.** It failed outright with
  `mkdir evals/azure.eval.yaml`, which is the flow the ideas list asks you
  to try.
- **The README's command table names only commands that exist.** It
  advertised `evaluator upload`, `evaluator builtins` and an
  `azd ai eval results` group, none of which are real, and left out the
  job group, version listings and `run output` entirely.

### Fixed in the round before

- **`generate` and `init` no longer rewrite your file.** They read the
  configuration, changed a field and wrote the whole thing back, which deleted
  every comment you had written and changed the indentation. Only the entries
  they add are written now. If you kept notes in `azure.eval.yaml` and lost
  them, that is why, and it should not happen again.
- **`$ref` works on an eval's `source:`, its `target:`, and the items of its
  `evaluators:` list.** Those deployed, then failed the moment `generate` or
  `init` read the same file. The known gap from the last round is closed.
- **A deploy that cannot find the project directory now says so** instead of
  resolving every relative path against whatever directory you started in,
  which could publish a same-named dataset from the wrong place.
- **A listing the service cannot finish is now an error**, not a short answer.
  A truncated catalog looked identical to a complete one, and those rows decide
  whether a name is ambiguous.

### Fixed in earlier rounds

- **An evaluator that carries its rubric is published.** Both publish paths
  selected on `source:` alone, and a rubric written under `definition:` leaves
  `source:` empty, so it was skipped without a word.
- **`generate` no longer corrupts an entry it cannot edit.** An entry reached
  through `$ref`, one already carrying its rubric under `definition:`, and one
  pinned to a `version:` are each refused with an explanation.
- **A `$ref` on one entry no longer changes what another entry means.** A
  directive on an unrelated dataset used to turn a mistyped `dimensions:` into
  rubric content and publish it.
- **`$ref` works on datasets and evals, not just evaluators.**
- **`azd up` from a subdirectory** no longer reports every dataset as not yet
  generated.
- **`--version` means the version to publish.** `azd ai dataset update
  --version 4.0` used to publish **5.0**. It publishes 4.0. The flag works on
  `create` too, and a version that already exists is refused rather than bumped.
- **An evaluator can carry its rubric.** A `$ref` to a rubric file, or a
  `definition:` block written in place, now publishes. It silently published
  nothing before, and the eval then scored against an evaluator the service had
  never been told about.
- **`$ref` means the same thing everywhere.** A configuration that deployed with
  `azd up` but failed every `azd ai eval` command, or the reverse, should not
  happen now. YAML anchors (`&judge` / `*judge`) survive resolution.
- **Editor validation.** `azure.yaml` and `azure.eval.yaml` are described by a
  published schema, so a mistyped key is underlined as you type.
- **Pass rate counts only the rows that were scored.** Errored rows are reported
  separately instead of counting as failures. `--fail-on any-failure` still
  counts them against the run.

### One thing that is not a bug

A fresh clone republishes every dataset and evaluator on its first deploy.
Version identity lives in the azd environment, which is not in the repository,
so a new clone cannot know what is already registered. Noisy, not wrong, and
already tracked.

## Scenarios

These are **examples, not a script**. Work through them to get oriented, then go
wherever you like -- the most useful findings come from things nobody wrote down.

Quick start above is scenario 1. Give it its own folder. Scenarios 2 to 6 share
a second folder, because 3 onwards read what 2 writes. Scaffold it the same way:

```bash
mkdir azd-eval-bugbash-2
cd azd-eval-bugbash-2
azd init --minimal --no-prompt -e bugbash
```

### 2. A repeatable baseline

Generate a dataset and a rubric evaluator, then declare an eval over them.

```bash
azd ai eval generate --from agent --target support-agent --generation-model gpt-4.1-nano --dataset-name <you>-regression --evaluator-name <you>-quality
azd ai eval init --name <you>-regression-eval --dataset <you>-regression --target support-agent --judge-model gpt-4.1-nano --evaluator builtin.task_adherence --evaluator <you>-quality --no-prompt
azd ai eval create
azd ai eval run start
```

**Expect:** `generate` submits two jobs, downloads both artifacts into `evals/`,
and registers them in Foundry as it goes. So the `create` that follows reports
the dataset and evaluator as already **unchanged** and only creates the eval --
that is correct, not a missed publish. A second `create` with no edits skips
everything, the eval included.

Then open one sample. This is the only place a rubric's per-dimension scores and
the judge's full reasons are visible; the listing truncates them to a cell.

```bash
azd ai eval run output list --eval <you>-regression-eval
azd ai eval run output show <item-id> --eval <you>-regression-eval
```

### 3. Inner loop

First re-run unchanged. Nothing changed, so nothing should be republished.

```bash
azd ai eval create
azd ai eval run start
```

Now change **what is being evaluated** -- not how it is judged, which is
scenario 4 -- and run it again. That is the loop this scenario exists for, and
it is what makes the comparison below mean something. Edit the agent's
instructions in the Foundry portal, then:

```bash
azd ai eval create
azd ai eval run start
azd ai eval run list --eval <you>-regression-eval
```

**Expect:** the first `create` skips everything; the second publishes only what
you edited. `run list` shows the runs side by side with their pass rates, so you
can see quality move.

### 4. Tuning the evaluation

Edit a dimension's description in `evals/evaluators/<you>-quality.json`, then:

```bash
azd ai eval create
azd ai eval evaluator versions list <you>-quality
```

**Expect:** the evaluator gains version 2 and the eval is left alone -- a rubric
edit must not split the run history. A further `create` with no edits skips
everything.

Then edit the same evaluator **in the Foundry portal** and run `create` again.

**Expect:** it stops rather than publishing over the portal's version, saying
the remote version is ahead of the one this environment deployed. Publishing
anyway would overwrite an edit nobody in the repo can see.

### 5. Automation and CI/CD

Add a second eval named `<you>-gate` to `evals/azure.eval.yaml`. **Give it
something of its own** -- a different dataset, evaluator or sample cap. An eval
copied from the first with only the name changed is refused, on the grounds
that two identical evals are almost always a copy-paste slip. Then:

```bash
azd ai eval create <you>-gate
azd ai eval run start --eval <you>-gate --no-prompt --fail-on pass-rate=0.8
azd ai eval run start --eval <you>-gate --no-prompt --fail-on any-failure
```

Name the eval on `create`: once the file declares more than one, a bare
`azd ai eval create` refuses rather than guessing. `--no-prompt` is what a
pipeline passes, and this scenario is the one that should be exercised the way a
pipeline would run it.

Start without blocking, take the run id out of the JSON, then reattach and
export:

```bash
azd ai eval run start --eval <you>-gate --no-prompt --no-wait -o json
# the run_id field of that JSON is what the next two commands take
azd ai eval run show <run-id> --eval <you>-gate --no-prompt --wait --fail-on pass-rate=0.8
azd ai eval run output export <run-id> --eval <you>-gate --output-file results.csv
```

**Expect:** `--no-wait -o json` returns as soon as the run is accepted, carrying
`run_id`, `eval_id`, `eval_name`, `dataset`, `dataset_version`, `status` and
`created_at`. A breached gate exits non-zero; a completed run with failing
samples exits 0 unless you asked for a gate.

### 6. Deploying evals as an azd service

The extension registers an `azure.ai.eval` **service target**, which is what runs
when `azd` deploys the service `init` added to `azure.yaml`. Give the environment
a subscription and location of your choosing, then:

```bash
azd env set AZURE_SUBSCRIPTION_ID <your-subscription-id>
azd env set AZURE_LOCATION <your-region>
azd deploy
```

**Expect:** the same reconciliation `create` does, ending in `SUCCESS`. No Azure
infrastructure is provisioned -- eval resources are data-plane only. While it
runs, progress lines say what happened; they are replaced by the service table
when it finishes, so watch as it goes rather than reading the summary:

```
support-agent-evals: Deploying (Created eval <you>-reg-eval (eval_...))
support-agent-evals: Deploying (Eval <you>-reg-eval is unchanged (eval_...))
```

`azd ai eval create` afterwards should agree, and prints the same detail without
the hurry. A disagreement is a finding.

### 7. `azd up` in a project that has infrastructure

`azd up` is the right verb only where the project really provisions something.
In a folder with an `infra/` template that creates at least one resource, `azd
up` provisions and then runs the same eval service target. Worth a pass if you
already have such a project; `azd ai eval init` should recommend `azd up` there
rather than `azd ai eval create`.

### More to try

Short prompts, no commands -- improvise.

- **Empty state:** every `list` in a new project; no endpoint configured; no azd
  environment; a config file that is missing, empty, or has a misspelled key.
- **Bad input:** names with spaces, unicode, slashes or `..`; a `--from-file`
  directory with several `.jsonl` files, or none; an empty `.jsonl` and one with
  a malformed row; a file saved by Notepad (UTF-8 with BOM).
- **Config edits:** an evaluator needing a column the dataset lacks; a dataset
  pinned to a deleted version; renaming an eval; editing only a description.
- **Somewhere other than `evals/`:** `init --path ./quality`, then run the rest
  without `--path`; a `--path` that is absolute, nested, or already holds a
  config; two `init`s in one project.
- **More than one environment:** `azd env new`, give each a different
  `FOUNDRY_PROJECT_ENDPOINT`, then run with `-e` and check the right project
  was touched and the right environment recorded the ids
  (`azd env get-values -e <name>`).
- **Scripting:** `-o json` everywhere including failures; `--output-file` at a
  directory, a read-only path, a deep path; very long names; narrow terminals.
- **Interruption:** Ctrl-C mid-`create` and mid-run, then re-run; two `create`s
  at once; `--no-wait`, cancel, then ask for output.
- **Cross-extension:** `azd ai dataset list` vs `azd ai eval dataset list`, and
  anything else in both. They should answer the same way, exit codes included.

---

## Command reference

**`azd ai eval`** -- `init`, `generate`, `create [name]`, `list`, `show <eval>`,
`delete <eval>`

- `create [name]` takes the name as an argument, not a flag. `--from-file`
  creates from a file instead of a project.

**`azd ai eval dataset`** and **`azd ai eval evaluator`** -- `create`, `update`,
`list`, `show`, `delete`, `versions list`

- `evaluator list --builtin` is how you discover the `builtin.*` names.

**`azd ai eval run`** -- `start`, `show`, `list`, `cancel`, `delete`, and
`output list|show|export`

- `run list` shows one pass rate per run. The per-evaluator breakdown is in
  `-o json`, under `per_testing_criteria_results`, because a column per
  evaluator stops being readable once two runs score different ones.

**`azd ai eval job`** -- `list`, `show`, `cancel`, `delete` for generation jobs.
`--dataset` and `--evaluator` are switches choosing which kind of job to act
on, not filters taking a name. One is required, so a bare `job list` fails.

**`azd ai dataset`** -- `create`, `update`, `list`, `show`, `delete`,
`versions list`

Every command takes `-o json`, `--debug` and `--help`. All except `init` also
take `--project-endpoint`, which wins over the environment variable.

---

## Cleanup

Artifacts are not removed by uninstalling. Delete evals by **id** -- `azd ai eval
list -o json` gives you them -- because a name shared by more than one eval is
refused. Deleting an eval also discards its runs.

```bash
azd ai eval delete <eval-id> --force
azd ai eval dataset delete <name> --version <version> --force
azd ai eval evaluator delete <name> --version <version> --force
```

`--force` is what skips the confirmation. Without it these ask, and under
`--no-prompt` or `-o json` they refuse rather than assume.

To remove the extensions and the feed:

```bash
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension source remove foundry-bugbash
```

---

## Notes

**Other shells.** Command Prompt takes no quotes and no spaces around `=`:
`set FOUNDRY_PROJECT_ENDPOINT=https://...`. bash and zsh use
`export FOUNDRY_PROJECT_ENDPOINT=https://...`.

**Check the endpoint took:** `azd ai eval list -o json` should print `[` and
exit 0, even in an empty project.

**What is already in the project.** Agents `support-agent` (used by the
scenarios) and `test-agent`. Models `gpt-4.1-nano`, `gpt-4o-mini`, `gpt-4.1`,
`gpt-5.1`, `o4-mini`, `text-embedding-3-large`. Region swedencentral. Access is
via the Evaluation Service Team group (`raisvcteam@microsoft.com`).

**Install can look stuck.** Each extension is about 15 MB from a GitHub release,
so the progress bar may sit still for a few minutes on first install. Let it
finish.

**`--source` matters.** Without it, `azd` may find the id in more than one
registry and stop on a prompt, which hangs a script. `azd extension upgrade`
has no `--source` flag at all, so if it asks, answer `foundry-bugbash`; to
avoid the question entirely, `azd extension uninstall <id>` then install again
with `--source`.

**`azd init` prompts.** Always use `azd init --minimal --no-prompt -e <name>`;
plain `azd init` and even `--minimal` ask questions.

**There is a second eval surface, and it is not this one.** The agents extension
ships `azd ai agent eval`, with its own config at `eval.yaml` in the project
root, while this extension writes `evals/azure.eval.yaml`. They are not
interchangeable and neither reads the other's file. `azd ai agent eval` is one
agent, one command, sensible defaults; `azd ai eval` declares datasets,
evaluators and evals in a config you keep in the repo and reconcile. Which to
use is worth an opinion from you, and confusion between them is a legitimate
finding.

---

## Appendix A: known issues

Already reported. Please don't re-file these; anything else is fair game.

1. **Names must be unique.** Two evals with the same name cannot be run: `run
   start` refuses to guess, and the id it offers is rejected by
   `run start --eval <id>` until that eval has already run once. The two errors
   point at each other. Prefix everything with your alias. If you are stuck,
   `azd ai eval list -o json` then `azd ai eval delete <eval-id> --force`.

2. **`--judge-model` and `--generation-model` read as optional but are
   effectively required** against a shared project reached by endpoint, because
   there is no `azure.yaml` model deployment to detect. `init` says so rather
   than writing an undeployable config. It now also reads
   `AZURE_AI_MODEL_DEPLOYMENT_NAME` from the azd environment, which the bug bash
   flow does not set, so pass the flag here. (Bug 5511012.)

3. **Agent-seeded generation fails server-side for every agent.** `generate`
   detects it, says so, and retries from the agent's instructions alone, which
   succeeds. Expected until the service is fixed.

4. **`-o json` still prints prose on the failure path**, so a failing command
   breaks a JSON pipe. The `ERROR:` line comes from `azd` itself, not these
   extensions.

5. **`WARNING: 1 extension did not start.`** That is `azure.ai.agents`, not
   either extension under test. It and `azure.ai.projects` both try to register
   the same provisioning provider and the second loses. Harmless here.

6. **A gated run exits 1, not 2.** `azd` does not propagate an extension's exit
   code, so gating and operational failure share 1. Tell them apart by the gate
   message. Non-zero vs zero is still correct.
