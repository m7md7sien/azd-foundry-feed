# Build 39: source and acceptance checklist

**Published and promoted to Latest on 2026-09-23.** Local packaging, parent
source checks, scoped exact-package acceptance, final installed-CLI/full-race
CI, anonymous asset verification, and fresh pinned/Latest Windows installations
passed. Both extensions use the exact source SHA below. Build 38's release
assets and historical findings are unchanged.

**Current readiness qualification:** subsequent external reports identify a
high-priority simulation-init validation blocker and a reopened failed-run
reporting issue. The historical passes below remain scoped evidence, not a
blanket "bug-bash ready" or "all blockers fixed" declaration. Published packages
and versions have not changed.

| Item | Published value |
| --- | --- |
| Release | [extensions-2026-09-23-39](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-23-39) |
| Evaluations | `1.0.39-beta` |
| Dataset | `1.0.0-beta.27` |
| Required azd | `>=1.33.0`, confirmed in both packaged manifests and the registry |
| Source commit for both extensions | [`9a549449c2d5c2b4c6661f0ee8f8da7e3036c898`](https://github.com/m7md7sien/azure-dev/commit/9a549449c2d5c2b4c6661f0ee8f8da7e3036c898) |
| Registry SHA256 | `5fc9456319ad0f6408e36cb693e0a8007d750c5721011aea0badf88d68fb0c44` |
| Latest promotion | Completed after final hosted CI and anonymous/install gates; stable URL and fresh install verified |

## Newly reported known issues

These reports were received after the earlier scoped build 39 verification.
Fixes being prepared for a later candidate are not present in these immutable
packages. Tracking numbers below are issue identifiers, not GitHub issue links.

| Tracking | Status | Build 39 observation and boundary |
| --- | --- | --- |
| `5640927` | **High / P1, release blocker** | Simulation `init` accepts a blank or whitespace-only seed description or `desired_num_turns: 0` and writes configuration. `create` still rejects these inputs before publishing dependencies, but that protection does not fix the premature local writes by `init`. |
| `5595070` | **Reopened CLI reporting issue** | Failed-run summary/detail output is still reported to lack actionable follow-up commands. Follow-up synthetic HTTP-caller tests cover whole-run failure with absent or zero result counts, but are not execution of a real operationally failed service payload or a shipped build 40 fix. |
| `5631330` | **Known service cost blocker, open/unassigned** | A `simple_qna` generation request for 15 produced a reported result/file count of 16; the separate seed case returned 15. Updated service triage confirms that the merged patch does not cover the reported Q&A generation path. This is not merely an unconfirmed rollout. No client-side truncation remedy is claimed; generation counts are distinct from dataset-run caps. |
| `5595119` | **Deferred, not included** | The deprecated agent-hint change remains outside these packages. No fix or changed behavior is claimed here. |
| `5571322` | **Scoped published-39 retest passed** | An independent retest on the exact published build 39 passed Ctrl+C cancellation in the `run start` and `create` eval pickers without an ambiguity error. A nonzero exit is not required for intentional user cancellation. The earlier external pass omitted this case; that omission was a coverage gap, not a confirmed regression. This does not establish Escape cancellation. |
| `5572139` | **Backend cleanup issue remains open** | Terminal data-generation job deletion remains service-blocked. Historical HTTP 409 records were not cleared, and no retention TTL has been verified. |

Before simulation `init`, inspect each seed row: `test_case_description` must
contain non-whitespace text, and a supplied `desired_num_turns` must be a
positive whole number within an explicit maximum. Do not interpret successful
build 39 scaffolding as confirmation of those semantics. The existing
pre-publication `create` guard remains necessary, and no invalid input should
be treated as ready to run.

For failed-run investigation, the installed help supports manual inspection:

```bash
azd ai eval run output list --eval <eval-name-or-id> --run <run-id> --status failed,errored --all
azd ai eval run output show <output-item-id> --eval <eval-name-or-id> --run <run-id>
azd ai eval run output export <run-id> --eval <eval-name-or-id> --output-file results.json
```

Use IDs from the actual run/list output and retain sensitive result content
privately. These documented commands do not claim that the affected summaries
already print the required guidance. Build 40 is preparation-only until an exact
source SHA, build, and separate publication approval are provided.

## Install this build

Use a fresh `AZD_CONFIG_DIR` so an old development registry or installed binary
cannot mask the published package. The pinned source is independent of Latest:

```bash
azd extension source add -n foundry-candidate-39 -t url -l https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-39/registry.json
azd extension install azure.ai.evaluations --source foundry-candidate-39 --version 1.0.39-beta
azd extension install azure.ai.dataset --source foundry-candidate-39 --version 1.0.0-beta.27
azd ai eval version -o json
azd ai dataset version -o json
```

Changing a source does not automatically replace installed extensions. Use an
isolated configuration or explicitly upgrade; check an existing source's URL
instead of silently reusing the name.

[SHA256SUMS](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-39/SHA256SUMS)
and [source-provenance.json](https://github.com/m7md7sien/azd-foundry-feed/releases/download/extensions-2026-09-23-39/source-provenance.json)
describe the immutable artifacts. Provenance verification fields are a build-time
snapshot; the acceptance record below includes later results.

## Included source changes

The [six integrated follow-up commits](https://github.com/m7md7sien/azure-dev/compare/c5be500196d66bb4326bb62700a1dde83c1f92a5...9a549449c2d5c2b4c6661f0ee8f8da7e3036c898)
address:

- Single-file downloads in both `azd ai dataset` and `azd ai eval dataset`.
- Observed conversation-output identities and lifecycle statuses from a complete
  output listing, distinct from requested settings and evaluation verdicts.
- Generated init handoff guidance for unattended use.
- Truly unregistered local files whose remote version listing is empty.
- Actual returned rubric dimension data and preservation of nested result/sample
  fields, without deriving missing values from rubric definitions.
- Human detail lookup identifiers while preserving JSON service identities.

Each claim must be checked against the frozen integrated source and the exact
new packages. Build 38's known issues remain documented in
[its acceptance record](./Build-38-Verification.md). A source fix, passing
worker test, or earlier candidate run does not by itself verify build 39.

## Packaging and publication gates

Both extensions must come from one approved immutable commit, fetched into a
new artifacts-owned `b39-<source-sha-prefix>` directory. Do not reuse a previous
`bin`, bundle, registry, or installed development extension.

Keep source `go.mod`, `go.sum`, and changelogs unchanged. Version overrides are
limited to each extension's `extension.yaml` and `version.txt` in isolated
staging. Use the established `azd x build --all --skip-install` and `azd x pack`
format: six archives per extension, each containing `extension.yaml` and its
platform-named entrypoint. Windows/macOS use ZIP and Linux uses tar.gz.

| Gate | Status |
| --- | --- |
| Exact source SHA and BUILD approval | Approved for `9a549449c2d5c2b4c6661f0ee8f8da7e3036c898` |
| Parent combined source checks | Both modules passed full short suites with `NO_COLOR=1` and no skips, build, vet (including evaluation tags), zero lint issues, and clean `go fix` diff |
| Full hosted source-race suites | [Passed for both modules](https://github.com/m7md7sien/azure-dev/actions/runs/35810319528) at the exact source with unchanged Go 1.26.4 commands |
| Twelve archive layouts, manifests, entrypoints, SHA256 hashes, extracted binary bytes and VCS/platform metadata | Passed from fresh `b39-9a549449c2d5` staging; only four packaging version files changed |
| Fresh isolated Windows bundle installation and exact runtime versions | Passed with azd 1.33.0, exact JSON versions, and installed bytes matching the archives |
| Candidate-specific regression/live acceptance | Scoped pass on the exact packages: both download surfaces, local/registered static runs, rubric properties/sample/export, and lookup IDs; details below |
| Independent focused fresh-user follow-up | Completed on the published Windows amd64 packages using only public docs/help; no new functional defect was observed in that earlier scope. Subsequent external issues are listed above. |
| Actual Windows ConPTY checkpoint | Completed: 13 scoped cases on the published evaluation package, with final local process/fixture cleanup confirmed. Escape was not a cancellation pass. |
| Owned verification-fixture cleanup | Passed: owned eval, both runs, dataset version, and evaluator version returned 404; the local-only dataset remained unpublished |
| Separate PUBLISH approval | Approved for exact source and registry hash above, non-Latest first |
| Complete new draft: individually uploaded assets, no overwrite, literal `registry.json` last | Passed: all 15 remote sizes, SHA256 digests, and upload states matched |
| Non-Latest publication and anonymous pinned registry/all-asset verification | Passed: all 15 assets anonymously downloaded and SHA256 matched |
| Fresh published-source Windows install | Passed on azd 1.33.0; exact JSON versions and installed binary bytes match final archives |
| Final Linux/Windows CLI/source CI with exact published pins | [All four jobs passed](https://github.com/m7md7sien/azure-dev/actions/runs/35810319528): 68/68 unique actual installed-CLI checks per OS, both full race suites, exact registry/archive/installed bytes and versions; no skips |
| Latest promotion, anonymous stable URL, and fresh stable-source install | Passed: Latest registry matches approved SHA256; all 15 assets re-downloaded/hash-checked and a fresh Windows install matches versions and final bytes |

The [frozen CI manifest](https://github.com/m7md7sien/azure-dev/blob/0ce78ee5c10208e0f739b7aaa5b3496736c359c3/eng/scripts/eval-candidate-proof/candidate.json)
pins the source, registry, four Linux/Windows amd64 extension archives, and azd
1.33.0. Workflow commit `0ce78ee5c10208e0f739b7aaa5b3496736c359c3` is not the
extension source commit. Downloaded CI evidence was checked for unique commands,
exit codes, source/run identities, and sanitized fixtures. Live-cloud evaluation
and cloud quality-gate execution were **not run**.

The release should contain 12 platform archives, `source-provenance.json`,
`SHA256SUMS`, and the literal `registry.json`. Preserve every older release and
asset. Synchronize the repository registry/docs only with real published assets.
Report publication separately from Latest promotion.

Local build, packaging, and installation checks completed in 2.20 minutes.
That measured result is not a guarantee for later builds or a live-service
readiness claim.

## Candidate-specific acceptance targets

The matrix defines the acceptance scope. The recorded results below cover only
the measured cases, not every target or possible service response. Record exact
source, versions, package hashes, commands, and sanitized outcomes for each result.

| Area | Required evidence |
| --- | --- |
| Standalone and embedded dataset download | The affected single-file `--output-file` path writes the exact expected bytes in both namespaces. Container-backed single files require a complete one-file listing and `isSingleFile: true`; one-file folders still require directory output. Existing destination/force safeguards remain intact. |
| Truly unregistered local files | Exercise a direct declaration and a dataset override with confirmed remote absence and bounded rows. A valid complete empty listing requires confirming not-found lookups; malformed/incomplete listings and authorization/service failures must not enable inline fallback. Verify no implicit dataset publication. |
| Registered dataset identity and caps | Retain service-issued version identity and explicit positive-cap rejection. Do not replace registered data with an inline copy or automatically publish a temporary subset. |
| Observed conversation outputs | A waited run that already fetched all rows reports unique conversation IDs and output-item lifecycle statuses. Duplicates count once; unknown/conflicting statuses and rows without IDs stay separate. Paged/filtered or unfetched detail views must not imply complete coverage. No new fetch, generated/completed-conversation total, or transcript-based actual-turn inference is promised. |
| Rubric detail and raw fields | Display dimension values only when actually returned. Verify applicable scores, weights and reasons against sanitized retained service evidence; preserve nested JSON fields rather than silently dropping them. |
| Detail lookup identifiers | A human-displayed lookup ID works with the detail command; JSON retains the service identity. |
| Unattended handoff | Confirm generated interactive `init` guidance explains the explicit independent judge/simulator inputs required with `--no-prompt`, without contaminating JSON output or resubmitting generation. |
| Existing behavior | Retain the prior semantic preflight/no-mutation, static-versus-simulation, pinning, cap-zero, and JSON-output guarantees using evidence tied to the candidate or an explicitly justified source boundary. |

The approved README specifies that rubric detail uses returned
`properties.dimension_scores`, including available scores, applicability,
weights, and reasons. Applicability is not a pass/fail verdict and missing values
must not become zero or false. JSON retains nested `properties` and `sample`
fields; these may contain sensitive prompts and answers, so prefer a private
output file rather than shared terminal or CI logs. The two-dimension case below
has candidate-specific evidence; it is not proof for the deleted build 38
fresh-user run.

### Recorded scoped package result

The verifier independently matched the bundle and installed binary hashes,
versions, embedded `9a549449c2d5c2b4c6661f0ee8f8da7e3036c898` VCS revision,
and registry hash `5fc9456319ad0f6408e36cb693e0a8007d750c5721011aea0badf88d68fb0c44`
in a fresh configuration, leaving build 38 untouched.

Both dataset command namespaces passed exact-file/source-byte checks, refusal
to overwrite without force, forced replacement, and directory output. Separate
recorded tests passed seven cases per module covering folder/multiple-file
guards; those are not live tests of every dataset shape.

Two bounded static runs scored three rows with zero errors: a genuinely local
inline dataset capped at one row and an uncapped registered dataset with two
rows. The registered positive-cap refusal and absence of implicit local-dataset
publication passed. No data generation or agent invocation was performed.

For a returned two-dimension rubric result, the typed JSON and export retained
the actual service properties and sample fields exactly. The human numeric
lookup ID and both dimensions' numeric values, applicability, and full reasons
matched the retained service evidence. The sensitive result content is not
included here. This result verifies that measured shape, not unseen rubric or
GA service cases. Cleanup was confirmed by 404 responses for the owned eval,
both runs, registered dataset version, and evaluator version. The local-only
dataset remained unpublished; no agent or conversation resources were created.

Use small owned fixtures and bounded model calls; do not regenerate large
datasets merely to replace provenance. Delete only resources proved to belong
to the test. Prior build 38 runs and its 64-check-per-platform CI remain
historical evidence, not new build 39 executions.

### Independent focused fresh-user follow-up

The public-docs/help-only follow-up completed on a fresh isolated Windows amd64
installation with azd 1.33.0, evaluations `1.0.39-beta`, and dataset
`1.0.0-beta.27`. The pinned registry SHA256 and installed binary digests matched
this release and were rechecked. Usage documentation was pinned to
[public revision `9c6381e`](https://github.com/m7md7sien/azd-foundry-feed/blob/9c6381e22943ae2bf5f0736647e08d35bb3b3237/Bugbash-Instructions.md).
The tester used public provenance for the source identity, not source inspection
in this lane. **No new functional defect was observed in that earlier focused
scope.** The later external reports above are additional findings, not erased
by this historical result.

| Area rerun on build 39 | Observed result |
| --- | --- |
| Both dataset namespaces | Exact-file and directory output matched source bytes; overwrite refusal and forced replacement passed. |
| Unregistered local inputs | Direct-declaration and dataset-override runs each completed with a one-row cap. Remote absence was checked before and after; neither implicitly published a dataset. |
| Registered identity and caps | Positive caps were refused. Explicit zero overrode a positive YAML cap while retaining service-issued `file_id` and version. Three static/manual-rubric rows total were scored across these cases. |
| Rubric detail and JSON | Human detail displayed returned scores, applicability, weights and reasons; a null score was `not reported`. Detail/list/export result objects were semantically equal for the measured properties/sample fields. Lookup ID `1` worked while JSON retained the service URI. |
| Bounded simulation | One seed, one repetition, maximum two turns: one passed evaluation, zero errors, and two user plus two assistant messages in the owned transcript. |
| Observed-output summary | The waited run reported one identified conversation ID with completed output, separately from quality verdicts. Generated/completed-conversation totals and aggregate actual turns stayed `not reported`. Later `list --all` lacked that section, a recorded surface distinction rather than an established contract violation. |
| Unattended model inputs | Public guidance/help and non-billable missing/explicit judge/simulator input validation passed. No new generation was submitted, so newly printed post-generation guidance was not executed. |
| Owned-fixture cleanup | Both evals and their four runs, two dataset versions, and one rubric version were removed with absence checks. No build 39 cleanup failure was observed. Historical build 38 HTTP 409 job records were not retried or claimed cleared. |

This was **not an all-scenarios build 39 rerun**. Broader agent-version, trace,
deployment, CRUD/versioning, gating, and cancellation journeys remain build 38
evidence. This follow-up was not authenticated live-cloud CI, an all-platform
test, a direct raw-REST preservation audit, GA-contract confirmation, or privacy
signoff. The original deleted build 38 run's raw shape remains unverified.

### Actual Windows ConPTY checkpoint

A separate interactive checkpoint completed on Windows amd64 with azd 1.33.0
and the unchanged published evaluations `1.0.39-beta` package from the pinned
source. **Thirteen scoped cases passed:** Simulation/Static and Turn pickers;
independent simulator/judge text inputs and a local judge-deployment picker;
evaluator selection and Add confirmation; explicit minimum `1/1` and maximum
`5/20` bounds and preservation of an omitted maximum; rejection of out-of-range
`0`, `6`, and `21` before prompting; Ctrl+C at the mode and simulator prompts;
explicit Cancel; and JSON-only stdout on a real console. Add-only YAML behavior
preserved existing entries and comments.

Those numeric-bound cases exercised flags and authored configuration paths;
they did not establish rejection of every invalid seed-row value during
simulation `init`. In particular, they do not close the newly reported
`desired_num_turns: 0` seed-row/local-write blocker.

This was a partially specified CLI workflow: source, name, dataset, target,
and some levels/bounds were supplied as flags. Numeric limits were **flags, not
interactive prompts**. JSON mode intentionally did not prompt even on a true
console; stderr was captured separately. Cancellation preserved all preexisting
authored files; a normal zero-byte environment lock file was the only new core
lock artifact, not an authored configuration change. Escape did not cancel the
tested picker during a bounded 12-second observation, so **Escape cancellation is unsupported/unproven here,
not passed**. This is not an all-platform or every-keyboard-path result.

The final cleanup receipt confirmed that all 14 owned PTY processes exited,
with zero owned descendants or extension processes remaining. New private
fixtures and the temporary virtual environment were removed; preexisting
configuration/binary hashes and the source tree were unchanged. These checks
created no Azure evaluation runs, generation jobs, or agent resources and used
no shared prompt data. Only this public-safe scope summary is published, not
the private reports or terminal captures.

## Remaining qualifications

### Public GA proposal versus current deployment

The public proposal [Azure/azure-rest-api-specs#45904](https://github.com/Azure/azure-rest-api-specs/pull/45904),
reviewed at
[`8363fd2a7898143ac8a06ea573faf74035c10a62`](https://github.com/Azure/azure-rest-api-specs/blob/8363fd2a7898143ac8a06ea573faf74035c10a62/specification/ai-foundry/data-plane/Foundry/src/openai/evaluations/user_conversation_simulation.tsp),
names the run discriminator `azure_ai_user_conversation_simulation`.
A separate, non-executing schema probe against the current bug-bash target
rejected that GA name as unknown and listed
`azure_ai_user_conversation_simulation_preview` among accepted types. The probe
did not supply resource/model inputs or create an evaluation run.

Build 39 therefore retains the preview run wire used by the previously scored
bug-bash simulations. A public proposal is not evidence that this target has
deployed the GA contract. Contract/deployment alignment remains an external
gate; do not unconditionally replace the wire discriminator or claim GA
readiness. This distinction concerns the run contract, not seed generation's
separate `simulation_seed` discriminator.

The reviewed proposal describes `desired_num_turns` as a **target** and the
maximum turn setting as a **hard limit**, not a promise of an exact observed
length. Earlier termination is allowed and does not by itself establish a CLI
bug. Record actual transcript observations separately from requested settings.

The scoped build 39 follow-up is complete, but does not establish an
all-scenarios rerun, authenticated live-cloud CI pass, final GA simulation
contract, privacy signoff, or service-fix deployment. Required AA and upstream
PR-review approvals also remain separate external gates; test results do not
grant those approvals. Keep unsupported or unverified behavior explicit.

Broader fresh-user scenario work belongs to build 38, including its failures
and gaps; build 39 has the completed targeted package and independent focused
follow-up evidence above, not a full rerun. Two historical terminal
data-generation job records
returned HTTP 409 when deletion was attempted. These records were neither
created nor retried by this candidate's tests. Their removal remains backend
cleanup work; no service-history cleanup fix or verified retention TTL is claimed.

Public files must contain no credentials, private prompts or handoff material,
customer data, or full raw service responses. Publish sanitized field/shape
comparisons and precise verification scope instead.
