---
title: Dedicated data endpoints for Azure Container Registry
description: Learn how dedicated data endpoints in Azure Container Registry enable scoped firewall rules and mitigate data exfiltration risks for container image layer downloads.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 05/27/2026
# Customer intent: As a security specialist, I want to implement dedicated data endpoints in Azure Container Registry, so that I can minimize data exfiltration risks and ensure secure access to storage resources.
---
# Dedicated data endpoints in Azure Container Registry

Dedicated data endpoints in Azure Container Registry provide scoped, registry-specific URLs for container image layer downloads, replacing the default wildcard Azure storage account endpoints (`*.blob.core.windows.net`). This enables tightly scoped client firewall rules and mitigates data exfiltration risks.

> [!NOTE]
> Dedicated data endpoints are only used during layer blob **downloads** (pulls). When uploading blobs during pushes, they go via the login server (global endpoint or regional endpoint), not the dedicated data endpoint.

The dedicated data endpoints feature is available for registries in the **Premium** [service tier](container-registry-skus.md).

## How container image downloads work: login server and 307 redirect to data endpoints

Pulling content from a registry involves two types of endpoints:

1. **Login server endpoint** — used for authentication and content discovery. The login server can be either:
   - The **global endpoint** (`contoso.azurecr.io`) — ACR automatically routes requests to the geo-replica with the best network performance profile for the client. For more information, see [geo-replication](container-registry-geo-replication.md).
   - A **regional endpoint** (`contoso.eastus.geo.azurecr.io`) — routes requests directly to a specific geo-replica, bypassing Azure-managed routing. For more information, see [geo-replication](container-registry-geo-replication.md).

   When you run a command like `docker pull contoso.azurecr.io/hello-world` (global endpoint) or `docker pull contoso.eastus.geo.azurecr.io/hello-world` (regional endpoint), the client authenticates against the login server and negotiates which layers make up the requested artifact.
2. **Data endpoint** — used for downloading the actual image layer blobs. After the login server identifies the required layers, it redirects the client to download them from a data endpoint.

:::image type="content" source="./media/dedicated-data-endpoints/endpoints.png" alt-text="Diagram to illustrate endpoints.":::

ACR manages the underlying storage accounts for data endpoints. Benefits include load balancing, content splitting across storage accounts for higher concurrent delivery, and multi-region support with [geo-replication](container-registry-geo-replication.md).

### Without dedicated data endpoints

Without dedicated data endpoints, the login server issues a 307 redirect for layer blob downloads to Azure storage accounts using wildcard URLs (`*.blob.core.windows.net`).

**Using the global endpoint (Azure-managed routing):**

```
Client
  │
  ▼
contoso.azurecr.io (global endpoint: auth and data plane APIs)
  │
  ▼ Azure-managed routing
Geo-replica with best network performance profile
  │
  ▼ 307 redirect
*.blob.core.windows.net (layer blob downloads)
```

**Using a regional endpoint (client-specified routing):**

```
Client
  │
  ▼
contoso.eastus.geo.azurecr.io (regional endpoint: auth and data plane APIs)
  │
  ▼ Direct to specific geo-replica
East US geo-replica
  │
  ▼ 307 redirect
*.blob.core.windows.net (layer blob downloads)
```

In both cases, to allow these downloads through a firewall, you must permit `*.blob.core.windows.net` — a broad wildcard that covers all Azure storage accounts, not just your registry's storage.

### With dedicated data endpoints

With dedicated data endpoints enabled, the login server issues a 307 redirect for layer blob downloads to registry-specific, region-scoped URLs.

**Using the global endpoint (Azure-managed routing):**

```
Client
  │
  ▼
contoso.azurecr.io (global endpoint: auth and data plane APIs)
  │
  ▼ Azure-managed routing
Geo-replica with best network performance profile
  │
  ▼ 307 redirect
contoso.eastus.data.azurecr.io (layer blob downloads, same region as serving geo-replica)
```

**Using a regional endpoint (client-specified routing):**

```
Client
  │
  ▼
contoso.eastus.geo.azurecr.io (regional endpoint: auth and data plane APIs)
  │
  ▼ Direct to specific geo-replica
East US geo-replica
  │
  ▼ 307 redirect
contoso.eastus.data.azurecr.io (layer blob downloads, always same region)
```

Each geo-replica gets its own dedicated data endpoint using the pattern `[registry].[region].data.azurecr.io`. Your firewall rules can be scoped to these specific endpoints instead of a broad wildcard.

## Why dedicated data endpoints matter

### Data exfiltration risk without dedicated data endpoints

Without dedicated data endpoints, firewall rules must allow `*.blob.core.windows.net` to permit layer downloads. This broad wildcard opens access to all Azure storage accounts — not just your registry's storage. A bad actor could deploy code that writes data to any Azure storage account reachable from the network.

:::image type="content" source="./media/dedicated-data-endpoints/client-firewall-2.png" alt-text="Diagram to illustrate client data exfiltration risks.":::

### Scoped firewall rules with dedicated data endpoints

Dedicated data endpoints let you replace the broad `*.blob.core.windows.net` wildcard with scoped firewall rules for your specific registry's data endpoints. For example, you can allow only `contoso.eastus.data.azurecr.io` and `contoso.westeurope.data.azurecr.io` in your firewall configuration — limiting access to your registry's storage and blocking exfiltration to other storage accounts.

:::image type="content" source="./media/dedicated-data-endpoints/contoso-example-0.png" alt-text="Diagram to illustrate contoso example with dedicated data endpoints.":::

This applies to clients connecting from on-premises hosts, IoT devices, custom build agents, or any environment where private endpoints aren't an option.

## Dedicated data endpoints with geo-replication

For [geo-replicated](container-registry-geo-replication.md) registries, each geo-replica gets its own dedicated data endpoint. The URL pattern is `[registry].[region].data.azurecr.io`.

When you pull an image from the global endpoint, ACR routes the request to the geo-replica with the best network performance profile for the client. The serving geo-replica then issues a 307 redirect for layer blob downloads to the serving geo-replica's own dedicated data endpoint. The redirect always stays within the **same region** as the serving geo-replica — a pull routed to the East US geo-replica always redirects to `contoso.eastus.data.azurecr.io`, never to a dedicated data endpoint in a different region.

This also applies when using [regional endpoints](container-registry-geo-replication.md#regional-endpoints-of-a-geo-replicated-registry-preview). A pull from a regional endpoint (`contoso.eastus.geo.azurecr.io`) redirects layer downloads to `contoso.eastus.data.azurecr.io` — never cross-region. This means your firewall rules can be scoped per region.

## Enable dedicated data endpoints

> [!NOTE]
> If you previously configured client firewall access to the existing `*.blob.core.windows.net` endpoints, switching to dedicated data endpoints impacts client connectivity, causing pull failures. To ensure clients have consistent access, add the new data endpoint rules to the client firewall rules. Once completed, enable dedicated data endpoints for your registries using the Azure CLI or other tools.
>
> During image pulls, if dedicated data endpoints are enabled, ACR gives the client a temporary download link each time it needs to fetch an image layer. This link points to the dedicated data endpoint and is valid for 20 minutes, providing a secure, short-lived URL for downloading the layer. After 20 minutes, the link expires, and the client simply requests a new one if it needs to download another layer when pulling images.

You can enable dedicated data endpoints using the Azure portal or the Azure CLI. The data endpoints follow a regional pattern, `<registry-name>.<region>.data.azurecr.io`. In a geo-replicated registry, enabling data endpoints enables endpoints in all replica regions.

### Enable dedicated data endpoints using the Azure portal

1. Go to your container registry.
1. In the service menu, under **Settings**, select **Networking**.
1. In **Public access**, select the **Use dedicated data endpoint** checkbox.
1. Select **Save**.

You now see the data endpoints in the Azure portal.

### Enable dedicated data endpoints using the Azure CLI

```azurecli
az acr update --name myregistry --data-endpoint-enabled
```

To view the data endpoints, use the [az acr show-endpoints](/cli/azure/acr#az-acr-show-endpoints) command:

```azurecli
az acr show-endpoints --name myregistry
```

This example output shows the full endpoint schema, including dedicated data endpoints and regional endpoints (if enabled):

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

After you set up dedicated data endpoints for your registry, you can enable client firewall access rules for the data endpoints. Enable data endpoint access rules for all required registry regions. For more information, see [Configure rules to access an Azure container registry behind a firewall](container-registry-firewall-access-rules.md).

## Restricting network access to data endpoints

After enabling dedicated data endpoints, you can restrict how clients reach those endpoints. There are two approaches:

### Azure private endpoints (virtual network)

[Azure private endpoints](container-registry-private-link.md) are the most secure way to control network access from clients in a virtual network to the registry. When private endpoints are configured, both the login server (global endpoint and regional endpoints) and data endpoints are accessible from within the virtual network using private IPs assigned to a private endpoint resource in the virtual network. Customers can also [disable public network access](container-registry-private-link.md#disable-public-access-to-a-registry) on the registry, ensuring that dedicated data endpoints are only reachable through the private endpoint.

If a registry has at least one private endpoint configured, dedicated data endpoints are automatically enabled. For more information, see [Connect privately to a registry from a virtual network using private endpoints](container-registry-private-link.md).

#### Private endpoint IP address considerations

Each private endpoint resource consumes private IP addresses from the virtual network subnet. Plan your subnet sizing accordingly:

- **1 IP** for the global endpoint (`myregistry.azurecr.io`)
- **1 IP per geo-replica** for dedicated data endpoints (`myregistry.<region>.data.azurecr.io`) — automatically enabled when a private endpoint is configured
- **1 IP per geo-replica** for regional endpoints (`myregistry.<region>.geo.azurecr.io`) — only if regional endpoints are enabled on the registry

For example, a registry with 3 geo-replicas and regional endpoints enabled requires 1 (global) + 3 (data) + 3 (regional) = **7 private IP addresses** per private endpoint resource. For more information, see [Connect privately to a registry from a virtual network using private endpoints](container-registry-private-link.md).

### Client firewall rules

If private endpoints aren't an option, configure your client firewall rules to allow access to the dedicated data endpoints for each required registry region. For more information, see [Configure rules to access an Azure container registry behind a firewall](container-registry-firewall-access-rules.md).

## Next steps

* Learn about [geo-replication](container-registry-geo-replication.md) and how dedicated data endpoints work with regional endpoints.
* See the [Azure Container Registry endpoint reference](container-registry-endpoint-reference.md) for a complete list of endpoint types, URL formats, and CLI flags.
