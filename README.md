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
[38](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-23-38):
evaluations `1.0.38-beta` and dataset `1.0.0-beta.26`, both built from
[`c5be500196d66bb4326bb62700a1dde83c1f92a5`](https://github.com/m7md7sien/azure-dev/commit/c5be500196d66bb4326bb62700a1dde83c1f92a5).
It bundles
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116),
[Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113), and
[Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102).
plus the follow-up fixes described in the
[build 38 acceptance record](./Build-38-Verification.md).

**Build 38 is Latest.** Anonymous downloads and fresh pinned/Latest Windows
installs passed. [Final CI](https://github.com/m7md7sien/azure-dev/actions/runs/35802906993)
passed 64/64 installed-CLI checks on each of Linux and Windows, plus both full
source-race suites. Use the stable Latest URL below or the
[pinned build 38 instructions](./Build-38-Verification.md#install-this-build).

## Bug bash

Start here: **[Bugbash Instructions](./Bugbash-Instructions.md)** for setup,
hero scenarios, full YAML authoring, and where to file findings.

For current candidate evidence and known limitations, use
**[Build 38 verification](./Build-38-Verification.md)**. The standalone dataset
`--output-file` issue has a verified `--output-dir` workaround. Richer observed
simulation counters and an all-scenarios fresh-user pass remain follow-up work.
Hosted source/offline CLI checks are not live-cloud CI approval.

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
