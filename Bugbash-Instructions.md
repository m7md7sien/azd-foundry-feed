# Bugbash Instructions: `azd ai eval` and `azd ai dataset`

Two prerelease `azd` extensions for Foundry evaluations. Everything below runs
against a **real Foundry project** â€” these commands create datasets, evaluators,
evals and runs in it, and cost real model calls.

| Extension | Namespace | PR |
|---|---|---|
| `azure.ai.evaluations` | `azd ai eval` | [Azure/azure-dev#9500](https://github.com/Azure/azure-dev/pull/9500) |
| `azure.ai.dataset` | `azd ai dataset` | [Azure/azure-dev#9499](https://github.com/Azure/azure-dev/pull/9499) |

---

## 1. Setup

**Prerequisites**

```bash
# azd 1.27.1 or later (both extensions declare requiredAzdVersion ">=1.27.1")
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

You also need access to the shared bug-bash Foundry project â€” see below. Access
is granted through the **Evaluation Service Team** group (`raisvcteam@microsoft.com`),
so if you are on the team you already have it.

**You need an azd project.** The scenarios run from the root of one, because the
eval service is added to its `azure.yaml`. If you do not have one, run these
three lines in any shell:

```
mkdir azd-eval-bugbash
cd azd-eval-bugbash
azd init --minimal --no-prompt -e bugbash
```

`--minimal --no-prompt` matters. Plain `azd init` asks how to initialise the
app, then asks for a project name; even `azd init --minimal` still asks for the
name. Both forms hang a script or an agent that cannot answer.

**Install from the feed**

```bash
azd extension source add -n foundry-bugbash -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-08-10/registry.json

azd extension install azure.ai.evaluations --source foundry-bugbash
azd extension install azure.ai.dataset --source foundry-bugbash
```

`--source` matters if you have ever added a local extension registry: without
it `azd` finds the extension in more than one source and stops on a prompt,
which will hang a script.

Each extension is about 15 MB and the feed is a GitHub release, so the progress
bar can sit still for several minutes on the first install. That is expected.
Let it finish rather than interrupting it.

Confirm both resolve to the `foundry-bugbash` source, not a local build, and
that you are on **1.0.4-beta** of `azure.ai.evaluations` and **1.0.0-beta.5**
of `azure.ai.dataset`:

```bash
azd extension list --installed
```

**If you installed earlier, upgrade.** Since beta.1 these have been fixed:
`eval show <name>`; `eval create` now publishes the dataset and evaluator it
references; `eval init` fails with a clear message when it has no judge model
rather than writing an undeployable config; `--from traces` works; a mistyped
key in `azure.eval.yaml` names the key and suggests the right one; Windows paths
in errors are copyable again; the dataset extension refuses a malformed `.jsonl`
row before uploading it; a generated rubric keeps its `pass_threshold` instead
of losing it on the way to disk and being republished without one; and
`eval create` now reports each artifact as published or unchanged rather than
claiming it created something every time; a misspelled `--target` says the agent
is not in the project instead of printing the whole 404; a dataset that was
never generated names the command that writes it; `eval delete` now takes
the name `eval list` prints, refusing only when that name is carried by more
than one eval, since deleting one would discard its runs;
`dataset versions list <name>` no longer says "No datasets found." in a project
full of datasets; the evaluation configuration is now replaced atomically, so a
second command reading it while one writes can no longer see it empty and write
back a file with every eval missing; `--fail-on` is refused with `--no-wait`
instead of being silently dropped; and a generation with nothing to seed from
says so in one line rather than returning thirty lines of JSON.

```bash
azd extension upgrade azure.ai.evaluations
azd extension upgrade azure.ai.dataset
```

**Point at the shared project.** Set it once for the session â€” you will be
running a lot of commands, and the endpoint is long enough that retyping it per
command invites a typo into the wrong project.

Pick the line for the shell you are in:

**PowerShell** (Windows default, and `pwsh` on macOS/Linux)

```powershell
$env:FOUNDRY_PROJECT_ENDPOINT="https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral"
```

**Command Prompt** (`cmd.exe` on Windows) â€” no quotes, no spaces around `=`

```bat
set FOUNDRY_PROJECT_ENDPOINT=https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral
```

**bash / zsh** (macOS, Linux, WSL, Git Bash)

```bash
export FOUNDRY_PROJECT_ENDPOINT=https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral
```

Check it took: `azd ai dataset list -o json` should print `[` and exit 0.

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

**Smoke test** â€” this must print `[` and exit 0 even in an empty project:

```bash
azd ai dataset list -o json
azd ai eval list -o json
```

**There is a second eval surface, and it is not this one.** The agents extension
ships `azd ai agent eval` with its own `generate`, `run`, `list`, `show` and
`update`. It is a different tool with a different config: it writes an
`eval.yaml` at the project root, while this extension writes
`evals/azure.eval.yaml`. The two are not interchangeable and neither reads the
other's file.

Which to use is worth an opinion from you. Rough shape of the difference:
`azd ai agent eval` is one agent, one command, sensible defaults; `azd ai eval`
declares datasets, evaluators and evals in a config you keep in the repo and
reconcile. Confusion between them is a legitimate finding — please file it.

---

## 2. How to test

The hero scenarios in section 3 and the list in section 4 are **examples, not a
script**. Work through them to get oriented, then go wherever you like â€” the
most useful findings usually come from something nobody thought to write down.

**File every finding as a bug in Azure DevOps** â€” not as a PR comment.

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

These follow the spec's own sequence, so run them **in order** â€” later ones use
artifacts earlier ones create. `support-agent` already exists in the shared
project. Where a model is asked for, use `gpt-4.1-nano`.
Scenario 1 stands alone, so give it its own folder. Scenarios 2 to 5 share one:
3, 4 and 5 read the config and rubric that 2 writes, so starting 3 in a fresh
directory just fails on a missing file.

The names below are the spec's, but the project is shared and evals persist, so
**put your alias in every name you create**. This is not politeness, it is the
difference between the scenarios working and not: once two evals share a name,
`run start` refuses to guess between them, and the id it tells you to use is
rejected by `run start --eval <id>` until that eval has already run once. The
two errors point at each other and there is no way out except a unique name.
The project already holds four evals called `support-agent-trace-eval`, so the
spec's literal names are already in that state.

**If you are already stuck**, list the ids and delete the ones that are yours:

```bash
azd ai eval list -o json
azd ai eval delete <eval-id> --no-prompt
```

This applies to anything you invent too, not just the scenarios below. Any eval
name you reuse from someone else lands you in the same place.
Commands are shown one per line so they paste into PowerShell and bash alike.

### Scenario 1 â€” First evaluation of an agent after manual testing

You have been chatting with an agent and want a first signal, with no dataset
prepared. This evaluates the traces the agent already produced.

```bash
azd ai eval init --source traces --target support-agent --judge-model gpt-4.1-nano --name <you>-trace-eval
azd ai eval create
azd ai eval run start
azd ai eval run output list --eval <you>-trace-eval
```

Replace `<you>` with your alias. `init` asks which evaluators to grade with, so
it stops for input; add `--no-prompt` to take the detected default
(`builtin.task_adherence`) and paste the block straight through.

`run start` reads the eval out of the config, so it needs no `--eval`.
`run output list` does not read the config, so it does.

`--judge-model` reads as optional ("detected from the project when omitted"),
but detection only ever worked for a model deployment declared in `azure.yaml`.
Against a shared project reached by endpoint there is nothing to detect, so pass
it explicitly.

Until beta.3 this failed silently: `init` exited 0 having written a config whose
graders had no `deployment_name`, and the first sign of trouble came much later
from the service. beta.3 makes it an error that names the flag. Tracked as
[Bug 5511012](https://msdata.visualstudio.com/Vienna/_workitems/edit/5511012) â€”
please don't re-file.

**Why not `azd up`?** Neither `azd up` nor `azd deploy` works in a project
scaffolded by `azd init --minimal`. `azd up` fails compiling a bicep template
that does not exist. `azd deploy` fails with "infrastructure has not been
provisioned" -- because that scaffold's environment carries no
`AZURE_SUBSCRIPTION_ID` or `AZURE_LOCATION`, not because of anything to do with
evals; set both and it does run.

Eval resources are data-plane only, so there is nothing to provision either way.
`azd ai eval create` reconciles the config directly, needs nothing but an
endpoint, and is what these scenarios use. As of 1.0.3-beta the extension's own
"Next:" hint says `azd ai eval create` too, and only says `azd up` in a project
that really does carry infrastructure. Known, please don't re-file.

**Expect:** `init` makes no service calls and writes two things:
`evals/azure.eval.yaml`, and a `support-agent-evals` service entry added to
`azure.yaml`. `azd ai eval create` reconciles the config. The run prints a
summary with a row per evaluator; a healthy first run is 20 samples in about a
minute.

### Scenario 2 â€” Turning that signal into a repeatable baseline

Generate a dataset and a rubric evaluator, then declare an eval over them.

```bash
azd ai eval generate --from agent --target support-agent --generation-model gpt-4.1-nano --dataset-name <you>-regression --evaluator-name <you>-quality
azd ai eval init --name <you>-regression-eval --dataset <you>-regression --target support-agent --judge-model gpt-4.1-nano --evaluator builtin.task_adherence --evaluator <you>-quality --no-prompt
azd ai eval create
azd ai eval run start
```

`--generation-model` is **required**. It names the deployment that writes the
samples and the rubric. `gpt-4.1-nano` exists in the shared project.

`--from traces` also works as of beta.4; before that it was rejected by the
service. Either is worth exercising. You will see a warning that agent-seeded
generation failed and that it is retrying from the instruction alone. That is
expected, it succeeds, and it is a known service-side defect.

The two `--evaluator` flags matter. Without them `init` plans a *second* rubric
named `<target>-quality` and writes it into the config, so you have to run
`generate` again for that one before `create` will work. Passing `--evaluator`
names the evaluator `generate` actually produced and skips that step. If you do
hit it, the error now names the command that writes the missing definition.
Then inspect one sample (take an item id from the run output):

```bash
azd ai eval run output show <item-id> --eval support-agent-regression-eval
```

**Expect:** `generate` submits two jobs, downloads both artifacts, and adds them
to `evals/azure.eval.yaml`. A second `azd ai eval create` with no edits must
publish **nothing**, and as of beta.7 it says so per artifact:

```
Dataset support-agent-regression is unchanged at version 1.0
Evaluator support-agent-quality is unchanged at version 1
(-) Skipped: Eval support-agent-regression-eval is unchanged (eval_...)
```

Before beta.7 every create printed `Created eval` whether or not it had created
anything, and the evaluator really was republished as version 2 on the first
create, because the downloaded rubric had lost its `pass_threshold`. If you see
a version 2 appear from an edit you did not make, that is the bug returning.

### Scenario 3 â€” Inner loop: change the agent, re-evaluate

```bash
# ...edit the agent's instructions, then:
azd ai eval create
azd ai eval run start --eval support-agent-regression-eval
azd ai eval run list --eval support-agent-regression-eval
```

**Expect:** the eval keeps its identity across runs so the history is
comparable. Editing one eval must not disturb another in the same file.

### Scenario 4 â€” Tuning the evaluation itself

```bash
# ...edit evals/evaluators/support-agent-quality.json, then:
azd ai eval create
azd ai eval evaluator versions list support-agent-quality
azd ai eval create
```

**Expect (important):** the evaluator gains a new version, and `azd ai eval create` must
report the eval as unchanged. The eval keeps its id, so runs
from before and after the rubric edit remain comparable. **If the eval is
recreated here, that is a bug** â€” unless you also edited the eval's own entry in
`evals/azure.eval.yaml`. Changing what the eval *declares* (its evaluator list,
dataset, target, level, or adding or removing a `version:` pin) is supposed to
create a new eval. Editing the rubric the evaluator points at is not.

### Scenario 5 â€” Automation and CI/CD

This one needs a gate eval. Add a second eval to `evals/azure.eval.yaml` named
`<you>-gate`, then create it **by name**:

```bash
azd ai eval create <you>-gate
```

Once the file declares two evals, bare `azd ai eval create` refuses and names
both forms: the argument for `create`, `--eval` for the run commands. Before
1.0.4-beta it said "choose one with --eval", which `create` does not accept.

```bash
azd ai eval run start --eval <you>-gate --fail-on pass-rate=0.8
azd ai eval run start --eval <you>-gate --fail-on any-failure
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

**Known rough edges â€” already reported, please don't re-file**

- `WARNING: 1 extension did not start.` It is **`azure.ai.agents`**, and it is
  not one of the two extensions under test. It and `azure.ai.projects` both try
  to register the `microsoft.foundry` provisioning provider; the second one
  loses with `AlreadyExists`. Harmless for eval work.
- `azd extension install <id>` stops on a source-selection prompt if the id is
  in more than one registry, which is why the install commands above pass
  `--source foundry-bugbash`. Only bites you if you have added a local registry.
- `azd up` needs `infra/main.bicep`, and `azd deploy` needs an environment that
  has provisioned something, so neither works in a scratch project. Use
  `azd ai eval create`. Verified end to end on 2026-08-11.
- `azd init` and `azd init --minimal` both prompt. Use
  `azd init --minimal --no-prompt -e <name>`.
- Token acquisition fails intermittently with
  `AzureDeveloperCLICredential: exit status 1` while `azd` reports a valid
  login. Retry the command; it usually succeeds on the next attempt.
- `--judge-model` and `--generation-model` are both effectively required despite
  reading as optional (Bug 5511012). On beta.3 `init` now says so instead of
  failing later. See Scenarios 1 and 2.
- `-o json` still prints prose on the failure path, so a failing command breaks
  a JSON pipe. The `ERROR:` line is written by `azd` itself, on stdout, not by
  these extensions, so it cannot be fixed from here.
- `--fail-on` is documented to exit **2**, but `azd` collapses every extension
  failure to exit **1**. Non-zero vs zero is still correct.
- Agent-seeded data generation fails server-side for every agent. `generate`
  detects it, says so, and retries from the agent's instructions alone, which
  succeeds. Expected until the service is fixed.

---

## 4. Other scenarios worth trying

Again â€” examples, not a checklist. Short descriptions only, so you improvise the
commands. Anything you invent beyond this list is fair game and welcome.

**Empty and missing state**
- Every `list` command in a brand-new, empty project
- Commands with no Foundry endpoint configured at all (clear it first:
  `Remove-Item Env:\FOUNDRY_PROJECT_ENDPOINT` in PowerShell,
  `set FOUNDRY_PROJECT_ENDPOINT=` in cmd, `unset FOUNDRY_PROJECT_ENDPOINT` in bash)
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
- Ctrl-C in the middle of `azd ai eval create` and of a run, then re-run the same command
- Two `azd ai eval create` runs at once against the same project
- `--no-wait`, then cancel the run, then ask for its output

**Cross-extension consistency**
- The same concept through both extensions â€” for example
  `azd ai dataset list` vs `azd ai eval dataset list` â€” and any command that
  exists in both. They should answer the same way, including exit codes.

---

## 5. Full command reference

### `azd ai eval`

| Command | Purpose |
|---|---|
| `init` | Scaffold evaluation config for an agent. Makes no service calls. |
| `generate` | Generate a dataset and a rubric evaluator, and download them. |
| `create [name]` | Create one eval declared in the configuration. Name it when the config declares more than one; `--from-file` creates from a file instead of a project. |
| `list` | List the project's evals. |
| `show` | Show an eval definition. |
| `delete` | Delete an eval and everything under it. |

**`azd ai eval dataset`** â€” `create`, `update`, `list`, `show`, `delete`, `versions list`

**`azd ai eval evaluator`** â€” `create`, `update`, `list`, `show`, `delete`, `versions list`

**`azd ai eval run`** â€” `start`, `show`, `list`, `cancel`, `delete`, `output`

**`azd ai eval run output`** â€” inspect per-sample results (`list`, `show`, `export`)

**`azd ai eval job`** â€” `list`, `show`, `cancel`, `delete` (generation jobs)

### `azd ai dataset`

| Command | Purpose |
|---|---|
| `create` | Register a dataset, publishing its first version. |
| `update` | Publish a new version of a dataset. |
| `list` | List the project's datasets. |
| `show` | Show a dataset version. |
| `delete` | Delete a dataset version. |
| `versions list` | List the versions of a dataset. |

Every command accepts `-o json`, `--debug`, and `--help`. All of them except
`init` also accept `--project-endpoint`; `init` makes no service calls, so it
has no endpoint to point at and rejects the flag. `--no-prompt` is `azd`'s own
global flag, and these commands honor it by skipping their prompts.

Two more worth knowing: `azd ai eval job` needs `--dataset` or `--evaluator`
(it fails if you run it bare), and `azd ai eval evaluator list --builtin` is how
you discover the `builtin.*` names used in Scenario 2.

---

## 6. Cleanup

The bug bash leaves datasets, evaluators, evals and runs behind. To remove the
extensions and the feed:

```bash
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension source remove foundry-bugbash
```

Artifacts created in the Foundry project are not removed by uninstalling. Delete
them from the project, or with the `delete` verbs. They confirm before deleting,
so add `--no-prompt` to run them unattended. There is no `--force`.

```bash
azd ai eval delete <eval-id> --no-prompt
azd ai eval dataset delete <name> --version <version> --no-prompt
azd ai eval evaluator delete <name> --version <version> --no-prompt
```

Delete evals by **id**, not name: `azd ai eval list -o json` gives you the ids,
and a name shared by more than one eval is refused. Deleting an eval also
discards its runs.
