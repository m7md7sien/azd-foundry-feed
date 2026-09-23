# Bug bash: `azd ai eval` and `azd ai dataset`

Two prerelease `azd` extensions for Foundry evaluations. Publishing and running
against a **real, shared Foundry project** creates resources and can cost real
model calls. Scaffolding with `init` is not a live evaluation.

| Extension | Namespace | Original proposal |
|---|---|---|
| `azure.ai.evaluations` | `azd ai eval` | [Azure/azure-dev#9500](https://github.com/Azure/azure-dev/pull/9500) |
| `azure.ai.dataset` | `azd ai dataset` | [Azure/azure-dev#9499](https://github.com/Azure/azure-dev/pull/9499) |

**File findings as bugs, not PR comments: https://aka.ms/evalsbug**
Include your OS, `azd version`, both extension versions, the release tag, and the
exact command with sanitized output. Never post tokens, private prompts,
customer data, or full raw live-service responses.

**Release status:** the published versions and bundled PRs are listed in the
[README](./README.md). Bundled means included in a feed build, not merged
upstream. These scenarios are test instructions, not a claim that every scenario
has passed. Baseline simulation and rubric evidence does not verify a newer
candidate. Service deployment, GA simulation contract/deployment alignment, and
privacy signoff remain separate gates.

These instructions target **build 39**, evaluations `1.0.39-beta` and dataset
`1.0.0-beta.27`. The source below pins that release for reproducible results.
The README separately documents the rolling Latest source. Build 39 has
[completed targeted package and independent focused fresh-user follow-up](./Build-39-Verification.md#independent-focused-fresh-user-follow-up)
with no new functional defect observed in that earlier scope. Broader scenario evidence
remains build 38 history; not all journeys were rerun on build 39. Keep those
evidence sets separate.

**Subsequent build 39 reports remain open:** simulation `init` can write
configuration despite blank seed descriptions or `desired_num_turns: 0`;
`create` protects publication, not those local writes. Failed-run output
actionability is also reopened. Read the
[current known issues](./Build-39-Verification.md#newly-reported-known-issues)
before testing. Existing scoped passes are not an all-blockers-fixed claim.

The [actual Windows ConPTY checkpoint](./Build-39-Verification.md#actual-windows-conpty-checkpoint)
also completed for 13 scoped cases, including Ctrl+C, explicit Cancel,
add-only file preservation, and JSON-only console stdout, with local cleanup
confirmed. Bounds were supplied as flags and some inputs were prefilled.
Escape cancellation was not established; this is not a blanket interactive
or all-platform pass.

---

## Quick start

Install Azure Developer CLI **1.33.0 or later** before starting. Check with
`azd version`, then authenticate using `az login` and `azd auth login`.

The shell examples below use Bash. On PowerShell, use the same single-line azd
commands and create the shown UTF-8 JSONL files in your editor rather than using
`printf` or Bash line continuations.

For a fresh-user check that does not alter your normal installed extensions,
set `AZD_CONFIG_DIR` to a new, empty directory before any azd command. Keep that
value for the whole check and authenticate in that configuration. For example,
in PowerShell:

```powershell
$env:AZD_CONFIG_DIR = Join-Path $env:TEMP ("azd-foundry-bugbash-" + [guid]::NewGuid())
$env:AZURE_DEV_COLLECT_TELEMETRY = "no"
```

Everywhere below, replace `<you>` with your alias. **Names must be unique** --
the project is shared and evals persist, so prefix your datasets, evaluators
and evals to avoid collisions with other testers.

**Returning testers:** use a fresh azd configuration or explicitly reinstall
the two extensions from `foundry-candidate-39` below. Adding a source does not
replace installed binaries. If this source name already exists, check that its
URL matches rather than silently using an older registry.

```bash
# 1. install the extensions
azd extension source add -n foundry-candidate-39 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-39/registry.json
azd extension install azure.ai.evaluations --source foundry-candidate-39 --version 1.0.39-beta
azd extension install azure.ai.dataset --source foundry-candidate-39 --version 1.0.0-beta.27

# 2. make a project to work in
mkdir azd-eval-bugbash
cd azd-eval-bugbash
azd init --minimal --no-prompt -e bugbash

# 3. point this environment at the shared project
azd env set FOUNDRY_PROJECT_ENDPOINT https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral

# 4. your first evaluation
azd ai eval init --source traces --target support-agent --judge-model gpt-4.1-nano --name <you>-trace-eval --no-prompt
azd ai eval create
azd ai eval run start
```

That evaluates traces the shared `support-agent` has already produced, not
necessarily your own requests. It is a shared-data example, **not an
own-trace-isolation test**. For owned-agent/version and isolated trace scenarios,
use [the setup below](#optional-owned-agent-and-own-trace-setup) instead of
changing the shared agent. Runtime depends on sample count, model latency and
service load. Then read the per-sample results:

```bash
azd ai eval run output list --eval <you>-trace-eval
```

**If that run fails with `No trace data found`**, check the selected agent,
version/window, tracing configuration, query permissions, and ingestion delay.
Do not assume this proves the agent has not run, and do not broaden an
own-trace test to other testers' traffic. Record the blocker or use a dataset:

```bash
mkdir -p evals/datasets
printf '%s\n' '{"query":"How do I reset my password?","response":"Settings, then Security, then Reset password."}' '{"query":"What are your support hours?","response":"Weekdays, 9am to 5pm."}' > evals/datasets/<you>-rows.jsonl

azd ai eval init --source dataset --dataset ./evals/datasets/<you>-rows.jsonl --target support-agent --judge-model gpt-4.1-nano --name <you>-ds-eval --evaluator builtin.relevance --no-prompt
azd ai eval create <you>-ds-eval
azd ai eval run start --eval <you>-ds-eval
azd ai eval run output list --eval <you>-ds-eval
```

The endpoint is saved in this azd environment. `--project-endpoint` overrides it;
otherwise the environment value takes precedence over a machine-wide
`azd ai project` selection and over variables exported in your shell.

**Check what you installed:** both extensions should use `foundry-candidate-39`
and match the versions in the [pinned registry][registry].

[registry]: https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-39/registry.json

```bash
azd extension list --installed
azd ai eval version
azd ai dataset version
```

Sources do not replace installed binaries automatically. Releases and versions
must not be overwritten. To select this build explicitly in an existing
configuration, uninstall the installed extensions and reinstall from the pinned
source (skip an uninstall if that extension is absent):

```bash
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension install azure.ai.evaluations --source foundry-candidate-39 --version 1.0.39-beta
azd extension install azure.ai.dataset --source foundry-candidate-39 --version 1.0.0-beta.27
```

## Optional: owned-agent and own-trace setup

The bug-bash feed installs only evaluations and datasets. Seeing only `eval`
and `dataset` under `azd ai --help` is expected. Agent management is a separate,
independently versioned [official extension][agents-install]. Install it from
`azd`, **not** `foundry-candidate-39`, in the same isolated configuration:

```bash
azd extension install azure.ai.agents --source azd --version 1.0.0-beta.16
azd extension list --installed
azd ai agent version
azd ai agent init --help
azd ai agent invoke --help
```

These agent commands were checked against installed `azure.ai.agents`
`1.0.0-beta.16` with azd 1.33.0. Its dependencies install separately; it is not
part of this feed. Help verification is not an end-to-end agent deployment pass.

### Existing project and uniquely owned agent

For CLI initialization, `--project-id` means the Azure Resource Manager **resource
ID**, not `FOUNDRY_PROJECT_ENDPOINT`. In [Foundry](https://ai.azure.com), open the
authorized existing project, then **Manage > Project details > Resource ID**, as
documented in [connecting an existing project][existing-project]. Copy that
value; do not derive a subscription or resource group from the endpoint. If you
cannot obtain it or lack access, record that setup blocker. Do not create a
replacement project, deployment, or role assignment in the shared subscription.

Use a new directory, a unique owned agent name, and an **existing** model
deployment. The installed prompt-agent initializer documents this syntax:

```bash
azd ai agent init --kind prompt --agent-name <you>-bugbash-agent --project-id "<existing-project-resource-id>" --model-deployment gpt-4.1-nano --instructions "Answer the test user's support questions clearly." --no-prompt
```

Review the generated `azure.yaml`: confirm the existing project, existing model,
and your unique agent identity before deployment. `init` creates configuration;
it does not prove an agent exists. The [official prompt-agent quickstart][prompt-agent]
also documents creation using the existing project endpoint and model. Its
resource-creation prerequisite links are not permission to provision new shared
infrastructure for this bug bash.

When deploying through azd, the help-checked form `azd deploy <owned-service-key>`
selects one service from your own `azure.yaml`; an unqualified `azd deploy` or
`azd up` can act on more resources. Never target the shared `support-agent`.
Inspect the actual deployed identity/version with
`azd ai agent show <owned-service-key> -o json`.

### Owned-agent version 2 is a separate test

For prompt agents, the official [versioning guidance][agent-versions] says to
save subsequent changes as new immutable versions. Make such changes **only to
the uniquely named agent you created**. Record the service-assigned name and
version before and after; do not assume the next version is `2`, and do not
change or deploy version 2 of `support-agent`.

This guide's publication does **not** establish a live-tested CLI recipe for
owned-agent version-2 creation. Use the official versioning workflow and report
the observed versions, or mark the scenario blocked. Dataset v1/v2 evidence is
not agent-versioning evidence. The installed agent `invoke --version` flag
selects an existing version; it does not create one.

### Generate and evaluate only your own traces

Follow [the official tracing setup][agent-tracing]: an Application Insights
connection and telemetry query permissions are required, including Log Analytics
Reader and any protected-table permissions. Ask the project owner to confirm
the existing shared configuration; do not change its connections or permissions.

After your owned agent is deployed, copy its endpoint from
`azd ai agent show <owned-service-key>` and record your request's UTC time window.
The installed help and [official invoke guidance][agent-invoke] support:

```bash
azd ai agent invoke --agent-endpoint "<owned-endpoint-from-agent-show>" --new-session "What are your support hours?"
```

Use only the endpoint of your owned agent. `--new-session` separates conversation
history, **not** traces across agents or users. Allow ingestion and confirm that
your requests appear in the project's traces before evaluating them:

```bash
azd ai eval init --name <you>-own-traces --source traces --target <you>-bugbash-agent --judge-model gpt-4.1-nano --trace-days 1 --max-traces 1 --no-prompt
```

In this eval's generated `source` block, replace the broad lookback with your
actual version and request window. The following fields are verified against
the evaluation schema retained from build 38; replace every placeholder with an observed value:

```yaml
source:
  type: traces
  agent_name: <you>-bugbash-agent
  agent_version: "<actual-agent-version>"
  start_time: "<request-start-UTC-RFC3339>"
  end_time: "<request-end-UTC-RFC3339>"
  max_traces: 1
```

Remove the generated `lookback_hours` when supplying `start_time`; they cannot
be combined. End must follow start. Then:

```bash
azd ai eval create <you>-own-traces
azd ai eval run start --eval <you>-own-traces --no-prompt
azd ai eval run output list --eval <you>-own-traces
```

Check the returned agent/version and trace or conversation identity against your
own request. A unique eval name alone does not isolate traces, and a time window
is not a user/session authorization boundary. If the intended trace is missing,
record the setup/service blocker instead of widening to shared-agent traffic.
Scoped replay has separate live evidence. The focused build 39 follow-up did
not repeat this complete owned-agent setup or versioning journey; broader
build 38 evidence must not be relabeled as a new build 39 execution.

[agents-install]: https://learn.microsoft.com/azure/foundry/agents/how-to/install-cli-foundry-extensions
[existing-project]: https://learn.microsoft.com/azure/foundry/agents/how-to/init-agent-project#connect-to-an-existing-foundry-project
[prompt-agent]: https://learn.microsoft.com/azure/foundry/agents/quickstarts/prompt-agent
[agent-versions]: https://learn.microsoft.com/azure/foundry/agents/concepts/development-lifecycle#save-changes-as-versions
[agent-tracing]: https://learn.microsoft.com/azure/foundry/observability/how-to/trace-agent-setup
[agent-invoke]: https://learn.microsoft.com/azure/foundry/agents/how-to/invoke-hosted-agent

## Command surface

| Group | Commands |
| --- | --- |
| `azd ai eval` | `init`, `generate`, `create [name]`, `list`, `show <eval>`, `delete <eval>` |
| `azd ai eval dataset` | `create`, `update`, `list`, `show`, `download`, `delete`, `versions list` |
| `azd ai eval evaluator` | `create`, `update`, `list`, `show`, `download`, `delete`, `versions list` |
| `azd ai eval run` | `start`, `list`, `show`, `cancel`, `delete`, `output list`, `output show`, `output export` |
| `azd ai eval job` | `list`, `show`, `cancel`, `delete` |
| `azd ai dataset` | `create`, `update`, `list`, `show`, `download`, `delete`, `versions list` |

Every command takes `-o json` and `--no-prompt`. Use `--help` for command-specific
flags.

## Hero Scenarios

These are **examples, not a script**. Work through them to get oriented, then go
wherever you like -- the most useful findings come from things nobody wrote down.

Quick start above is scenario 1. Give it its own folder. Scenarios 2 to 6 share
a second folder, because 3 onwards read what 2 writes. Scaffold it the same way:

```bash
mkdir azd-eval-bugbash-2
cd azd-eval-bugbash-2
azd init --minimal --no-prompt -e bugbash
azd env set FOUNDRY_PROJECT_ENDPOINT https://asayedahmed-ngen-swcentral-resou.services.ai.azure.com/api/projects/asayedahmed-ngen-swcentral
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

Then open one sample to inspect the scores and reasons actually returned.
Build 39 displays returned `properties.dimension_scores` and preserves nested
result/sample fields in JSON. Its scoped two-dimension package check matched
numeric values, applicability, full reasons, and JSON/export data to retained
service evidence. A rubric definition alone still does not guarantee dimension
output. Missing values must not be inferred as zero, false, or a failed verdict.

```bash
azd ai eval run output list --eval <you>-regression-eval
azd ai eval run output show <item-id> --eval <you>-regression-eval
azd ai eval run output show <item-id> --eval <you>-regression-eval -o json
```

Record which metrics and explanations are present. Applicability is not a
pass/fail verdict, and the detail view does not derive dimensions from the rubric
definition. JSON may include sensitive prompts and answers; keep it in private
outputs, not public bug reports or shared CI logs. Use the human lookup ID with
`output show`; JSON preserves the service's returned identity.

Build 38's original fresh-user run had no retained raw response, so its
per-dimension coverage remains unverified. That history is not replaced by the
new package's separately measured result.

### 3. Inner loop

First re-run unchanged. Nothing changed, so nothing should be republished.

```bash
azd ai eval create
azd ai eval run start
```

Now change **what is being evaluated** -- not how it is judged, which is
scenario 4 -- and run it again. That is the loop this scenario exists for, and
it is what makes the comparison below mean something. Edit the agent's
instructions in the Foundry portal **only for a separate agent you own**.
Never edit or redeploy the shared `support-agent`, its versions, or shared model
deployments. Complete the [owned-agent setup and versioning boundary](#optional-owned-agent-and-own-trace-setup)
first. Point your eval at your owned agent before this step, or skip the mutation
and record the coverage gap. Then:

```bash
azd ai eval create
azd ai eval run start
azd ai eval run list --eval <you>-regression-eval
```

**Expect:** an unchanged `create` skips everything. Editing an agent in the portal
does not make `eval create` deploy the agent. It reconciles only evaluation
resources; the next run invokes the configured agent. `run list` shows the runs
side by side. Record the agent version used so the comparison is meaningful.

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
something of its own** -- a different dataset or evaluator. Do not use a positive
sample cap to distinguish evals over registered datasets: build 39 rejects that
unsupported subset request. An eval
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
azd ai eval run output export <run-id> --eval <you>-gate --output-file results.json
```

Export supports **JSON only** (`--format json`, the default). It writes one
document containing the run and its evaluated items; naming the file `.csv`
does not convert it. Convert the exported JSON separately if another format
is needed.

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

```text
support-agent-evals: Deploying (Created eval <you>-reg-eval (eval_...))
support-agent-evals: Deploying (Eval <you>-reg-eval is unchanged (eval_...))
```

`azd ai eval create` afterwards should agree, and prints the same detail without
the hurry. A disagreement is a finding.

### 7. `azd up` in a project that has infrastructure

`azd up` is the right verb only where the project really provisions something.
In a folder with an `infra/` template that creates at least one resource, `azd
up` provisions and then runs the same eval service target. Worth a pass if you
already have such a project. `azd ai eval init` recommends a named
`azd ai eval create` for the added eval; `azd up` is a whole-project alternative,
not a prerequisite for deploying data-plane evaluation resources.

### 8. Full authoring: static conversations and simulation

`init` is an add-only scaffold, not a complete editor for every supported YAML
shape. It does not generate data or start a run. Its bounded, best-effort
built-in evaluator catalogue check is not an assurance that the project, agent,
models, or every evaluator will work at run time. `generate` produces artifacts;
it must not silently attach them to an existing eval or replace its configuration.
Read its next steps, then explicitly declare or initialize the eval you want.
The `--conversation-mode static|simulation` flags and explicit simulator limits
introduced in build 38 remain supported. Its
[authoring examples](./Build-38-Verification.md#candidate-authoring) show those
unchanged command forms; install build 39 using this guide's pinned source.
Full YAML remains useful beyond the scaffold.

Author the full configuration when the scaffold does not expose the desired
mode. Add a service to `azure.yaml` without replacing existing services:

```yaml
services:
  evals:
    host: azure.ai.eval
    $ref: ./evals/azure.eval.yaml
```

In a new scenario folder, create `evals/datasets/<you>-recorded.jsonl` containing
a completed conversation:

```jsonl
{"messages":[{"role":"user","content":"What are your support hours?"},{"role":"assistant","content":"Weekdays, 9am to 5pm."}]}
```

Create `evals/datasets/<you>-seeds.jsonl` containing a scenario, not a completed
exchange:

```jsonl
{"test_case_description":"Ask for support hours, then ask whether weekend support is available.","desired_num_turns":2}
```

Create `evals/azure.eval.yaml` as below. If using an existing config, merge the
catalog entries and evals into their existing lists instead of overwriting it.
Paths are relative to that file. The judge and simulation models must both be
deployed in your project, and the evaluator must support conversation level.

```yaml
datasets:
  - name: <you>-recorded
    file: ./datasets/<you>-recorded.jsonl
  - name: <you>-seeds
    file: ./datasets/<you>-seeds.jsonl
evals:
  - name: <you>-static-conversation
    dataset: <you>-recorded
    evaluation_level: conversation
    evaluators:
      - evaluator: builtin.task_completion
        initialization_parameters:
          model: gpt-4.1-nano
  - name: <you>-simulated-conversation
    dataset: <you>-seeds
    evaluation_level: conversation
    target:
      type: agent
      name: support-agent
    simulation:
      model: gpt-4o-mini
      num_conversations: 1
      max_turns: 2
    evaluators:
      - evaluator: builtin.task_completion
        initialization_parameters:
          model: gpt-4.1-nano
```

Run each eval explicitly:

```bash
azd ai eval create <you>-static-conversation
azd ai eval run start --eval <you>-static-conversation --no-prompt
azd ai eval create <you>-simulated-conversation
azd ai eval run start --eval <you>-simulated-conversation --no-prompt
```

**Expect:** the static eval scores the stored `messages` without invoking an
agent. It intentionally has neither `target` nor `simulation`. The simulation
eval uses the seed to create an interaction with the agent, then scores the
resulting transcript. A seed row is not an agent `query`; do not bind it to
`{{item.query}}`. Keep generation, simulation, and judge models separate.

`num_conversations` accepts 1 through 5 per seed and defaults to 1 when omitted.
`max_turns` accepts 1 through 20; omission preserves the service default.
Explicit zero is invalid for the numeric simulation settings. Seed rows also
require a nonblank description and positive whole-number `desired_num_turns`
when supplied, but **build 39 `init` can accept a blank description or zero
desired turns and write configuration**. Check those rows yourself before
scaffolding; `create` rejects them before dependency publication, which does
not resolve the init-local-write bug. `desired_num_turns` cannot exceed an explicit
`max_turns`. Desired turns are a target, not an exact-length guarantee; the
maximum is a hard ceiling. Early termination alone is not evidence of a CLI
bug. Start with one seed and a small turn budget because every
conversation invokes models. Inspect run output and transcript shape, not just
the run acceptance status.

### Sample caps and registered dataset identity

The following describes **build 39**. Build 37 allowed bounded inline subsets
of registered data and did not treat an explicit CLI zero as an override; that
historical behavior is not the current contract.

| Mode | Supported bound and expected behavior |
| --- | --- |
| Genuinely unregistered local dataset run | Inline rows and a positive cap are allowed only after confirmed remote absence. A complete valid empty listing requires not-found confirmation; malformed/incomplete listings and authorization/service failures stop the run. Build 39's scoped local-cap case scored one row without publishing a dataset. |
| Registered dataset, including an already published `file:` declaration | Uses the service-issued version identity. Positive sample caps are rejected because this source has no supported subset option. Publish/select a smaller dataset deliberately instead. |
| Identity lookup, authorization, or missing-ID failure | Stop rather than silently send inline rows. Do not construct or guess an `azureai://` identity. |
| Conversation simulation | Does not accept a sample cap. Bound the number of seed rows, `num_conversations`, and `max_turns` instead. Do not combine it with a `source` block. |
| Traces, other source-backed runs, and reruns selected by eval ID | Reject explicit sample-cap flags that cannot affect this source. Use trace-source limits or select response IDs. |
| Data generation | `generate --max-samples` requests a service generation count, not a dataset-run cap. A reported `simple_qna` request for 15 produced 16 results/file rows, while the separate seed case produced 15. Overshoot is not established as fixed or deployed. |

For ordinary dataset runs, an explicit CLI `--max-samples 0` overrides a positive
YAML cap. Negative values are invalid. No temporary subset dataset is published
automatically, and a registered source does not silently become an inline copy.

**Explicit publish-first route:** curate the desired rows in the local file
before publishing, explicitly register that dataset, then run the registered
version without a positive sample cap. For a new ordinary turn-dataset eval:

```bash
azd ai dataset create <you>-curated --from-file ./evals/datasets/<you>-curated.jsonl --no-prompt
azd ai eval init --source dataset --dataset <you>-curated --target support-agent --judge-model gpt-4.1-nano --evaluator builtin.relevance --name <you>-curated-eval --no-prompt
azd ai eval create <you>-curated-eval
azd ai eval run start --eval <you>-curated-eval --no-prompt
```

The registration is an explicit shared-service mutation, not an automatic
temporary dataset. Use your own unique name and confirm the published version.
If reusing an eval, remove its positive YAML `max_samples` or explicitly override
it with `--max-samples 0`. The registered run uses the service-issued `file_id`;
do not fabricate that identity. Build 38's unregistered-local failures and this
publish-first workaround remain historical evidence; its immutable packages
have not been replaced by the build 39 fix.

### Dataset downloads

Build 39's targeted checks passed `--output-file` with exact source bytes in
both namespaces, including refusal to overwrite without `--force` and directory
output:

```bash
azd ai dataset download <you>-curated --version <version> --output-file dataset.jsonl
azd ai eval dataset download <you>-curated --version <version> --output-file eval-dataset.jsonl
```

A container-backed single-file download must have a complete one-file listing
and `isSingleFile: true` metadata. A one-file folder is still a folder and
requires `--output-dir`; multiple-file datasets retain their relative layout.
Build 38 still has its documented single-file `--output-file` issue and
`--output-dir` workaround. Do not apply build 39 results to those older binaries.

### Observed conversation output

When a waited run already reads all output rows for its summary, build 39 can
report unique conversation output IDs and output-item lifecycle statuses.
These are **not** generated/completed-conversation totals, evaluation verdict
totals, or inferred actual turns. Duplicate IDs count once; missing IDs and
unknown/conflicting statuses are separate. Filtered/paged listings and detail
views without all rows do not establish complete counts. No extra service
fetch or inference of missing values is promised.

These authoring examples require a matching-candidate live pass before being
marked verified. A missing deployment, unavailable evaluator, or service
rejection is a recorded blocker, not a successful test. The current simulation
run uses `azure_ai_user_conversation_simulation_preview`. The
[public GA proposal and current target differ](./Build-39-Verification.md#public-ga-proposal-versus-current-deployment):
the target's non-executing schema check rejected the proposed GA name. Keep the
shipped preview wire; GA contract/deployment alignment remains external.

### More to try

Short prompts, no commands -- improvise.

- **Empty state:** every `list` in a new project; no endpoint configured; no azd
  environment; a config file that is missing, empty, or has a misspelled key.
- **Bad input:** names with spaces, unicode, slashes or `..`; a `--from-file`
  directory with several `.jsonl` files, or none; an empty `.jsonl` and one with
  a malformed row; a file saved by Notepad (UTF-8 with BOM).
- **Config edits:** an evaluator needing a column the dataset lacks; a dataset
  pinned to a deleted version; renaming an eval; editing only a description.
- **Two `init`s in one project:** both evals land in the one configuration. Give
  the second a different evaluator or dataset -- two evals identical apart from
  their name are refused, and `create` then refuses the whole file rather than
  just the duplicate.
- **Somewhere other than `evals/`:** `init --path ./quality`, then run the rest
  without `--path`; a `--path` that is absolute, nested, or already holds a
  config; two `init`s in one project.
- **More than one environment:** `azd env new`, give each a different
  `FOUNDRY_PROJECT_ENDPOINT`, then run with `-e` and check the right project
  was touched and lookups in each environment keep reaching that project's
  evals.
- **Scripting:** `-o json` everywhere including failures; `--output-file` at a
  directory, a read-only path, a deep path; very long names; narrow terminals.
- **Interruption:** Ctrl-C mid-`create` and mid-run, then re-run; two `create`s
  at once; `--no-wait`, cancel, then ask for output.
- **Cross-extension:** `azd ai dataset list` vs `azd ai eval dataset list`, and
  anything else in both. They should answer the same way, exit codes included.

---

## Cleanup

Artifacts are not removed by uninstalling. Delete evals by **id** -- `azd ai eval
list -o json` gives you them -- because a name shared by more than one eval is
refused. Deleting an eval also discards its runs. Delete only resources and
versions you can prove this test created. Never delete shared agents, models,
or another tester's dataset/evaluator versions.

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
azd extension source remove foundry-candidate-39
```
