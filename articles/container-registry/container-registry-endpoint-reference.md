---
title: Azure Container Registry endpoint reference
description: Reference for all Azure Container Registry endpoint types, including global, regional, and dedicated data endpoints, along with CLI flag details.
author: johnsonshi
ms.topic: reference
ms.author: johsh
ms.service: azure-container-registry
ms.date: 05/27/2026
# Customer intent: "As a developer or network administrator, I want to understand all endpoint types for Azure Container Registry and the CLI flags that control them."
---

# Azure Container Registry endpoint reference

Azure Container Registry exposes several endpoint types depending on your registry's configuration. This article provides a reference for each endpoint type, how to view them, and the CLI flags that control them.

For information on geo-replication and regional endpoints, see [Geo-replication in Azure Container Registry](container-registry-geo-replication.md). For information on dedicated data endpoints, see [Dedicated data endpoints in Azure Container Registry](container-registry-dedicated-data-endpoints.md).

## Endpoint types

| Endpoint type | URL format | Purpose | Use case |
|---------------|------------|---------|----------|
| Global endpoint | `myregistry.azurecr.io` | Login server with Azure-managed routing to any geo-replica | Default; optimal for most scenarios |
| Regional endpoint | `myregistry.<region-name>.geo.azurecr.io` | Login server for a specific geo-replica | Predictable routing, client-side failover, regional affinity, troubleshooting |
| Dedicated data endpoint | `myregistry.<region-name>.data.azurecr.io` | Layer blob downloads for registries with dedicated data endpoints or private endpoints | Scoped firewall rules; automatic 307 redirect from login server during pulls |
| Storage account | `*.blob.core.windows.net` | Layer blob downloads for registries without dedicated data endpoints or private endpoints | Automatic redirect from login server |

## View all endpoints

Use the `az acr show-endpoints` command to view all endpoints for your registry, including the global URL, regional endpoints (if enabled), and dedicated data endpoints (if enabled):

```azurecli
az acr show-endpoints --name myregistry --resource-group myrg
```

Example output for a registry with regional endpoints and dedicated data endpoints enabled:

```output
{
    "loginServer": "myregistry.azurecr.io",
    "dataEndpoints": [
        {
            "region": "eastus",
            "endpoint": "myregistry.eastus.data.azurecr.io"
        },
        {
            "region": "westus",
            "endpoint": "myregistry.westus.data.azurecr.io"
        }
    ],
    "regionalEndpoints": [
        {
            "region": "eastus",
            "endpoint": "myregistry.eastus.geo.azurecr.io"
        },
        {
            "region": "westus",
            "endpoint": "myregistry.westus.geo.azurecr.io"
        }
    ]
}
```

## CLI flags for registry endpoints

> [!IMPORTANT]
> Several CLI flags control different endpoint behaviors. Understanding the distinction is important to avoid misconfiguration.

| Flag | Scope | Purpose |
|------|-------|---------|
| `--regional-endpoints` | Registry-level (on `az acr create` or `az acr update`) | Enables dedicated regional endpoint URLs (`myregistry.<region>.geo.azurecr.io`) for **all** geo-replicas. |
| `--global-endpoint-routing` | Per-geo-replica (on `az acr replication create` or `az acr replication update`) | Controls whether the **global endpoint** (`myregistry.azurecr.io`) routes traffic to a specific geo-replica. Set to `false` to temporarily exclude a geo-replica from global endpoint routing (for maintenance or troubleshooting). Data continues syncing regardless of this setting. See [Temporarily exclude a geo-replica from global endpoint routing](container-registry-geo-replication.md#temporarily-exclude-a-geo-replica-from-global-endpoint-routing). |
| `--data-endpoint-enabled` | Registry-level (on `az acr create` or `az acr update`) | Enables dedicated data endpoints (`myregistry.<region>.data.azurecr.io`) for layer blob downloads. Auto-enabled when at least one private endpoint is configured. See [Dedicated data endpoints](container-registry-dedicated-data-endpoints.md). |

### Relationship between `--regional-endpoints` and `--global-endpoint-routing`

`--regional-endpoints` and `--global-endpoint-routing` are independent. Setting `--global-endpoint-routing false` on a geo-replica:

- Excludes that geo-replica from **global endpoint** routing only.
- Does **not** disable the geo-replica's **regional endpoint** URL. If `--regional-endpoints` is enabled at the registry level, clients can still directly access that geo-replica via the regional endpoint URL.
- Does **not** stop data syncing to that geo-replica.

**In short:**

- Use `--regional-endpoints` at the registry level to **enable dedicated regional URLs** for direct access to specific geo-replicas.
- Use `--global-endpoint-routing` on a specific geo-replica to **control global endpoint routing** to that geo-replica.
- Use `--data-endpoint-enabled` at the registry level to **enable dedicated data endpoints** for scoped layer blob downloads.

## Related content

- [Geo-replication in Azure Container Registry](container-registry-geo-replication.md)
- [Dedicated data endpoints in Azure Container Registry](container-registry-dedicated-data-endpoints.md)
- [Access a registry behind a firewall](container-registry-firewall-rules.md)
