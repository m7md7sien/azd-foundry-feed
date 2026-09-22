# azd Foundry extension feed (unofficial)

Prerelease builds of two `azd` extensions, for internal bug bash only.

| Extension | Namespace | Source |
|---|---|---|
| `azure.ai.evaluations` | `azd ai eval` | [Azure/azure-dev#9500](https://github.com/Azure/azure-dev/pull/9500) |
| `azure.ai.dataset` | `azd ai dataset` | [Azure/azure-dev#9499](https://github.com/Azure/azure-dev/pull/9499) |

This is **not** an official Microsoft feed and is not affiliated with the
`azd` extension registry. It exists so testers can install a build without
compiling one. Both extensions are built from the source in the pull requests
above, which are public.

## Bug bash

Start here: **[Bugbash Instructions](./Bugbash-Instructions.md)** — setup, the
five hero scenarios, and where to file findings.

Closing out specific bugs from the last round?
**[Build 37 — verification pass](./Build-37-Verification.md)** lists what each
fix should now do, and which findings are already settled.

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
