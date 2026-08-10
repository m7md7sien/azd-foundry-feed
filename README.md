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

## Use it

```bash
azd extension source add -n foundry-bugbash -t url \
  -l https://raw.githubusercontent.com/m7md7sien/azd-foundry-feed/main/registry.json

azd extension install azure.ai.evaluations
azd extension install azure.ai.dataset
```

## Stop using it

```bash
azd extension uninstall azure.ai.evaluations
azd extension uninstall azure.ai.dataset
azd extension source remove foundry-bugbash
```

Requires `azd` 1.27.0 or later.
