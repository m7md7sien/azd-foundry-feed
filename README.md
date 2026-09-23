# azd Foundry extension feed (unofficial)

Prerelease builds of two `azd` extensions, for internal bug bash only.

| Extension | Namespace | Original proposal |
|---|---|---|
| `azure.ai.evaluations` | `azd ai eval` | [Azure/azure-dev#9500](https://github.com/Azure/azure-dev/pull/9500) |
| `azure.ai.dataset` | `azd ai dataset` | [Azure/azure-dev#9499](https://github.com/Azure/azure-dev/pull/9499) |

This is **not** an official Microsoft feed and is not affiliated with the
`azd` extension registry. It exists so testers can install a build without
compiling one. The original proposals above are not sufficient source provenance
for a later build. Each release identifies its bundled changes; a PR bundled into
this feed is **not necessarily merged upstream**.

The newest published candidate is
[40](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-23-40):
evaluations `1.0.40-beta` and dataset `1.0.0-beta.28`, both built from
[`361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a`](https://github.com/m7md7sien/azure-dev/commit/361ca3c338069452a0bc1da6aa5a7b7c4e8bcf6a).
It bundles
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116),
[Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113), and
[Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102)
plus the follow-up fixes described in the
[build 40 acceptance record](./Build-40-Verification.md).

**Build 40 is now Latest.** Use the
[pinned build 40 instructions](./Build-40-Verification.md#install-this-build)
for reproducible installation. [Final hosted CI](https://github.com/m7md7sien/azure-dev/actions/runs/35826828347)
passed 124 installed CLI checks on each of Linux and Windows plus both full
source-race suites. The stable Latest registry, all 15 anonymous downloads, and
a fresh Latest-source Windows install matched the approved bytes. The
[acceptance record](./Build-40-Verification.md#final-hosted-proof) includes the
precise first-read zero-byte `.env.lock` allowance and scoped evidence limits.

**New confirmed build 40 issue:** a responses-backed evaluation
(`source.responses`) failed with zero output rows and a `response_id` mapping
error. This path is not working in the observed case; a correction is being
prepared, but no fixed replacement package has been verified or published.
Read the [runtime failure and diagnostic scope](./Build-40-Verification.md#confirmed-responses-backed-runtime-failure).
The earlier scoped passes do not establish that all build 40 scenarios passed.

Build 40 corrects build 39's high-priority simulation-init validation/local-write
defect, including interactive correction and mixed seed fields. Its exact Windows
packages passed 31 CLI cases and two actual ConPTY paths. Terminal failed/errored
run guidance originally had 15-group synthetic HTTP/source evidence; those
fixtures were not live-service proof. Diagnostic follow-up commands also
worked for the later observed responses-backed failure, without establishing
coverage of every operational-failure or errored-row shape. These fixes do not alter
[build 39's immutable packages or known issues](./Build-39-Verification.md#newly-reported-known-issues).
The reported Q&A generation count/cost blocker and terminal-job deletion remain
backend-open; deprecated agent-hint work remains deferred.

## Bug bash

Start here: **[Bugbash Instructions](./Bugbash-Instructions.md)** for setup,
hero scenarios, full YAML authoring, and where to file findings.

For current candidate evidence and known limitations, use
**[Build 40 verification](./Build-40-Verification.md)**. In addition to its
local/offline acceptance, a subsequent
[bounded live regression](./Build-40-Verification.md#bounded-published-package-live-regression)
completed two graded rows with no data-generation jobs and confirmed cleanup.
The simulation completed with a failed quality verdict and zero execution
errors; the static row passed with two returned rubric dimensions. This is
not an operationally failed-service replay or an all-scenarios rerun.
GA contract/deployment alignment, privacy/AA, upstream review and authenticated
live-cloud CI remain external gates. There is no unconditional readiness claim.

**[Build 39 verification](./Build-39-Verification.md)** retains its
[final 68-check-per-OS and full-race CI](https://github.com/m7md7sien/azure-dev/actions/runs/35810319528).
Its targeted exact-package
checks cover both single-file download surfaces, local-file caps, registered
identity guards, and preservation/rendering of returned rubric details.
Observed conversation-output identities/statuses are not generation totals or
inferred actual turns. An independent public-docs/help-only **focused build 39
fresh-user follow-up completed with no new functional defect observed in that
earlier measured scope**. Subsequent external reports are recorded separately
in the known issues above. Its scoped results and not-rerun limits remain in
the acceptance record. Broader
journeys remain build 38 evidence, not a complete build 39 rerun. A separate
actual Windows ConPTY checkpoint passed 13 scoped interactive cases and finished
process/fixture cleanup; Escape cancellation was not established. Hosted
source/offline checks are not live-cloud CI.

**[Build 38 verification](./Build-38-Verification.md)** retains that immutable
release's evidence and known issues, including its download workaround,
unregistered-local-file failure, and rubric projection gap. Its assets are
unchanged.

Closing out specific bugs in the older build?
**[Build 37 verification checklist](./Build-37-Verification.md)** distinguishes
expected behavior from historical evidence and remaining verification gates.
Installability and source tests do not establish live feature readiness.

## Use it

The source URL below follows GitHub's **Latest** release, so it does not change
with each build.

```bash
azd extension source add -n foundry-bugbash -t url \
  -l https://github.com/m7md7sien/azd-foundry-feed/releases/latest/download/registry.json

azd extension install azure.ai.evaluations --source foundry-bugbash
azd extension install azure.ai.dataset --source foundry-bugbash
```

If you added this source before and pinned it to a dated release, remove it once
with `azd extension source remove foundry-bugbash` and add it again as above.

## Stop using it

```bash
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension source remove foundry-bugbash
```

Requires `azd` **1.33.0 or later**. Both extensions declare
`requiredAzdVersion: >=1.33.0` in the published registry, so an older `azd`
will not resolve this build at all.
