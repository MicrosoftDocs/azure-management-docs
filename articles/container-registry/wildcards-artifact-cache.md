---
title: "Wildcard support for artifact cache in Azure Container Registry"
description: "Use wildcards to match multiple paths within the container image registry. Artifact cache currently supports registry and repository level wildcards."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: concept-article #Don't change
ms.custom: devx-track-azurecli
ms.date: 04/18/2025
ai-usage: ai-assisted
# Customer intent: As a developer, I want to configure wildcard rules for artifact caching in a container registry, so that I can efficiently manage and retrieve container images across multiple repositories.
---

# Wildcard support for artifact cache in Azure Container Registry

Wildcard use asterisks (*) to match multiple paths within the container image registry. This article lists the wildcards supported by the [artifact cache feature](artifact-cache-overview.md) for Azure Container Registry (ACR).

> [!NOTE]
> Cache rules map from `target repository` => `source repository`.

## Registry level wildcard

The registry level wildcard lets you cache all repositories from an upstream registry.

| Cache rule                                  | Mapping                                  | Example                                                                                                                                |
| ------------------------------------------- | ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `contoso.azurecr.io/*` => `mcr.microsoft.com/*` | Mapping for all images under ACR to MCR. | `contoso.azurecr.io/myapp/image1` => `mcr.microsoft.com/myapp/image1<br>contoso.azurecr.io/myapp/image2` => `mcr.microsoft.com/myapp/image2` |

## Repository level wildcard

The repository level wildcard lets you cache all repositories from an upstream registry mapping to the repository prefix.

| Cache Rule                                                                                                                              | Mapping                                                                                     | Example                                                                                                                                            |
| --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `contoso.azurecr.io/dotnet/*` => `mcr.microsoft.com/dotnet/*`                                                                               | Mapping specific repositories under ACR to corresponding repositories in MCR.               | `contoso.azurecr.io/dotnet/sdk` => `mcr.microsoft.com/dotnet/sdk`<br>`contoso.azurecr.io/dotnet/runtime` => `mcr.microsoft.com/dotnet/runtime`             |
| `contoso.azurecr.io/library/dotnet/*` => `mcr.microsoft.com/dotnet/*` <br>`contoso.azurecr.io/library/python/*` => `docker.io/library/python/`* | Mapping specific repositories under ACR to repositories from different upstream registries. | `contoso.azurecr.io/library/dotnet/app1` => `mcr.microsoft.com/dotnet/app1` <br>`contoso.azurecr.io/library/python/app3` => `docker.io/library/python/app3` |

## Limitations for wildcard-based cache rules

Wildcard cache rules use asterisks (*) to match multiple paths within the container image registry. These rules can't overlap with other wildcard cache rules by targeting the same repository path, or by targeting multiple paths where one path includes the other. In other words, if you have a wildcard cache rule for a certain registry path, you can't add another wildcard rule that overlaps with it.

Here are some examples of overlapping rules:

**Example 1**:

Existing cache rule: `contoso.azurecr.io/* => mcr.microsoft.com/*`<br>
New cache rule: `contoso.azurecr.io/library/* => docker.io/library/*`<br>

The addition of the new cache rule is blocked because the target repository path `contoso.azurecr.io/library/*` overlaps with the existing wildcard rule `contoso.azurecr.io/*`.

**Example 2:**

Existing cache rule: `contoso.azurecr.io/library/*` => `mcr.microsoft.com/library/*`<br>
New cache rule: `contoso.azurecr.io/library/dotnet/*` => `docker.io/library/dotnet/*`<br>

**Example 3:**

Existing cache rule: `contoso.azurecr.io/* => mcr.microsoft.com/*`<br>
New cache rule: `contoso.azurecr.io/* => docker.io/library/*`<br>

The addition of the new cache rule is blocked because two rules use the same target repository path `contoso.azurecr.io/*`.

The addition of the new cache rule is blocked because the target repository path `contoso.azurecr.io/library/dotnet/*` overlaps with the existing wildcard rule  `contoso.azurecr.io/library/*`.

## Limitations for static/fixed cache rules

Static or fixed cache rules are more specific and don't use wildcards. They can potentially overlap with wildcard-based cache rules. If a cache rule specifies a fixed repository path, then it allows overlapping with a wildcard-based cache rule. However, you can't create multiple static rules that point to the same exact repository path.

**Example 1**:

Existing cache rule: `contoso.azurecr.io/*` => `mcr.microsoft.com/*`<br>
New cache rule: `contoso.azurecr.io/library/dotnet` => `docker.io/library/dotnet`<br>

The addition of the new cache rule is allowed because `contoso.azurecr.io/library/dotnet` is a static path and can overlap with the wildcard cache rule `contoso.azurecr.io/*`.

**Example 2:**

Existing cache rule: `contoso.azurecr.io => mcr.microsoft.com/*`<br>
New cache rule: `contoso.azurecr.io => docker.io/library/*`<br>

The addition of the new cache rule is blocked because two rules use the same target repository path `contoso.azurecr.io`.

## Next steps

- Learn more about the [artifact cache feature](artifact-cache-overview.md).
- Get help [troubleshooting artifact cache issues](troubleshoot-artifact-cache.md).
