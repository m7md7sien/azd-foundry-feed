# Bugbash Instructions: `azd ai eval` and `azd ai dataset`

Two prerelease `azd` extensions for Foundry evaluations. Everything below runs
against a **real Foundry project** — these commands create datasets, evaluators,
evals and runs in it, and cost real model calls.

| Extension | Namespace | PR |
|---|---|---|
| `azure.ai.evaluations` | `azd ai eval` | [Azure/azure-dev#9500](https://github.com/Azure/azure-dev/pull/9500) |
| `azure.ai.dataset` | `azd ai dataset` | [Azure/azure-dev#9499](https://github.com/Azure/azure-dev/pull/9499) |

---

## 1. Setup

**Prerequisites**

```bash
# azd 1.27.0 or later
azd version

# if you need to install or upgrade it
curl -fsSL https://aka.ms/install-azd.sh | bash        # macOS / Linux
```

```powershell
# Windows
powershell -ex AllSigned -c "Invoke-RestMethod 'https://aka.ms/install-azd.ps1' | Invoke-Expression"
```

```bash
# sign in to both
az login
azd auth login

# confirm you are on the right subscription
az account show --query "{name:name, id:id}" -o table
```

You also need access to the shared bug-bash Foundry project — see below. Access
is granted through the **Evaluation Service Team** group (`raisvcteam@microsoft.com`),
so if you are on the team you already have it.

**You need an azd project.** The scenarios run from the root of one, because the
eval service is added to its `azure.yaml`. If you do not have one:

```bash
mkdir azd-eval-bugbash && cd azd-eval-bugbash
azd init
```

**Install from the feed**

```bash
azd extension source add -n foundry-bugbash -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-08-10/registry.json

azd extension install azure.ai.evaluations
azd extension install azure.ai.dataset
```

Confirm both resolve to the `foundry-bugbash` source, not a local build:

```bash
azd extension list --installed
```

**Point at the shared project.** Set it once for the session — you will be
running a lot of commands, and the endpoint is long enough that retyping it per
command invites a typo into the wrong project.

```bash
export FOUNDRY_PROJECT_ENDPOINT=https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral
```

```powershell
$env:FOUNDRY_PROJECT_ENDPOINT="https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral"
```

Every command also takes `--project-endpoint <url>`, which wins over the
environment. Use it only to aim a single command somewhere else.

**What is already there**

| | |
|---|---|
| Agents | `support-agent` (the one the scenarios use), `test-agent` (minimal) |
| Models | `gpt-4.1-nano`, `gpt-4o-mini`, `gpt-4.1`, `gpt-5.1`, `o4-mini`, `text-embedding-3-large` |
| Region | swedencentral |

The project is shared and already holds other people's work. Prefix anything you
create with your alias so it is easy to tell apart and clean up.

**Smoke test** — this must print `[` and exit 0 even in an empty project:

```bash
azd ai dataset list -o json
azd ai eval list -o json
```

---

## 2. How to test

The hero scenarios in section 3 and the list in section 4 are **examples, not a
script**. Work through them to get oriented, then go wherever you like — the
most useful findings usually come from something nobody thought to write down.

**File every finding as a bug in Azure DevOps** — not as a PR comment.

> **Bug template (start here): https://aka.ms/evalsbug**
>
> It opens a pre-filled work item in
> [msdata.visualstudio.com/Vienna](https://msdata.visualstudio.com/Vienna)
> under area path `Vienna\Observability\Evaluation`. Sign in with your
> Microsoft account if prompted.

Include:

- Your OS and `azd version`
- The exact command and its full output
- If the failure is not self-explanatory, re-run with `--debug` and attach the log

---

## 3. Hero scenarios

These follow the spec's own sequence, so run them **in order** — later ones use
artifacts earlier ones create. `support-agent` already exists in the shared
project. Where a model is asked for, use `gpt-4.1-nano`.

Commands are shown one per line so they paste into PowerShell and bash alike.

### Scenario 1 — First evaluation of an agent after manual testing

You have been chatting with an agent and want a first signal, with no dataset
prepared. This evaluates the traces the agent already produced.

```bash
azd ai eval init --source traces --target support-agent
azd up
azd ai eval run start --eval support-agent-trace-eval
azd ai eval run output list --eval support-agent-trace-eval
```

**Expect:** `init` makes no service calls and writes `evals/azure.eval.yaml`.
`azd up` reconciles it. The run prints a summary with a row per evaluator.

### Scenario 2 — Turning that signal into a repeatable baseline

Generate a dataset and a rubric evaluator, then declare an eval over them.

```bash
azd ai eval generate --from traces --dataset-name support-agent-regression --evaluator-name support-agent-quality
azd ai eval init --name support-agent-regression-eval --dataset support-agent-regression
azd up
azd ai eval run start --eval support-agent-regression-eval
```

Then inspect one sample (take an item id from the run output):

```bash
azd ai eval run output show <item-id> --eval support-agent-regression-eval
```

**Expect:** `generate` submits two jobs, downloads both artifacts, and adds them
to `evals/azure.eval.yaml`. A second `azd up` with no edits must publish
**nothing** — every line should say `(unchanged)`.

### Scenario 3 — Inner loop: change the agent, re-evaluate

```bash
# ...edit the agent's instructions, then:
azd up
azd ai eval run start --eval support-agent-regression-eval
azd ai eval run list --eval support-agent-regression-eval
```

**Expect:** the eval keeps its identity across runs so the history is
comparable. Editing one eval must not disturb another in the same file.

### Scenario 4 — Tuning the evaluation itself

```bash
# ...edit evals/evaluators/support-agent-quality.json, then:
azd up
azd ai eval evaluator versions list support-agent-quality
azd up
```

**Expect (important):** the evaluator gains a new version, and `azd up` must
report the eval as `Skipped ... (unchanged)`. The eval keeps its id, so runs
from before and after the rubric edit remain comparable. **If the eval is
recreated here, that is a bug** — unless you also edited the eval's own entry in
`evals/azure.eval.yaml`. Changing what the eval *declares* (its evaluator list,
dataset, target, level, or adding or removing a `version:` pin) is supposed to
create a new eval. Editing the rubric the evaluator points at is not.

### Scenario 5 — Automation and CI/CD

This one needs a gate eval. Declare it by adding a second eval to
`evals/azure.eval.yaml` named `support-agent-gate` and running `azd up`, or just
substitute `support-agent-regression-eval` below.

```bash
azd ai eval run start --eval support-agent-gate --fail-on pass-rate=0.8
azd ai eval run start --eval support-agent-gate --fail-on any-failure
```

Start without blocking, capture the id, then reattach:

```bash
azd ai eval run start --eval support-agent-gate --no-wait
```

```powershell
$EVAL_RUN_ID = "<the run id printed above>"
azd ai eval run show $EVAL_RUN_ID --eval support-agent-gate --wait --fail-on pass-rate=0.8
```

```bash
export EVAL_RUN_ID=<the run id printed above>
azd ai eval run show "$EVAL_RUN_ID" --eval support-agent-gate --wait --fail-on pass-rate=0.8
```

Export results for a build artifact:

```bash
azd ai eval run output export <run-id> --eval support-agent-gate --format csv --output-file results.csv
```

**Known limitation, not a bug:** `--fail-on` is documented to exit **2**, but
`azd` currently collapses every extension failure to exit **1**. Non-zero vs
zero is still correct — only the specific code is wrong. Please do not file it.

---

## 4. Other scenarios worth trying

Again — examples, not a checklist. Short descriptions only, so you improvise the
commands. Anything you invent beyond this list is fair game and welcome.

**Empty and missing state**
- Every `list` command in a brand-new, empty project
- Commands with no Foundry endpoint configured at all (unset
  `FOUNDRY_PROJECT_ENDPOINT` first — `unset` in bash,
  `Remove-Item Env:\FOUNDRY_PROJECT_ENDPOINT` in PowerShell)
- Commands with no active `azd` environment
- An eval config file that does not exist, and one that is completely empty

**Bad input**
- Dataset and evaluator names containing spaces, unicode, slashes, or `..`
- A `--from-file` pointing at a directory holding several `.jsonl` files
- A `--from-file` pointing at a directory holding none, or at a `.csv`
- An empty `.jsonl`, and one with a malformed row in the middle
- A dataset file saved by Notepad (UTF-8 with BOM)
- Names that already exist, and names that do not exist, for every verb

**Config editing**
- A misspelled key in `evals/azure.eval.yaml` (e.g. `evaulators:`)
- An evaluator that requires a column the dataset does not have
- A dataset pinned with `version:` to a version that was deleted
- Two evals in one file where you edit only one of them
- Renaming an eval, and editing only its `description:`

**Output and scripting**
- `-o json` on every command, piped into a parser, including failure cases
- `--output-file` pointing at a directory, a read-only path, and a deep path
- Very long names and values, to see whether tables break
- Terminals at narrow widths

**Concurrency and interruption**
- Ctrl-C in the middle of `azd up` and of a run, then re-run the same command
- Two `azd up` runs at once against the same project
- `--no-wait`, then cancel the run, then ask for its output

**Cross-extension consistency**
- The same concept through both extensions — for example
  `azd ai dataset list` vs `azd ai eval dataset list` — and any command that
  exists in both. They should answer the same way, including exit codes.

---

## 5. Full command reference

### `azd ai eval`

| Command | Purpose |
|---|---|
| `init` | Scaffold evaluation config for an agent. Makes no service calls. |
| `generate` | Generate a dataset and a rubric evaluator, and download them. |
| `create` | Create one eval declared in the configuration. |
| `list` | List the project's evals. |
| `show` | Show an eval definition. |
| `delete` | Delete an eval and everything under it. |

**`azd ai eval dataset`** — `create`, `update`, `list`, `show`, `delete`, `versions list`

**`azd ai eval evaluator`** — `create`, `update`, `list`, `show`, `delete`, `versions list`

**`azd ai eval run`** — `start`, `show`, `list`, `cancel`, `delete`, `output`

**`azd ai eval run output`** — inspect per-sample results (`list`, `show`, `export`)

**`azd ai eval job`** — `list`, `show`, `cancel`, `delete` (generation jobs)

### `azd ai dataset`

| Command | Purpose |
|---|---|
| `create` | Register a dataset, publishing its first version. |
| `update` | Publish a new version of a dataset. |
| `list` | List the project's datasets. |
| `show` | Show a dataset version. |
| `delete` | Delete a dataset version. |
| `versions list` | List the versions of a dataset. |

Every command accepts `--project-endpoint`, `-o json`, `--debug`, and `--help`.

---

## 6. Cleanup

The bug bash leaves datasets, evaluators, evals and runs behind. To remove the
extensions and the feed:

```bash
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension source remove foundry-bugbash
```

Artifacts created in the Foundry project must be deleted through the project
itself, or with the `delete` verbs above.
