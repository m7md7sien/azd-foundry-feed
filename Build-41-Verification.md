# Build 41: source and acceptance checklist

**Published non-Latest on 2026-09-23.** Combined source gates, packaging,
independent targeted package acceptance, all 15 anonymous downloads and a
fresh public Windows installation passed. Final hosted CI and Latest
promotion remain pending; build 40 remains Latest.

This release corrects the observed responses-backed evaluation contract,
catalog evaluator-pin reconciliation, and init dataset binding/custom paths.
Earlier build 40 evidence is inherited history, not a build 41 rerun.

| Item | Published value |
| --- | --- |
| Release | [extensions-2026-09-23-41](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-23-41) |
| Evaluations | `1.0.41-beta` |
| Dataset | `1.0.0-beta.29` |
| Required azd | `>=1.33.0`, confirmed in both manifests and the registry |
| Source for both extensions | [`8ef8b6df77336950c60506ab2966037f579d92cd`](https://github.com/m7md7sien/azure-dev/commit/8ef8b6df77336950c60506ab2966037f579d92cd) |
| Registry SHA256 | `aff0d6f456e3fb08773b1c888136eed12ec8a06bd2eba0addda0383142ae7d79` |
| Publication | Non-Latest at `2026-09-23T10:41:09Z`; final hosted proof required before promotion |

## Install this build

Use a fresh `AZD_CONFIG_DIR` so an older development registry or installed
binary cannot mask the package. These pins do not follow the Latest designation:

```bash
azd extension source add -n foundry-candidate-41 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-41/registry.json
azd extension install azure.ai.evaluations --source foundry-candidate-41 --version 1.0.41-beta
azd extension install azure.ai.dataset --source foundry-candidate-41 --version 1.0.0-beta.29
azd ai eval version -o json
azd ai dataset version -o json
```

Adding a source does not replace installed binaries. Check an existing
source's URL, or explicitly uninstall/reinstall in the intended configuration.

[SHA256SUMS](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-41/SHA256SUMS)
and [source-provenance.json](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-41/source-provenance.json)
describe the immutable artifacts. Provenance verification fields record the
build-time state and are not rewritten after later acceptance or publication.

## Changes and measured acceptance

### Responses-backed evaluation

The canonical stored-response request shape and compatible evaluation schema
are used together. The exact versioned Windows package completed **one graded
row with two distinct passing evaluator results and zero execution errors**.
Native JSON matched the measured service results, and human output showed both
results. Only the owned response and conversation were reused.

This verifies the corrected contract combination for the measured case. It
does not independently isolate every cause of the previous failure, validate
every responses-backed scenario, or confirm a GA simulation contract. The
[build 40 failure](./Build-40-Verification.md#confirmed-responses-backed-runtime-failure)
remains a known issue in that immutable older package.

### Catalog evaluator pins and legacy history

A native create/reconcile control on a manual custom rubric changed the
authored evaluator pin from `1` to `2` to unset. Actual GET results contained
versions `"1"`, `"2"` and `""`, respectively. Three immutable eval IDs were
created; unchanged repeats reused the corresponding ID, and earlier criteria
remained unchanged. This control submitted **zero grading runs**.

Exact-source HTTP/in-process fixtures cover legacy-state migration and
history preservation. The persisted legacy-index seam requires both matching
effective pins and an appropriate response schema before cached reuse.
Live legacy-state migration, rename and `up` were **not** exercised.

### Init dataset binding and custom configuration paths

Six actual CLI cases covered same-file reuse and same-basename collisions
across the tested custom, reference-only, default-directory and legacy-directory
configurations. Existing pinned declarations, unknown metadata, prior evals
and references were preserved. Rejected collisions made no authored/private
state changes, and no unwanted default sidecar was created for the tested
custom/legacy files.

Two actual Windows ConPTY cases passed: correction from an invalid named
dataset through a same-basename collision to a distinct valid file, and Ctrl+C
at collision correction with no authored changes. Models, limits and existing
declarations were retained. Add-only semantics and comments were preserved;
normal YAML indentation normalization was permitted.

The final ConPTY checks explicitly supplied an invalid local dataset to enter
correction. An initial harness expected a text prompt where two declared
datasets selected a picker; that attempt is not counted as a passing case.
Its process exited, and a read-only diagnostic was cancelled. These two
successful paths are not exhaustive interactive coverage.

## Packaging and gates

Both extensions were built in fresh isolated staging at the exact approved
source. `go.mod`, `go.sum` and official changelogs are unchanged. Only each
extension's `extension.yaml` and `version.txt` received package-version
overrides. Existing `azd x build --all --skip-install --no-prompt` and
`azd x pack --bundle` tooling produced six archives per extension: ZIP for
Windows/macOS and tar.gz for Linux, each with its manifest and platform entrypoint.
Build, packaging and fresh local installation took 2.13 minutes with warm tooling.

| Gate | Status |
| --- | --- |
| Exact source BUILD and PUBLISH approvals | Approved for the source/version/registry tuple above |
| Combined source gates | Passed: full short tests, builds, vet, lint and modernization checks; the final persisted pin/schema integration regression passed |
| Independent source fixtures | Passed 28 exact-source groups, including catalog pins, response schemas, legacy-index migration and the combined guard; not live coverage of every migration path |
| Twelve archive layouts/manifests/entrypoints/hashes and binary provenance | Passed; all extracted executable bytes and source/platform metadata match |
| Fresh isolated Windows install on azd 1.33.0 | Passed exact bundle/archive/registry/installed hashes, JSON versions, VCS identity and help |
| Targeted package behavior | Passed six CLI and two ConPTY cases, one live response row/two passing results, and the zero-run manual catalog-pin control |
| Independent cleanup | Passed: ten owned service identities returned HTTP 404; zero owned processes/children; temporary fixtures/configuration/virtual environment removed |
| Non-Latest publication | Passed: all 15 assets uploaded individually and server sizes/digests verified, with literal `registry.json` last |
| Anonymous downloads and fresh public installation | Passed all 15 SHA256s and registry metadata/URLs; exact public Windows JSON versions and executable bytes match |
| Final hosted CLI and full source-race proof | Pending: planned 160 actual CLI checks per Linux/Windows OS plus both full source-race suites |
| Latest promotion and stable-URL/fresh-install verification | Not started; build 40 remains Latest |

The public Windows amd64 executable SHA256s match independent acceptance:
evaluations `ce8b8906a52f9879470ace66daab4edf71795d0566bd45243271eed9c54d0255`;
dataset `43b6223ecb3d2702f3d00c0731e1ad8f758800b940971a5fe9050b18d419cbf5`.
Builds 38, 39 and 40 retain their original assets and versions.

## Cleanup and evidence boundaries

The independent gate used one graded row, one evaluation run and one stored
agent response. It submitted zero data-generation jobs, paid retries or
payload variants. All ten newly owned service identities returned HTTP 404
after cleanup. Temporary fixtures, isolated configuration and virtual
environment were removed, with zero owned processes/children. Original build
39 files and prior build 40 packages, registry and receipts stayed unchanged.

This is a targeted corrective gate, not a repetition of the complete build
40 matrix, dataset-download scenarios, all targets/platforms, GA probing or
broad trace access. The earlier 40 critical scenarios and other historical
results retain their source/package identities. Private ledgers, raw service
responses, prompts and infrastructure details are not published here.

| Remaining issue or gate | Boundary |
| --- | --- |
| Service generation-count/cost issue | The reported `simple_qna` request for 15 returned 16; a separate seed case returned 15. The merged service patch does not cover the reported path. No client truncation or deployed fix is claimed. |
| Terminal data-generation deletion | Backend issue remains open; historical HTTP 409 records are not cleared and have no verified retention TTL. |
| Deprecated agent hint | Deferred and outside these packages. |
| GA, privacy and approvals | GA contract/deployment alignment, privacy/AA and upstream approvals remain external. The shipped preview simulation contract is not relabeled as GA. |
| Authenticated live-cloud CI | Still requires appropriate isolated-project identity/access and separate authorization. Offline hosted proof is not live-cloud evaluation or quality-gate CI. |

No all-scenarios, all-platform, all-blockers-fixed or unconditional readiness
claim is made.
