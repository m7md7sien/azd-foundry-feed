# Build 43: required-only source and acceptance

**Published as a non-Latest prerelease on 2026-09-24.**
Exact-source gates, packaging and required local package acceptance passed.
All 15 anonymous asset downloads and a fresh pinned Windows installation
matched the accepted packages. **Build 42 remains Latest and the default.**
The new published-package hosted gate and a separate Latest decision remain
pending. This is an unofficial beta bug-bash build, not GA.

| Item | Published value |
| --- | --- |
| Release | [extensions-2026-09-24-43](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-24-43) |
| Evaluations | `1.0.43-beta` |
| Dataset | `1.0.0-beta.31` |
| Required and measured azd | `>=1.33.0`; fresh local/public checks used `1.33.0` |
| Source for both extensions | [`067b2fd5622494db2dc6ac3a2e9bc19e871ed9e7`](https://github.com/m7md7sien/azure-dev/commit/067b2fd5622494db2dc6ac3a2e9bc19e871ed9e7) |
| Source tree | `17d7ef40b02aab32540cde560be46caeb57f5e11` |
| Integration branch | `m7md7sien-build-42-integration` |
| Published baseline | Build 42, [`d40a3b5a1e7c5944b1b43decd14c96096a99e5b6`](https://github.com/m7md7sien/azure-dev/commit/d40a3b5a1e7c5944b1b43decd14c96096a99e5b6) |
| Registry SHA256 | `4027cd85bf2a5853db90b4bed12c225eb125197e758e3015a88f9e9aca7a3215` |
| Publication | Non-Latest prerelease at `2026-09-24T07:14:49Z` |

## Install this build

Use a new, empty `AZD_CONFIG_DIR` and azd 1.33.0 or later. These pins select
build 43 even while the rolling Latest source remains build 42:

```bash
azd extension source add -n foundry-candidate-43 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-24-43/registry.json
azd extension install azure.ai.evaluations --source foundry-candidate-43 --version 1.0.43-beta
azd extension install azure.ai.dataset --source foundry-candidate-43 --version 1.0.0-beta.31
azd ai eval version -o json
azd ai dataset version -o json
```

Adding a source does not replace installed binaries. Check source provenance
and archive hashes, not version text alone, and do not reuse an older cached
installation. The main [bug-bash guide](./Bugbash-Instructions.md) continues
to pin build 42 until promotion is separately approved and verified.

[SHA256SUMS](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-24-43/SHA256SUMS)
contains the other 14 assets. The immutable
[source provenance](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-24-43/source-provenance.json)
records build-time verification statuses; later acceptance does not rewrite it.

## Included changes

The net difference from published build 42 contains **ten evaluation files**:
three production, six test and one documentation file. Only these selected
inputs and reviewed test/documentation integration adapters are added:

| Input | Included behavior |
| --- | --- |
| [`f874bf28`](https://github.com/m7md7sien/azure-dev/commit/f874bf28fa84357492875158382de8ada3d06e07) | Preserve exact integer/decimal JSON values and numeric types in typed output-item data, including nested unknown fields; retain strict malformed/trailing-input refusal |
| [`599b5b0e`](https://github.com/m7md7sien/azure-dev/commit/599b5b0e9efcccb26936a27c011debc6f54b738b) | Treat a new YAML path as a file while respecting existing filesystem types; reject unsupported init output formats before side effects |
| [`32ff9bc7`](https://github.com/m7md7sien/azure-dev/commit/32ff9bc772e7500b37ba99709401709e9bda1165) | Reject explicitly empty evaluator selections while preserving omitted defaults and existing selection behavior |

All other tracked files match build 42, including dataset, core, telemetry,
dependencies, schemas, models, manifests, source versions and changelogs.
**There is no new dataset functional fix.** Both extensions were packaged
from the same exact source, with isolated package-only version overrides.

Earlier work from [Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113),
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116) and
[Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102) is inherited
through build 42. This does not import those branches' moving tips or establish
that all bundled changes merged upstream.

## Exact-package local acceptance

### Fresh native Windows commands

Actual azd 1.33.0 commands ran in a fresh source-qualified configuration with
both installed executable hashes, embedded source and runtime JSON verified.
**Three families passed 38 assertions and ten commands including setup.**

| Family | Measured result |
| --- | --- |
| Configuration paths | 20 assertions / six commands including setup: new custom YAML regular file and exact root reference; existing YAML-named directory, existing custom file, canonical and legacy controls |
| Output format | Eight assertions / two commands: unsupported `toml` exits 1 before captured project/destination changes; JSON control exits 0 and produces the exact file/reference |
| Empty evaluator | Ten assertions / two commands: actual empty argument exits 1 before captured project/destination changes; omitted flag exits 0 and persists `builtin.task_completion` |

Project mutation inventories explicitly exclude `.git` internals. Commands
were awaited and owned locks released; project/configuration/evidence were
retained. No global, publisher, authentication or Azure state was changed.
Testing independently checked the same receipts and rehashed both installed
Windows executables. That review adds no cases.

A separate, nonblocking supplement checked `--evaluator=` and a literal
CSV-empty quote pair: six assertions and two commands, both prewrite refusals.
Combined native coverage is 44 assertions / 12 commands including setup,
not a replacement for the original 38 / ten acceptance record. Installation,
version and Go metadata records are identity checks, not additional behavior
cases.

### Fresh Linux numeric runtime

The unmodified Linux amd64 evaluation binary passed **seven fresh cases**:
typed detail, page, file, raw export, ordinary values, malformed input and
trailing input. A new source-qualified test directory was used; no earlier
candidate passes were transferred. Original public build 42 controls were
retained as comparison evidence, with zero new baseline invocations.

Typed detail/page/file preserve the numeric tokens `9007199254740993`,
`-9007199254740993` and `0.123456789012345678901234567890`, their numeric
types and nested unknown fields. The oracle used arbitrary-precision
integer/decimal values plus raw tokens, not floating-point approximation or
quoted numbers. The retained build 42 controls show rounding in these typed
paths.

Raw service export was already exact and remains so; it is not a newly
repaired path. Ordinary output and malformed/trailing-input error behavior
remain unchanged against the retained build 42 controls.

**Scope:** mock SDK gRPC, an explicitly allowlisted mock azd-auth subprocess,
a process-local CA and normal hostname/chain-verified TLS. This is not real
azd-core authentication, live Azure evaluation, billing or service-retention
proof. Served-payload hashes, unchanged executable hashes, TLS negatives,
fail-closed RPC/auth controls and owned-loopback-only network traces passed.
Four adapter-negative checks are separate from the seven executable cases.
All recorded owned processes and servers stopped. No external forwarding,
global trust/DNS changes or product transport changes occurred.

Testing independently verified the same seven case identities, aggregate,
compatibility and cleanup evidence. This adds no test count. No lifecycle
commands for the excluded rubric, local-cleanup or first-publication changes
were run for this candidate.

## Source, packaging and publication gates

| Gate | Status |
| --- | --- |
| Exact-source scope and BUILD approval | Passed: net ten-file required-only map, retained production/test blobs and reviewed integration adapters |
| Fresh Windows source gates, both modules | Reported passed: build, full short tests, vet, tagged checks, lint, modernization, formatting and spelling |
| Genuine Linux source races | Passed on exact source with Go 1.26.4/CGO: focused count 3 and both full suites using `go test -race -p=2 -count=1 -timeout 15m ./internal/...` |
| Packaging | Passed: six archives per extension, layouts/manifests/entrypoints, extracted bytes, VCS/platform metadata, 15 assets and 14-row checksum coverage |
| Registry | Passed: two IDs, one version each, six platforms, exact URLs/digests/entrypoints and minimum azd 1.33.0 |
| Fresh local Windows installation | Passed exact JSON versions, source and installed/archive executable equality |
| Required local package acceptance | Testing consolidated PASS+CLEAN for the scoped Windows and Linux cases above |
| Non-Latest publication | Passed: 15 individual checked uploads, literal `registry.json` last, matching server digests |
| Anonymous public assets | Passed all 15 asset hashes, registry and checksum manifest |
| Fresh public pinned Windows installation | Passed on azd 1.33.0: both JSON versions, embedded source and installed executable bytes |
| New public hosted proof | Pending: exact build 43 pins, 160 actual CLI commands per OS and both full Linux race suites, with downloaded artifacts |
| Latest promotion | Not approved or performed; build 42 remains the default |

Both extensions were built with the existing
`azd x build --all --skip-install --no-prompt` and `azd x pack --bundle`
tooling. `go.mod`, `go.sum` and official changelogs were unchanged; only each
extension's isolated `extension.yaml` and `version.txt` changed. Cross-build,
packaging and initial fresh installation took **2.64 minutes**. Additional
identity, checksum and preservation checks followed.

| Windows amd64 executable | SHA256 |
| --- | --- |
| Evaluations | `9d3e8140c34550c7c52f7b2aedf73ea24e88bd23248db3e8ac7a40b230835d9a` |
| Dataset | `ec402e5c3c4c2d132828584849160e8a2ef0c81f188f3ab0d2c532644c53d178` |

## Deferred work and known limitations

The editable-rubric download/update, local deleted-evaluation cleanup and
empty-version first-publication fixes are **excluded from this release**.
Complete package lifecycle acceptance was not established for that group;
deferral is not an observed product failure. A limited earlier projection
check does not establish whole rubric lifecycle acceptance.

The conditional first-publication problem when the service returns a valid
empty evaluator-version listing **remains a known limitation**. Do not infer
a fix from source fixtures for a change that is not included. Local deletion
state and backend terminal-job deletion/retention are separate concerns.

Broader simulation-contract, qualified-model, nested-seed and other deferred
changes remain excluded. Backend generation-count/cost and terminal deletion
issues, GA contract/deployment alignment, privacy and access approvals remain
external. Historical live results retain their original package identities.
No new paid-cloud testing, all-scenarios pass or unconditional readiness is
claimed.
