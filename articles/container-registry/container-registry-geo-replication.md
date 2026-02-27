---
title: Geo-replication in Azure Container Registry
description: Replicate your container registry across multiple Azure regions for simplified management and enhanced regional availability.
author: johnsonshi
ms.topic: how-to
ms.author: johsh
ms.service: azure-container-registry
ms.date: 02/26/2026
# Customer intent: "As a developer, I want to replicate container registry content across multiple Azure regions for high availability."
---

# Geo-replication in Azure Container Registry

Enabling geo-replication for an Azure Container Registry (ACR) creates geo-replica resources in Azure regions of your choosing. When you push images to a geo-replicated registry, content automatically syncs to all geo-replicas.

With geo-replication:

- **Manage one registry**: Maintain a single set of credentials, role assignments, networking rules, and registry configuration across all geo-replicas.
- **Use one global endpoint**: Reference `myregistry.azurecr.io/myimage:tag` in all your builds and deployments. Azure routes requests to the geo-replica with the best network performance profile for the client, which is usually the closest geo-replica. However, if the client is equidistant to multiple geo-replicas or the closest geo-replica is unavailable, requests may be routed elsewhere.
- **Automatic syncing**: Push tags and digests once; ACR replicates content and metadata to all geo-replicas.

Geo-replication requires the [Premium SKU](container-registry-skus.md).

> [!NOTE]
> - To copy images between separate registries, see [Import container images](container-registry-import-images.md).
> - To move a registry's home region to a different region, see [Relocate Azure Container Registry](/azure/azure-resource-manager/management/relocation/relocation-container-registry).

## Core considerations

### Replication model

ACR geo-replication uses an **active-active** model.

- All geo-replicas are active and writable—you can push and pull images from any geo-replica, not just the home region's geo-replica.
- This differs from primary-secondary replication models where only one region accepts writes and secondary regions are passive.

### Consistency model

ACR uses **eventual consistency**.

- After you push an image, ACR eventually replicates pushed content and metadata to all geo-replicas in the background.
- Replication time is a function of image size.
- Until replication eventually completes in the background, a geo-replica may not have the latest pushed content or metadata. You can use [webhooks](container-registry-webhook.md) to receive notifications when replication for a specific pushed image completes in each geo-replica.

## High availability considerations

Geo-replication improves availability by keeping images in multiple regions. If one region has an outage, images remain accessible from other geo-replicas—pushing and pulling continue working through the remaining geo-replicas.

For maximum resilience:

- If the home region (where you created the registry) is unavailable, you can still push and pull images, but you can't modify registry properties until the home region recovers.
- If your registry uses a [customer-managed key](tutorial-enable-customer-managed-keys.md), review the [key vault failover and redundancy guidance](/azure/key-vault/general/disaster-recovery-guidance).

### SLA and service tier considerations

Azure Container Registry SLAs apply to each geo-replica independently.

Certain service tier limits have the following special consideration:

- **Storage**: Storage limits are shared across all geo-replicas. When you push a 1 GiB image, it occupies 1 GiB in all geo-replicas after eventual replication completes.
- **API rate limits**: Throttling limits on API operations, such as the number of reads and writes per minute, are geo-replica-specific.

For more information on service tiers and limits, see [ACR service tiers](container-registry-skus.md).

### Pricing considerations

Geo-replication can reduce costs by enabling in-region image pushes and pulls, which avoids cross-region data transfer charges during these push or pull operations.

However, cross-region charges still apply for replicating data to geo-replicas when you push images. Each geo-replica also incurs its own storage costs.

For more information, see [ACR pricing](https://azure.microsoft.com/pricing/details/container-registry/).

## Configure geo-replication

### Permissions needed

To manage geo-replicas, your identity needs these permissions:

| Permission | Description |
| ---------- | ----------- | 
| `Microsoft.ContainerRegistry/registries/read` | Get registry properties |
| `Microsoft.ContainerRegistry/registries/write` | Create or update registry properties |
| `Microsoft.ContainerRegistry/registries/replications/read` | List geo-replicas |
| `Microsoft.ContainerRegistry/registries/replications/write` | Create or update a geo-replica |
| `Microsoft.ContainerRegistry/registries/replications/delete` | Delete a geo-replica |
| `Microsoft.ContainerRegistry/registries/replications/operationStatuses/read` | Get geo-replica operation status | 

### Azure portal

1. Go to your registry in the [Azure portal](https://portal.azure.com).
2. Under **Services**, select **Geo-replications**.
3. On the map:
   - **Blue hexagon**: Home region (where you created the registry)
   - **Green hexagons**: Available regions
   - **Gray hexagons**: Unavailable regions
4. Select a green hexagon, then select **Create**.

:::image type="content" source="media/container-registry-geo-replication/registry-geo-map.png" alt-text="Screenshot of the geo-replication map in the Azure portal." lightbox="media/container-registry-geo-replication/registry-geo-map.png":::

### Azure CLI

```azurecli
# Create a replica
az acr replication create --registry myregistry --location eastus

# List replicas
az acr replication list --registry myregistry --output table

# Delete a replica
az acr replication delete --registry myregistry --name eastus
```

For more commands, see [az acr replication](/cli/azure/acr/replication).

## Push or pull images through the global endpoint

After configuring geo-replication, you can push or pull content to your registry through the registry's global endpoint (`myregistry.azurecr.io`).

- When you push or pull through the global endpoint, ACR routes the request to the geo-replica with the best network performance profile for the client.
- This is usually the closest geo-replica.
- However, if the client is equidistant to multiple geo-replicas or the closest geo-replica is unavailable, requests may be routed elsewhere.
- This routing is managed by ACR—you don't control which geo-replica handles a specific request.

### Temporarily exclude a geo-replica from global endpoint routing

You can exclude a geo-replica from global endpoint routing by disabling the `--region-endpoint-enabled` setting for a specific geo-replica. This is useful for maintenance or troubleshooting.

- When the `--region-endpoint-enabled` setting for a specific geo-replica is set to `false`, ACR stops routing requests to that specific geo-replica for requests that go to the global endpoint.
- Data continues syncing with a geo-replica even if global endpoint routing is disabled for that specific geo-replica.
- As such, storage quota and costs continue accruing for that geo-replica.

```azurecli
# Prevent the global endpoint from routing to a specific geo-replica.
# This excludes only the specific geo-replica from global endpoint routing.
az acr replication update --registry myregistry --name eastus \
  --region-endpoint-enabled false

# Re-enable the geo-replica in global endpoint routing.
# This allows the global endpoint to route certain requests to the geo-replica
# depending on the client's network performance profile with the geo-replica.
az acr replication update --registry myregistry --name eastus \
  --region-endpoint-enabled true
```

## Push or pull images through geo-replica regional endpoints

Regional endpoints give you dedicated URLs for each replica, letting you specify exactly which regional geo-replica handles your push or pull request:

- myregistry.**_eastus.geo_**.azurecr.io
- myregistry.**_westeurope.geo_**.azurecr.io

Use regional endpoints when you need:

- **Predictable routing**: Ensure a workload always uses a specific replica for in-region affinity.
- **Client-side failover**: Implement your own failover logic between regional geo-replicas in case of outages.
- **Push-pull consistency**: Push and pull from the same replica to avoid replication lag.

> [!NOTE]
> Regional endpoints are currently in private preview.
> For enrollment and documentation, see [Regional endpoints for geo-replicated registries](https://github.com/Azure/acr/tree/main/docs/preview/regional-endpoints).

## Troubleshooting

### Push fails with manifest errors

Some Linux DNS resolvers don't cache responses consistently. If there are several geo-replicas in nearby regions, DNS may resolve to different replicas during a single push (DNS bouncing), causing the pushed manifest to reference layers that were pushed to a different geo-replica.

**Solutions**:

- Use [regional endpoints](#push-or-pull-images-through-geo-replica-regional-endpoints) to push to a specific geo-replica.
- Use a DNS cache like `dnsmasq` on the client.
- For Linux VMs in Azure, see [DNS name resolution options](/azure/virtual-machines/linux/azure-dns).

### Geo-replica creation stuck for private endpoint-enabled registries

This usually arises when the identity creating a geo-replica for a private endpoint-enabled registry does not have sufficient permissions to create private endpoint networking resources.

**Solution**:

- To resolve, manually delete the geo-replica that got stuck in the provisioning state.
- Afterwards, ensure the identity has the permission `Microsoft.Network/privateEndpoints/privateLinkServiceProxies/write` before creating a geo-replica.

## Next steps

- [Regional endpoints](https://github.com/Azure/acr/tree/main/docs/preview/regional-endpoints)

## Related content

- [ACR SKUs and service tiers](container-registry-skus.md)
- [Webhooks](container-registry-webhook.md)
- [Private endpoints](container-registry-private-link.md)
