# Build 42: source and acceptance checklist

**Published as a non-Latest prerelease on 2026-09-24.** Exact-source gates,
packaging, scoped installed-package acceptance, all 15 anonymous asset downloads
and a fresh pinned public Windows installation passed. Testing independently
crosschecked the local acceptance and installed identities.

**Build 41 remains Latest and the default feed.** Build 42's exact-candidate
hosted CLI/full-race proof and separate Latest approval are still pending.

This is a focused update to the published build 41 source. It separates an
explicit picker **Cancel** choice from Ctrl+C or prompt failure and initializes
the SDK Project client before concurrent generation workers use it. It does
not include the broader new simulation-contract changes.

| Item | Published prerelease value |
| --- | --- |
| Release | [extensions-2026-09-24-42](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-24-42) |
| Evaluations | `1.0.42-beta` |
| Dataset | `1.0.0-beta.30` |
| Required and measured azd | `>=1.33.0`; local checks used `1.33.0` |
| Source for both extensions | [`d40a3b5a1e7c5944b1b43decd14c96096a99e5b6`](https://github.com/m7md7sien/azure-dev/commit/d40a3b5a1e7c5944b1b43decd14c96096a99e5b6) |
| Integration branch | `m7md7sien-build-42-integration` |
| Direct source parent | [`8ef8b6df77336950c60506ab2966037f579d92cd`](https://github.com/m7md7sien/azure-dev/commit/8ef8b6df77336950c60506ab2966037f579d92cd), published build 41 |
| Registry SHA256 | `83026575746f7db5c5cc7a3035f9e75b776f709875d2f0768bd9c215aaaca635` |
| Publication / Latest | Prerelease published `2026-09-24T02:56:15Z`; not promoted |

## Install this build

Use a fresh `AZD_CONFIG_DIR` and azd 1.33.0 or later. These explicit pins install
the prerelease without following or changing the Latest designation:

```bash
azd extension source add -n foundry-candidate-42 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-24-42/registry.json
azd extension install azure.ai.evaluations --source foundry-candidate-42 --version 1.0.42-beta
azd extension install azure.ai.dataset --source foundry-candidate-42 --version 1.0.0-beta.30
azd ai eval version -o json
azd ai dataset version -o json
```

Adding a source does not replace an already installed extension. Check the
reported versions instead of treating a previous build 41 installation as 42.

[SHA256SUMS](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-24-42/SHA256SUMS)
and [source-provenance.json](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-24-42/source-provenance.json)
describe the immutable assets. Provenance verification fields reflect the
build-time snapshot and are not rewritten after subsequent acceptance.

## Included source changes

The integration contains these selected changes rather than complete moving
review branches:

| Input | Included behavior |
| --- | --- |
| [`bf6c1818`](https://github.com/m7md7sien/azure-dev/commit/bf6c1818722116fc8bcd44cc6ea655a8ca00e2b5) | Evaluation-picker cancellation handling and coverage |
| [`d5392c6`](https://github.com/m7md7sien/azure-dev/commit/d5392c60f44d70633481bc2aac59a9b628d1f029) | Picker documentation/comment correction |
| [`86340741`](https://github.com/m7md7sien/azure-dev/commit/863407412e09c29948ef62f04ef73c480499006a) | Isolated SDK Project initialization before concurrent generation work, with concurrency coverage |

Three assertions in two existing build 41 tests were adapted to check explicit
gRPC cancellation codes. The frozen delta has 14 evaluation-extension files.
Independent source review confirmed all other tracked build 41 files unchanged,
including dataset, core, telemetry, dependencies, source versions, changelogs,
schemas, model handling, simulation contracts, recovery and reporting.

**There is no new dataset functional fix.** The dataset package is versioned
and built from the same approved source as evaluations.

### Picker behavior

Selecting **Cancel** deliberately declines the selection and exits successfully.
Ctrl+C interrupts the prompt and exits nonzero without reporting a successful
cancellation. Prompt failures remain errors.

Under `--no-prompt` or JSON output, no picker is shown. Ambiguous selection
remains an error, and cancellation prose must not contaminate JSON stdout.
The exact build 42 Windows packages passed the targeted checks below.

### Exact-package edge and picker acceptance

A fresh isolated bundle installation matched the source, both versions,
registry hash and both Windows executable hashes in this record. **Eight
actual-command cases passed on their first attempts, with no retries.**
Both `create` and `run start` exercised each of these paths:

| Path | Measured result |
| --- | --- |
| ConPTY input byte `03` | Exit 1; interruption was not reported as successful cancellation |
| Two Down keys, visibly selected Cancel, then Enter | Exit 0 |
| `--no-prompt` with ambiguous selection | Exit 1 and an ambiguity error |
| JSON with ambiguous selection | Exit 1, one error document on stdout, separately captured empty stderr |

The harness confirmed terminal-backed stdin and stdout using
`isatty`/`GetConsoleMode`. The interruption check injected input byte `03`;
it is **not** proof of OS `CTRL_C_EVENT` or `CTRL_BREAK` handling.
Authored, environment and private-state hashes were unchanged except for the
optional normal zero-byte core lock.

All eight owned test processes exited and no owned build 42 extension process
remained. Temporary fixtures were removed; there were no timeouts, forced
cleanup or proxy calls, and no cloud resources were created. The tester retained
its pinned installation and harness for further local coverage. Publisher
assets and configuration were untouched. This is local exact-package evidence,
not public-feed installation or live-service proof.
Testing independently corroborated these same eight cases, parsed each whole
console JSON document and rehashed both retained installed executables.
That corroboration does not add another eight cases.

### Exact-package practical acceptance

A separate fresh Windows installation matched both bundle and archive
digests, installed executable bytes, embedded source, runtime JSON versions
and azd 1.33.0. **Six local cases passed 25 assertions**: static authoring,
flat-seed simulation authoring, reattach preservation, and dataset/run/export
help. The simulation scaffold retained an independent judge, one requested
conversation and a maximum of two turns; this was local authoring, not a
generated or graded service conversation.

The receipt contains 16 command records: 14 `azd` invocations and two Go
metadata inspections. One recorded exit 1 came from a corrected harness
command typo, not a package regression. The original records and correction
chain were retained; this is not an all-commands-first-attempt claim.

All commands were awaited and locks released. No Azure operations, global
configuration mutations or writes to publisher configuration occurred. The
tester's isolated configuration and evidence were retained. This check is
not interactive-picker, Linux-race, hosted, live-service or deferred-contract
acceptance.

Testing independently checked the practical final receipt and rehashed both
installed executables, completing its consolidated local Windows acceptance.
The practical scope is three authoring cases and three help-only cases, not
a live lifecycle. This review adds no cases and does not waive source, hosted
or publication gates.

### Isolated SDK initialization

Before multiple generation plans run concurrently, the shared SDK Project
client is constructed on the calling thread rather than lazily by competing
workers. This does not change generation schemas, request contracts or backend
count/cost behavior. Local/source concurrency checks are not proof of a new
live generation run.

## Branch status is not package provenance

[Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113) was merged
upstream on 2026-09-24 at
[`0fec3b785ac36ea0021952399de59c79d9825223`](https://github.com/Azure/azure-dev/commit/0fec3b785ac36ea0021952399de59c79d9825223).
Its final head was `d5392c60f44d70633481bc2aac59a9b628d1f029`.
That upstream merge is separate from this candidate's pinned integration SHA.

Only the isolated SDK change selected from
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116) is added.
The entire latest PR branch is not bundled by this update. Existing telemetry
from [Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102) is
preserved, not replaced by an unverified moving branch.

## Packaging and current gates

Both extensions were built in a fresh isolated checkout of the exact approved
source using the existing `azd x build --all --skip-install --no-prompt` and
`azd x pack --bundle` tooling. `go.mod`, `go.sum` and official changelogs are
unchanged; only each extension's isolated `extension.yaml` and `version.txt`
received package-version overrides.

The build produced six archives per extension: ZIP for Windows/macOS and
tar.gz for Linux, each containing its manifest and platform entrypoint.
Cross-build, packaging and the initial fresh local installation took
**4.24 minutes**. Additional runtime JSON, installed-byte and checksum checks
then passed. No Azure operations were performed by packaging.

| Gate | Current status |
| --- | --- |
| Exact-source reduced-scope review and BUILD approval | Passed for the source and versions above |
| Windows source gates, both modules | Reported passed: build, full short tests, vet, tagged compile, lint, modernization, focused repetitions, formatting and spelling |
| Focused Linux source race receipt | Passed on exact source: Go 1.26.4, CGO enabled, three repetitions in 81.552 seconds across SDK/private-state, inherited recovery/handoff and gRPC picker/error/noninteractive paths |
| Full Linux source races, both modules | Passed on unchanged source with Go 1.26.4, CGO and GCC: `go test -race -p=2 -count=1 -timeout 15m ./internal/...`; distinct from the required post-publication hosted gate |
| Twelve archive layouts, manifests, entrypoints, extracted bytes and source/platform metadata | Passed |
| Fifteen assets and checksum manifest | Passed; `SHA256SUMS` contains the other 14 assets |
| Registry metadata | Passed: exactly the two extension IDs, one version each, six platforms each, SHA256s, entrypoints, candidate URLs and minimum azd |
| Fresh isolated Windows installation | Passed exact runtime JSON versions and installed executable bytes |
| Matching picker/negative/ConPTY acceptance | Passed: eight first-attempt cases with exact package identity and cleanup |
| Matching practical acceptance | Passed: six local cases and 25 assertions, with the corrected harness typo qualified separately |
| Testing consolidated local Windows gate | Passed: matching receipts, scoped results and both installed executable identities independently crosschecked |
| Non-Latest prerelease publication | Passed: 15 individual checked uploads, matching server sizes/digests, literal `registry.json` last |
| Anonymous downloads and fresh public installation | Passed: all 15 asset SHA256s and registry URLs/metadata; fresh pinned Windows install on azd 1.33.0 matched runtime JSON, source and executable bytes |
| Exact-candidate hosted proof | Pending: 160 actual CLI commands on each Linux/Windows runner plus both full Linux source-race suites |
| Latest promotion, stable registry and fresh unversioned installation | Pending |

Local Windows amd64 executable SHA256s:

| Extension | SHA256 |
| --- | --- |
| Evaluations | `f55c588d6aa333c57454569c70b42a69b96983a730c6d6c39cd2527e9f1224a5` |
| Dataset | `b6cd32c24595ffa8fd17b8fd8ad11b69597d1836d4c1fb141167013c9055cd5f` |

The intended hosted gate retains the existing 160-command-per-OS matrix:
68 earlier checks, 50 seed refusals and six valid controls, and 36 dataset-binding
checks. Its precise cold-read allowance permits only a new zero-byte
`.azure/.env.lock` on the first relevant core read, not ignored directories or
other authored/private writes. New build 42 pins and downloaded evidence are
required; [build 41 proof](./Build-41-Verification.md#final-hosted-proof) does
not satisfy this candidate's gate.

## Deferred work and evidence boundaries

The broader new simulation-contract, qualified-model, nested-seed,
normalization, metadata-fallback and data-mapping changes are excluded.
Their new live qualification is **deferred, not passed**. There is no new
generation or paid-cloud authorization for this smaller release.

Deprecated agent-hint and meta-package work are excluded. Backend generation
count/cost and terminal deletion remain unresolved; no deployed fix, cleared
record or retention TTL is claimed. GA contract/deployment alignment, privacy
and applicable approval/access gates remain external.

Earlier [build 41 acceptance](./Build-41-Verification.md) and older live results
retain their original source and package identities. Offline hosted proof is
not cloud evaluation, quality-gate or interactive proof. Archive availability
for macOS and ARM does not establish runtime acceptance on those platforms.
All prior releases and assets remain immutable. No all-scenarios,
all-platform or unconditional readiness claim is made.
