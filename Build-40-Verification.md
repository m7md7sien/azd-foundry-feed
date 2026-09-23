# Candidate 40: source and acceptance checklist

**Corrected source build approved and in progress. Not published or approved
for publication.** The earlier `0f7caa5f` attempt was withheld for an
integration-only test signature mismatch; it was not a new product defect.
Build 39 remains Latest with its [known issues](./Build-39-Verification.md#newly-reported-known-issues).
Its registry, versions, packages, and historical evidence are unchanged.
Source changes and development-artifact passes do not establish acceptance of
this candidate's final packages.

| Item | Candidate value |
| --- | --- |
| Planned tag | `extensions-2026-09-23-40`, subject to immutable collision checks |
| Evaluations | `1.0.40-beta` |
| Dataset | `1.0.0-beta.28` |
| Required azd | `>=1.33.0`, to be rechecked in the corrected packages |
| Source for both extensions | [`361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a`](https://github.com/m7md7sien/azure-dev/commit/361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a) |
| Corrected registry SHA256 | Pending; earlier attempt's hashes do not identify this build |
| Publication | Separate approval pending |

No installation URL is offered before real published assets exist.

## Approved source scope

The [integrated follow-ups from the build 39 source](https://github.com/m7md7sien/azure-dev/compare/9a549449c2d5c2b4c6661f0ee8f8da7e3036c898...361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a)
address the following behaviors. These are acceptance targets, not final-package
pass claims. The corrected commit changes two test assignments in one file
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

The earlier 29-case matrix and the development artifact's 31 CLI plus two
ConPTY passes are preliminary evidence only. Final versioned packages require
their own recorded checks. The build 39 init blocker is not relabeled fixed
until a corrected package is verified and published.

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

Both extensions must come from the exact approved SHA in a fresh
`b40-361ca3c33806` staging directory. Source dependency manifests and changelogs
remain unchanged; only isolated extension manifests and version files receive
package versions. Existing `azd x build --all --skip-install` and `azd x pack`
format is retained: six archives per extension, each with `extension.yaml` and
one platform-named entrypoint, ZIP for Windows/macOS and tar.gz for Linux.

| Gate | Status |
| --- | --- |
| Exact source and BUILD approval | Approved for the SHA above |
| Combined source tests/build/vet/lint and go-fix checks | Rerunning on corrected source; no pass inherited from earlier attempts |
| Twelve archive layouts/manifests/entrypoints/hashes and extracted binary VCS/platform identity | Corrected build in progress |
| Fresh isolated Windows installation, exact JSON versions and binary hashes | Pending corrected packages |
| Final packaged init CLI matrix and two actual ConPTY correction/cancel paths | Pending |
| Independent synthetic HTTP-caller proof at the final source | Pending; never a live Azure failure claim |
| Separate PUBLISH approval | Pending |
| Complete new draft, individual asset uploads, literal registry last | Not started |
| Non-Latest publication, anonymous downloads and fresh published-source install | Not started |
| Final published Linux/Windows CLI and full source-race CI | Pending: planned 124 checks per OS plus both unchanged full race suites |
| Latest promotion and stable-URL/fresh-install verification | Not started |

The release should contain 12 platform archives plus `source-provenance.json`,
`SHA256SUMS`, and literal `registry.json`. Recheck tag/version availability before
publication, never overwrite older assets, and keep final digests tied to the
exact source and runtime versions. Publication and Latest promotion are separate.

The withheld `0f7caa5f` source completed production build, packaging, and local
installation checks in 2.30 minutes, but its combined tests did not compile.
Those earlier artifacts and receipts remain intact as **prior-attempt evidence
only**. The corrected source uses fresh staging and requires new version,
digest, and acceptance receipts. No earlier binary is overwritten or relabeled.

## Evidence boundaries and remaining issues

Build 40 acceptance is **local-only**: versioned init CLI/ConPTY checks and
synthetic source HTTP fixtures. Build 39 live-service evidence stays explicitly
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
