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
[39](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-23-39):
evaluations `1.0.39-beta` and dataset `1.0.0-beta.27`, both built from
[`9a549449c2d5c2b4c6661f0ee8f8da7e3036c898`](https://github.com/m7md7sien/azure-dev/commit/9a549449c2d5c2b4c6661f0ee8f8da7e3036c898).
It bundles
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116),
[Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113), and
[Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102)
plus the follow-up fixes described in the
[build 39 acceptance record](./Build-39-Verification.md).

**Build 39 is Latest.** [Final CI](https://github.com/m7md7sien/azure-dev/actions/runs/35810319528)
passed 68/68 installed-CLI checks on each of Linux and Windows, plus both full
source-race suites. Anonymous asset checks and fresh pinned/Latest Windows
installs matched approved bytes. Use the
[pinned build 39 instructions](./Build-39-Verification.md#install-this-build)
or the stable Latest URL below.

## Bug bash

Start here: **[Bugbash Instructions](./Bugbash-Instructions.md)** for setup,
hero scenarios, full YAML authoring, and where to file findings.

For current candidate evidence and known limitations, use
**[Build 39 verification](./Build-39-Verification.md)**. Targeted exact-package
checks cover both single-file download surfaces, local-file caps, registered
identity guards, and preservation/rendering of returned rubric details.
Observed conversation-output identities/statuses are not generation totals or
inferred actual turns. An independent public-docs/help-only **focused build 39
fresh-user follow-up completed with no new functional defect observed**.
Its scoped results and not-rerun limits are in the acceptance record. Broader
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
