---
title: "Optimize image pulls with artifact cache in Azure Container Registry"
description: "Artifact cache is a feature that allows you to cache container images in Azure Container Registry, improving performance and efficiency."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: conceptual #Don't change
ms.custom: devx-track-azurecli
ms.date: 04/23/2025
ai-usage: ai-assisted
#customer intent: As a developer, I want Artifact cache capabilities so that I can efficiently deliver and serve containerized applications to end-users in real-time.
---

# Optimize image pulls with artifact cache in Azure Container Registry

The artifact cache feature of Azure Container Registry lets you cache container images in both public and private repositories.

Artifact cache enables faster and more *reliable pull operations* through Azure Container Registry (ACR). It uses features like geo-replication and availability zone support for higher availability and faster image pulls. You can access cached registries over private networks to align with your firewall configurations and compliance standards.

Artifact cache addresses the challenge of pull limits imposed by public registries. We recommend authenticating your cache rules with your upstream source credentials. Then, you can pull images from the local ACR, helping to mitigate rate limits.

The artifact cache feature is available in *Basic*, *Standard*, and *Premium* [service tiers](container-registry-skus.md). You can enable artifact cache rules in the [Azure portal](artifact-cache-portal.md) or by using [Azure CLI](artifact-cache-cli.md).

## Terminology

When working with artifact caching, it's helpful to understand the following terminology:

- **Cache Rule**: A rule you create to pull artifacts from a supported repository into your cache. A cache rule contains four parts:

  - **Rule name**: The name of your cache rule. For example, `Hello-World-Cache`.
  - **Source**: The name of the source registry.
  - **Repository path**: The source path of the repository to find and retrieve artifacts you want to cache. For example, `docker.io/library/hello-world`.
  - **New ACR repository namespace**: The name of the new repository path to store artifacts. For example, `hello-world`. The repository can't already exist inside the ACR instance.

- **Credentials**: A username and password set for the source registry. You require credentials to authenticate with a public or private repository. Credentials contain four parts:

  - **Credentials**: The name of your credentials.
  - **Source registry login server**: The login server of your source registry.
  - **Source authentication**: The key vault locations to store credentials.
  - **Username and password secrets**: Secrets containing the username and password.

## Current limitations

When using artifact cache, keep in mind the following limitations:

- Cache only occurs after at least one image pull is complete on the available container image. For every new image available, a new image pull must be complete. Currently, artifact cache doesn't automatically pull new tags of images when a new tag is available.
- Artifact cache supports a maximum of 1,000 cache rules.
- Artifact cache rules can't overlap with other cache rules. In other words, if you have an artifact cache rule for a certain registry path, you can't add another cache rule that overlaps with it.

## Upstream support

Artifact cache currently supports the following upstream registries. Review the following table for details about which types of pulls are supported and how to use them.

>[!WARNING]
> To source content from Docker Hub, you must generate a credential set by using [Azure CLI](artifact-cache-cli.md#create-the-credentials) or the [Azure portal](artifact-cache-portal.md#create-new-credentials).

| Upstream registry                          | Support                                                  | Availability             |
|----------------------------------------------|----------------------------------------------------------|--------------------------|
| Docker Hub                                   | Supports authenticated pulls only.                       | Azure CLI, Azure portal  |
| Microsoft Artifact Registry                  | Supports unauthenticated pulls only.                     | Azure CLI, Azure portal  |
| AWS Elastic Container Registry (ECR) Public Gallery | Supports unauthenticated pulls only.              | Azure CLI, Azure portal  |
| GitHub Container Registry                    | Supports both authenticated and unauthenticated pulls.   | Azure CLI, Azure portal  |
| Quay                                         | Supports both authenticated and unauthenticated pulls.   | Azure CLI, Azure portal  |
| `registry.k8s.io`                              | Supports both authenticated and unauthenticated pulls.   | Azure CLI                |
| Google Container Registry                    | Supports both authenticated and unauthenticated pulls.   | Azure CLI                |

- The login server for Docker Hub is `docker.io`.
- The login server for Microsoft Artifact Registry is `mcr.microsoft.com`.

The Azure portal autofills these fields for you, including adding `library/` if required for Docker repositories.

## Next steps

- Learn how to enable artifact caching using the [Azure portal](artifact-cache-portal.md) or [Azure CLI](artifact-cache-cli.md).
- Learn about using [wildcards](wildcards-artifact-cache.md) to match multiple paths within the container image registry.
