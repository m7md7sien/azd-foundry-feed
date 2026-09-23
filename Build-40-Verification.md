# Build 40: source and acceptance checklist

**Published and promoted to Latest on 2026-09-23.** Corrected source gates,
scoped local acceptance, final Linux/Windows installed-CLI and full source-race
CI, all 15 anonymous asset checks, and fresh pinned/Latest Windows installations
passed. Promotion followed the final cold-entry filesystem checks below.
The earlier `0f7caa5f` attempt was withheld for an
integration-only test signature mismatch; it was not a new product defect.
Build 39 retains its [known issues](./Build-39-Verification.md#newly-reported-known-issues).
Its registry, versions, packages, and historical evidence are unchanged.
Earlier source changes and development-artifact passes were not substituted
for this candidate's exact-package acceptance.

| Item | Candidate value |
| --- | --- |
| Release | [extensions-2026-09-23-40](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-23-40) |
| Evaluations | `1.0.40-beta` |
| Dataset | `1.0.0-beta.28` |
| Required azd | `>=1.33.0`, verified in both corrected package manifests and registry |
| Source for both extensions | [`361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a`](https://github.com/m7md7sien/azure-dev/commit/361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a) |
| Registry SHA256 | `648c9632fcf5a4e1f993cb0cb4c5bc37691e5821df8ec8422c36a2b30f548d9f` |
| Publication | Complete; non-Latest first, then Latest after final hosted and anonymous gates |

## Install this build

Use a fresh `AZD_CONFIG_DIR` so an existing development registry or installed
binary cannot mask the published package. This pinned source is independent
of Latest:

```bash
azd extension source add -n foundry-candidate-40 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-40/registry.json
azd extension install azure.ai.evaluations --source foundry-candidate-40 --version 1.0.40-beta
azd extension install azure.ai.dataset --source foundry-candidate-40 --version 1.0.0-beta.28
azd ai eval version -o json
azd ai dataset version -o json
```

Adding a source does not replace installed binaries. Check an existing source's
URL instead of silently reusing it. Use an isolated configuration or explicitly
replace the installed extensions.

[SHA256SUMS](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-40/SHA256SUMS)
and [source-provenance.json](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-40/source-provenance.json)
describe the immutable artifacts. Provenance verification fields are a
build-time snapshot, not the current acceptance status; later evidence is below.
The live-scenario field does not promise a new build 40 Azure run.

## Approved source scope

The [integrated follow-ups from the build 39 source](https://github.com/m7md7sien/azure-dev/compare/9a549449c2d5c2b4c6661f0ee8f8da7e3036c898...361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a)
address the following behaviors, with the measured acceptance scope below.
The corrected commit changes two test assignments in one file
relative to `0f7caa5f`, with no production-code changes or assertions removed.

### Simulation init before authored writes

Locally available seed rows, including declared files and local nested `$ref`
entries, must be checked before configuration writes. Each seed requires
non-whitespace text in `test_case_description`. A supplied `desired_num_turns`
must be a positive whole number no greater than an explicit maximum; omitted
counts remain valid. Seed rows must not mix in `messages`, `query`, or
`response`, including empty or null values.

Interactive init should report invalid rows and allow a corrected or different
dataset before confirmation. Ctrl+C at the correction prompt must preserve
authored files. `--no-prompt` and JSON output should reject invalid local rows
without writing configuration. Valid input must retain add-only behavior.
Shared validation and existing `create`/run guards must remain consistent.
Registered datasets with no local file are not fetched by init; this is not a
new live-init lookup.

The earlier 29-case matrix and development-artifact passes were preliminary
evidence only. The final versioned packages independently passed the 31-case
CLI matrix and two ConPTY paths and are now published. This corrects the
reported build 39 init blocker in build 40, not in the old immutable packages.

### Actionable terminal-run guidance

Waited `run start` summaries and `run show` details should offer an unfiltered
output listing and a JSON export using the resolved immutable eval ID and run
ID. Friendly human labels may remain, but guidance must not select a different
eval after redeployment. Failed-verdict guidance is additional, not a substitute
for unfiltered output; errored rows require their own `--status errored` command.

A whole run can fail with absent/zero result counts and no output rows.
Guidance must inspect **available** output and run diagnostics rather than
implying successful grading. Returned run-level failure messages should redact
URL credentials, query strings, and fragments in human output. JSON should keep
its existing document and exit behavior without appended prose.

Independent synthetic HTTP-caller fixtures are the proof path for those failed
service-response shapes. They are **not** a real operationally failed Azure run.
No paid failure reproduction or TLS-validation bypass is part of this
candidate's acceptance.

## Packaging and gates

Both extensions came from the exact approved SHA in a fresh
`b40-361ca3c33806` staging directory. Source dependency manifests and changelogs
remain unchanged; only isolated extension manifests and version files receive
package versions. Existing `azd x build --all --skip-install` and `azd x pack`
format is retained: six archives per extension, each with `extension.yaml` and
one platform-named entrypoint, ZIP for Windows/macOS and tar.gz for Linux.

| Gate | Status |
| --- | --- |
| Exact source and BUILD approval | Approved for the SHA above |
| Combined source tests/build/vet/lint and go-fix checks | Passed on the corrected source for both modules: full short tests with `NO_COLOR=1`, no skips, builds, vet (evaluation tags), zero lint findings, and no modernization changes |
| Twelve archive layouts/manifests/entrypoints/hashes and extracted binary VCS/platform identity | Passed for the corrected source; only four isolated packaging version files changed |
| Fresh isolated Windows installation, exact JSON versions and binary hashes | Passed on azd 1.33.0; both corrected installed binaries match the extracted archives |
| Final packaged init CLI matrix and two actual ConPTY correction/cancel paths | Passed on the exact corrected Windows amd64 packages: 31 CLI cases (24 invalid refused, 7 valid) and two actual ConPTY paths; details below |
| Independent public-docs-driven local follow-up | Passed on published build 40: 26 actual init invocations, with 18 invalid cases preserving authored YAML/JSONL and eight valid controls; exact public package/registry/installed bytes matched. No Azure or PTY coverage in this lane. |
| Independent verification cleanup and preservation | Passed: zero owned processes; current gate fixtures, isolated config, virtual environment, and test archives removed. The verifier's original build 39 files remained byte-identical. |
| Independent synthetic HTTP-caller proof at the final source | Passed 15 source-test groups at the corrected SHA, including failed/errored responses, immutable IDs, URL redaction, and mixed/valid lifecycle cases. This is source-fixture evidence, not native CLI or live Azure failure execution. |
| Separate PUBLISH approval | Approved for the exact source, versions and registry digest above |
| Complete new draft, individual asset uploads, literal registry last | Passed: all 15 asset names/sizes/server SHA256s verified before publication; no old assets overwritten |
| Non-Latest publication, anonymous downloads and fresh published-source install | Passed: published at `2026-09-23T06:06:42Z`; all 15 anonymous SHA256s, registry metadata/URLs, exact Windows JSON versions and installed binary digests match |
| Final published Linux/Windows CLI and full source-race CI | Passed: [final run 35826828347](https://github.com/m7md7sien/azure-dev/actions/runs/35826828347), 124 actual CLI checks per OS plus both unchanged full race suites; both downloaded evidence artifacts verified |
| Latest promotion and stable-URL/fresh-install verification | Passed: promoted at `2026-09-23T06:30:30Z`; anonymous stable registry matches the approved SHA256, all 15 assets rechecked, and fresh Latest-source Windows install matches versions and executable bytes |

The release contains 12 platform archives plus `source-provenance.json`,
`SHA256SUMS`, and literal `registry.json`. Tag/version collisions were checked
before publication. Final digests are tied to the exact source and runtime
versions. Publication and Latest promotion are separate.

The withheld `0f7caa5f` source completed production build, packaging, and local
installation checks in 2.30 minutes, but its combined tests did not compile.
Those earlier artifacts and receipts remain intact as **prior-attempt evidence
only**. The corrected source used fresh staging with its own version,
digest, and acceptance receipts. No earlier binary was overwritten or relabeled.

The corrected source completed production build, packaging, and fresh local
installation checks in 2.12 minutes. Its exact versioned bundles and new hashes
were delivered for the separate 31-case CLI and two-path ConPTY acceptance.
Those scoped package checks, parent combined-source gates, publication and
final hosted proof have passed.

The fresh public Windows amd64 installation matched these accepted executable
SHA256s: evaluations `fc94f9a171d5e598fb4479d413e5d7a33957b2634d2fb229d43795561a5c0dc8`;
dataset `5e551a44090d384dd66285cb13cdaf2d63e6aff6410cf3b9e387ed3c12fcd828`.

### Final hosted proof

[Run 35826828347](https://github.com/m7md7sien/azure-dev/actions/runs/35826828347)
passed all four jobs using the
[frozen manifest at `c117d9c82891c4c5f6d849a5708f541b71f13f8c`](https://github.com/m7md7sien/azure-dev/blob/c117d9c82891c4c5f6d849a5708f541b71f13f8c/eng/scripts/eval-candidate-proof/candidate.json).
Both source fields match the approved build SHA, and the registry, four
Linux/Windows amd64 archive hashes, extension versions and azd 1.33.0 match the
published packages. Both downloaded CLI artifacts contain **124 unique actual
commands with their expected outcomes** per OS: 68 retained checks, 50 seed-JSON
refusal/state-preservation checks and six valid controls.

Each OS records four cold-entry cases that create exactly the normal
**zero-byte `.azure/.env.lock`**, plus 46 repeated refusals with the entire
project unchanged. All 50 cases preserve every other authored/private path and
content, including the isolated global configuration. Expected/after digests
match in all 50 cases; before/after digests also match in the 46 unchanged
cases. No directories or paths are ignored. The allowed core read-lock residue
is not authored configuration mutation.

Both full internal source-race suites passed at the exact approved source with
Go 1.26.4 and `go test -race -count=1 -timeout 15m ./internal/...`. The earlier
[passing run 35826281452](https://github.com/m7md7sien/azure-dev/actions/runs/35826281452)
used precreated read locks; the final run strengthens cold-entry evidence
without changing source or package pins.

These are hosted **offline** checks. CI did not run interactive correction,
authenticated live-cloud evaluation, a cloud quality gate, or a real failed-run
replay. The separate local ConPTY and synthetic HTTP evidence below retains
its own scope. Final publication and Latest checks did not modify any of the
15 assets or the historical build 38/39 releases.

### Recorded exact-package local acceptance

The independent verifier matched both final bundle hashes, the registry
SHA256, installed executable hashes, JSON versions, embedded corrected VCS
revision, and command help on a fresh azd 1.33.0 Windows amd64 installation.
The withheld `0f7caa5f` attempt was not installed in this verification lane.

All **31 actual CLI cases passed**: 24 invalid-input cases were refused and
seven valid controls succeeded. Both actual Windows ConPTY paths passed:
correcting the dataset twice before a valid confirmation retained independent
models and the requested `5/20` limits; Ctrl+C at correction left authored
files unchanged. This is the measured init matrix and those two interactive
paths, not every possible terminal, keyboard input, or invalid dataset.

The separate 15-group combined-source HTTP fixture result remains source-test
evidence for failed/errored responses, immutable identities, URL redaction, and
lifecycle guards. It does not establish native CLI replay of a real
operationally failed service payload. No Azure operations, paid jobs, shared
resource changes, or retries of historical HTTP 409 job deletion were performed.

Cleanup completed with zero owned processes. The current gate's private fixtures,
isolated build 40 configuration, virtual environment, and test archives were
removed; all nine files in the verifier's original build 39 configuration
remained byte-identical. Packaging staging and immutable bundles are retained
separately for the publication gate. Only this public-safe summary is recorded,
not the private ledger or terminal captures.

### Independent published-package fresh-user follow-up

A separate public-docs/help-driven local follow-up passed **26 actual init
invocations** against published build 40. Eighteen invalid cases made no
authored YAML/JSONL changes, and eight valid controls passed. The public ZIPs,
registry and installed executable bytes matched the candidate identity above.

This was local validation, not Azure execution, authentication, billing, or
an operationally failed-run replay. It did not rerun nested-reference chains
or use a PTY. The earlier 31-case and two-ConPTY acceptance remains separate;
these counts are not combined into a claim of unique or exhaustive coverage.
All owned fixtures, isolated configuration and downloads were removed, with
zero owned processes remaining.

## Evidence boundaries and remaining issues

Build 40 acceptance is **local/offline**: versioned init CLI/ConPTY checks,
synthetic source HTTP fixtures, and package-download/install verification.
Build 39 live-service evidence stays explicitly
tied to build 39; no new build 40 Azure failure execution is claimed.
Earlier broader scenario, field-preservation, simulation, and cleanup evidence
must not be silently relabeled as a build 40 rerun.

| Tracking | Boundary retained for this candidate |
| --- | --- |
| `5631330` | Known open service cost blocker: `simple_qna` request 15 returned result/file count 16, while a separate seed case returned 15. The merged service patch does not cover the reported path. No client-side truncation or service fix is included or claimed. |
| `5595119` | Deprecated agent-hint work remains deferred and outside these packages. |
| `5571322` | Exact published build 39 `run start`/`create` eval-picker Ctrl+C checks passed without ambiguity errors; intentional cancellation does not require a nonzero exit. This is separate from candidate 40 init correction checks. Escape remains unsupported/unproven. |
| `5572139` | Terminal-job deletion remains a backend issue; historical HTTP 409 records have no verified retention TTL or cleanup fix. |

GA proposal/deployment alignment, privacy/AA/upstream review approvals, and
authenticated live-cloud evaluation or quality-gate CI remain external gates.
No all-scenarios, all-platform, or unconditional readiness claim is made.
Public summaries must not include private handoffs, credentials, prompts,
service payloads, or internal architecture/code paths.
