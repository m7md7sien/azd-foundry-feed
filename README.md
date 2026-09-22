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

The currently published build is
[37](https://github.com/m7md7sien/azd-foundry-feed/releases/tag/extensions-2026-09-22-37):
evaluations `1.0.37-beta` and dataset `1.0.0-beta.25`. It bundles
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116),
[Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113), and
[Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102).
No newer candidate is published or verified by this document.

## Bug bash

Start here: **[Bugbash Instructions](./Bugbash-Instructions.md)** for setup,
hero scenarios, full YAML authoring, and where to file findings.

Closing out specific bugs from the last round?
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
