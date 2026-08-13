# Bug bash: `azd ai eval` and `azd ai dataset`

Two prerelease `azd` extensions for Foundry evaluations. Everything here runs
against a **real, shared Foundry project** — these commands create datasets,
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

Everywhere below, replace `<you>` with your alias. **Names must be unique** —
the project is shared and evals persist, so reusing a name someone else used
leaves you unable to run it. See [Appendix A](#appendix-a-known-issues).

```bash
# 1. install the extensions
azd extension source add -n foundry-bugbash -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-08-10/registry.json
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

That evaluates the traces `support-agent` has already produced. A healthy first
run is about 20 samples in a minute, ending with a table of evaluators and an
overall pass rate.

Then read the per-sample results:

```bash
azd ai eval run output list --eval <you>-trace-eval
```

**Check you are current:** `azd extension list --installed` should show
`azure.ai.evaluations` **1.0.4-beta** and `azure.ai.dataset` **1.0.0-beta.6**,
both from `foundry-bugbash`. If not, `azd extension upgrade <id>`.

---

## Scenarios

These are **examples, not a script**. Work through them to get oriented, then go
wherever you like — the most useful findings come from things nobody wrote down.

Quick start above is scenario 1. Give it its own folder. Scenarios 2 to 6 share
a second folder, because 3 onwards read what 2 writes.

### 2. A repeatable baseline

Generate a dataset and a rubric evaluator, then declare an eval over them.

```bash
azd ai eval generate --from agent --target support-agent --generation-model gpt-4.1-nano --dataset-name <you>-regression --evaluator-name <you>-quality
azd ai eval init --name <you>-regression-eval --dataset <you>-regression --target support-agent --judge-model gpt-4.1-nano --evaluator builtin.task_adherence --evaluator <you>-quality --no-prompt
azd ai eval create
azd ai eval run start
```

**Expect:** `generate` submits two jobs and downloads both artifacts into
`evals/`. A second `azd ai eval create` with no edits publishes nothing and says
so per artifact.

### 3. Inner loop

Re-run and compare. Nothing changed, so nothing should be republished.

```bash
azd ai eval create
azd ai eval run start
azd ai eval run list --eval <you>-regression-eval
```

**Expect:** `create` skips everything; `run list` shows both runs side by side
with their pass rates, so you can see quality move.

### 4. Tuning the evaluation

Edit a dimension's description in `evals/evaluators/<you>-quality.json`, then:

```bash
azd ai eval create
azd ai eval evaluator versions list <you>-quality
```

**Expect:** the evaluator gains version 2 and the eval is left alone — a rubric
edit must not split the run history. A further `create` with no edits skips
everything.

### 5. Automation and CI/CD

Add a second eval named `<you>-gate` to `evals/azure.eval.yaml`, then:

```bash
azd ai eval create <you>-gate
azd ai eval run start --eval <you>-gate --fail-on pass-rate=0.8
azd ai eval run start --eval <you>-gate --fail-on any-failure
```

Name the eval on `create`: once the file declares more than one, a bare
`azd ai eval create` refuses rather than guessing.

Start without blocking, then reattach and export:

```bash
azd ai eval run start --eval <you>-gate --no-wait
azd ai eval run show <run-id> --eval <you>-gate --wait --fail-on pass-rate=0.8
azd ai eval run output export <run-id> --eval <you>-gate --output-file results.csv
```

**Expect:** a breached gate exits non-zero; a completed run with failing samples
exits 0 unless you asked for a gate.

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
infrastructure is provisioned — eval resources are data-plane only. The progress
lines say what happened, and a second `azd deploy` publishes nothing:

```
support-agent-evals: Deploying (Created eval <you>-reg-eval (eval_...))
support-agent-evals: Deploying (Eval <you>-reg-eval is unchanged (eval_...))
```

`azd ai eval create` afterwards should agree. A disagreement is a finding.

### 7. `azd up` in a project that has infrastructure

`azd up` is the right verb only where the project really provisions something.
In a folder with an `infra/` template that creates at least one resource, `azd
up` provisions and then runs the same eval service target. Worth a pass if you
already have such a project; `azd ai eval init` should recommend `azd up` there
rather than `azd ai eval create`.

### More to try

Short prompts, no commands — improvise.

- **Empty state:** every `list` in a new project; no endpoint configured; no azd
  environment; a config file that is missing, empty, or has a misspelled key.
- **Bad input:** names with spaces, unicode, slashes or `..`; a `--from-file`
  directory with several `.jsonl` files, or none; an empty `.jsonl` and one with
  a malformed row; a file saved by Notepad (UTF-8 with BOM).
- **Config edits:** an evaluator needing a column the dataset lacks; a dataset
  pinned to a deleted version; renaming an eval; editing only a description.
- **Scripting:** `-o json` everywhere including failures; `--output-file` at a
  directory, a read-only path, a deep path; very long names; narrow terminals.
- **Interruption:** Ctrl-C mid-`create` and mid-run, then re-run; two `create`s
  at once; `--no-wait`, cancel, then ask for output.
- **Cross-extension:** `azd ai dataset list` vs `azd ai eval dataset list`, and
  anything else in both. They should answer the same way, exit codes included.

---

## Command reference

**`azd ai eval`** — `init`, `generate`, `create [name]`, `list`, `show <eval>`,
`delete <eval>`

- `create [name]` takes the name as an argument, not a flag. `--from-file`
  creates from a file instead of a project.

**`azd ai eval dataset`** and **`azd ai eval evaluator`** — `create`, `update`,
`list`, `show`, `delete`, `versions list`

- `evaluator list --builtin` is how you discover the `builtin.*` names.

**`azd ai eval run`** — `start`, `show`, `list`, `cancel`, `delete`, and
`output list|show|export`

**`azd ai eval job`** — `list`, `show`, `cancel`, `delete` for generation jobs.
Needs `--dataset` or `--evaluator`; it fails if you run it bare.

**`azd ai dataset`** — `create`, `update`, `list`, `show`, `delete`,
`versions list`

Every command takes `-o json`, `--debug` and `--help`. All except `init` also
take `--project-endpoint`, which wins over the environment variable.

---

## Cleanup

Artifacts are not removed by uninstalling. Delete evals by **id** — `azd ai eval
list -o json` gives you them — because a name shared by more than one eval is
refused. Deleting an eval also discards its runs.

```bash
azd ai eval delete <eval-id> --no-prompt
azd ai eval dataset delete <name> --version <version> --no-prompt
azd ai eval evaluator delete <name> --version <version> --no-prompt
```

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
registry and stop on a prompt, which hangs a script.

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
   `azd ai eval list -o json` then `azd ai eval delete <eval-id> --no-prompt`.

2. **`--judge-model` and `--generation-model` read as optional but are
   effectively required** against a shared project reached by endpoint, because
   there is no `azure.yaml` model deployment to detect. `init` says so rather
   than writing an undeployable config. (Bug 5511012.)

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
