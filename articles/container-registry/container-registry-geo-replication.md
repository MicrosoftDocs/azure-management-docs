---
title: Geo-replicate Azure Container Registry to Multiple Regions
description: Learn how to create and manage a geo-replicated Azure container registry to serve multiple regions efficiently with the Premium service tier.
author: rayoef
ms.topic: how-to
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 02/06/2026
# Customer intent: "As a DevOps engineer, I want to configure geo-replication for my container registry, so that I can efficiently serve multiple Azure regions with reduced latency and improved performance while managing a single registry across all regions."
---
# Geo-replication in Azure Container Registry

Companies that want a local presence or a hot backup choose to run services from multiple Azure regions. Placing a container registry in each region where you run images allows network-close operations and enables fast, reliable image layer transfers, but it requires each registry to be managed separately. By using geo-replication, one Azure container registry can serve multiple regions. You manage a single registry and image name across all regions. Push to a single registry, and Azure Container Registry automatically replicates the content to the other regions you configure.

A geo-replicated registry provides the following benefits:

* Use single registry, image, and tag names across multiple regions.
* Get better performance and reliability of regional deployments with network-close registry access.
* Pay lower data transfer costs by pulling image layers from a local, replicated registry in the same or nearby region as your container host.
* Manage a registry across multiple regions.
* Enhance registry resilience if a regional outage occurs.

Geo-replication is a feature of the [Premium service tier](container-registry-skus.md) of Azure Container Registry. When you replicate a registry to your desired regions, you incur Premium registry fees for each region. However, you can save on data transfer costs and improve performance by serving images from a local replica in each region.

> [!NOTE]
> * If you need to maintain copies of container images in more than one Azure container registry, Azure Container Registry also supports [image import](container-registry-import-images.md). For example, in a DevOps workflow, you can import an image from a development registry to a production registry without needing to use Docker commands.
> * To move a registry to a different Azure region, instead of geo-replicating the registry, see [Manually move a container registry to another region](manual-regional-move.md).

Azure Container Registry also supports [availability zones](zone-redundancy.md) to create a resilient and highly available Azure container registry within an Azure region. The combination of availability zones for redundancy within a region, and geo-replication across multiple regions, enhances both the reliability and performance of a registry.

## Example use case

Contoso runs a public presence website located across the US, Canada, and Europe. To serve these markets with local and network-close content, Contoso runs [Azure Kubernetes Service (AKS)](/azure/aks/) clusters in West US, East US, Canada Central, and West Europe. The website application, deployed as a Docker image, uses the same code and image across all regions. Content local to a region is retrieved from a database that's provisioned uniquely in each region. Each regional deployment has its unique configuration for resources like the local database.

Before using geo-replication, Contoso had a US-based registry in West US, with an additional registry in West Europe. To serve these different regions, the development team pushed images to the registries in these two regions.

```bash
docker push contoso.azurecr.io/public/products/web:1.2
docker push contosowesteu.azurecr.io/public/products/web:1.2
```

:::image type="content" source="media/container-registry-geo-replication/before-geo-replicate.png" alt-text="Diagram of the example scenario's regional distribution before using geo-replication.":::

With this setup, all of Contoso's East US, West US, and Canada Central clusters pull from the West US registry, incurring egress fees. The development team has to configure and maintain each regional deployment with image names referencing the local registry, and they must configure registry access for each region.

:::image type="content" source="media/container-registry-geo-replication/before-geo-replicate-pull.png" alt-text="Diagram showing pulls from two registries before geo-replication.":::

By using geo-replication, Contoso can consolidate to a single registry with replicas in the desired regions. With a single registry named `contoso.azurecr.io`, the team can manage a single configuration of image deployments, as all regions use the same image URL (`contoso.azurecr.io/public/products/web:1.2`). When the development team pushes an image, Azure Container Registry automatically replicates the image to the other regions. Each regional deployment pulls from the closest replica, improving performance and reliability while eliminating egress fees.

This architecture also provides a highly available registry. If an outage occurs in one region, the registry remains available in the other regions, allowing regional deployments to continue pulling images without interruption. Contoso can also configure regional [webhooks](container-registry-webhook.md) to get notifications for events in specific replicas.

:::image type="content" source="media/container-registry-geo-replication/after-geo-replicate-pull.png" alt-text="Screenshot of the example scenario with geo-replication enabled, allowing registries to pull from the nearest replica.":::

In this example, Contoso consolidated two registries down to one, adding replicas to East US, Canada Central, and West Europe. Contoso pays for the four Premium registries, with no extra configuration or management required. Each region now pulls their images locally, improving performance and reliability without network egress fees from the West US to Canada and the East US.

## Considerations for using a geo-replicated registry

Keep in mind the following considerations and recommendations when using geo-replication:

* Each region in a geo-replicated registry is independent once set up. Azure Container Registry SLAs apply to each geo-replicated region.
* For every push or pull image operation on a geo-replicated registry, Azure Traffic Manager in the background sends a request to the registry's closest location in the region to maintain network latency.
* After you push an image or tag update to the closest region, it takes some time for Azure Container Registry to replicate the manifests and layers to the remaining regions you opted into. Larger images take longer to replicate than smaller ones. Images and tags are synchronized across the replication regions with an eventual consistency model.
* To manage workflows that depend on push updates to a geo-replicated registry, configure [webhooks](container-registry-webhook.md) to respond to the push events. You can set up regional webhooks within a geo-replicated registry to track push events as they complete across the geo-replicated regions.
* To serve blobs representing content layers, Azure Container Registry uses data endpoints. You can enable [dedicated data endpoints](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints) for your registry in each of your registry's geo-replicated regions. These endpoints allow configuration of tightly scoped firewall access rules. For troubleshooting purposes, you can optionally [disable routing to a replication](#temporarily-disable-routing-to-replication) while maintaining replicated data.
* If you configure a [private link](container-registry-private-link.md) for your registry using private endpoints in a virtual network, dedicated data endpoints in each of the geo-replicated regions are enabled by default. 

## Geo-replication considerations for high availability

For high availability and resiliency, create a registry in a region that supports [zone redundancy](zone-redundancy.md), and enable zone redundancy in each replica region.

If an outage occurs in the registry's home region (the region where you created it) or one of its replica regions, a geo-replicated registry remains available for data plane operations such as pushing or pulling container images. If the registry's home region becomes unavailable, you might be unable to carry out registry management operations, including configuring network rules, enabling availability zones, and managing replicas.

To plan for high availability of a geo-replicated registry encrypted with a [customer-managed key](tutorial-enable-customer-managed-keys.md) stored in an Azure key vault, review the guidance for key vault [failover and redundancy](/azure/key-vault/general/disaster-recovery-guidance).

## Configure geo-replication

Configuring geo-replication is as simple as selecting regions on a map in the Azure portal. You can manage geo-replication by using tools such as the [`az acr replication`](/cli/azure/acr/replication) commands in the Azure CLI. You can also deploy a registry enabled for geo-replication by using an [Azure Resource Manager template](https://azure.microsoft.com/resources/templates/container-registry-geo-replication/).

To create or delete replications, you need the following permissions for a container registry:

* Microsoft.ContainerRegistry/registries/write | Create a replication |
* Microsoft.ContainerRegistry/registries/replications/write | Delete a replication |

To configure geo-replication in the [Azure portal](https://portal.azure.com), go to your Azure Container Registry. In the service menu, under **Services**, select **Geo-replications**.

You see a map showing the different Azure regions. The blue hexagon represents the registry's home region, where you created the registry. The green hexagons represent regions where you can create replicas. The gray hexagons represent regions that aren't yet available for replication.

To configure a replica, select a green hexagon, and then select **Create**. In the **Create replication** pane, review the settings, and then select **Create** to create the replica in that region. The map updates to show a solid green hexagon for the new replica. You can repeat this process to create replicas in additional regions.

The following example shows the map for a registry created in the West US region, with replicas in East US, Canada Central, and West Europe.

:::image type="content" source="media/container-registry-geo-replication/registry-geo-map.png" alt-text="Screenshot of a map showing geo-replications for a container registry in the Azure portal." lightbox="media/container-registry-geo-replication/registry-geo-map.png":::

When you create a replica, the registry begins syncing images. The status updates to **Ready** once the sync is complete. You might have to refresh the page to see the updated status. After the initial replication, any new pushes to the registry are automatically replicated to all regions.

After you configure a replica for your registry, you can delete it at any time. To delete a replica in the Azure portal, select the replica, and then select **Delete**. To delete a replica by using the Azure CLI, use the [`az acr replication delete`](/cli/azure/acr/replication#az-acr-replication-delete) command. For example:

```azurecli
az acr replication delete --name eastus --registry myregistry
```

## Troubleshoot push operations with geo-replicated registries

A Docker client that pushes an image to a geo-replicated registry might not push all image layers and its manifest to a single replicated region. Azure Traffic Manager routes registry requests to the network-closest replicated registry. If the registry has two *nearby* replication regions, image layers and the manifest can be distributed to the two sites, and the push operation fails when the manifest is validated. This problem occurs because of the way the DNS name of the registry is resolved on some Linux hosts. 

If this problem occurs, one solution is to apply a client-side DNS cache such as `dnsmasq` on the Linux host. This solution helps ensure that the registry's name is resolved consistently. If you're using a Linux VM in Azure to push to a registry, see options in [DNS Name Resolution options for Linux virtual machines in Azure](/azure/virtual-machines/linux/azure-dns).

To optimize DNS resolution to the closest replica when pushing images, configure a geo-replicated registry in the same Azure regions as the source of the push operations, or the closest region when working outside of Azure.

### Temporarily disable routing to replication

To troubleshoot operations with a geo-replicated registry, you might want to temporarily disable Traffic Manager routing to one or more replications. Starting in Azure CLI version 2.8, you can configure a `--region-endpoint-enabled` option (preview) when you create or update a replicated region. When you set `--region-endpoint-enabled` to `false`, Traffic Manager no longer routes docker push or pull requests to that region. By default, routing to all replications is enabled, and data synchronization across all replications takes place whether routing is enabled or disabled.

To disable routing to an existing replication, first run [az acr replication list][az-acr-replication-list] to list the replications in the registry. Then, run [az acr replication update][az-acr-replication-update] and set `--region-endpoint-enabled false` for a specific replication. For example, to configure the setting for the *westus* replication in *myregistry*:

```azurecli
# Show names of existing replications
az acr replication list --registry myregistry --output table

# Disable routing to replication
az acr replication update --name westus \
  --registry myregistry --resource-group MyResourceGroup \
  --region-endpoint-enabled false
```

To restore routing to a replication:

```azurecli
az acr replication update --name westus \
  --registry myregistry --resource-group MyResourceGroup \
  --region-endpoint-enabled true
```

### Geo-replication for registries enabled with Private Endpoint

When creating a new registry replication for the primary registry enabled with Private Endpoint, validate that the user identity has valid Private Endpoint creation permissions. Otherwise, the operation might get stuck in the provisioning state while creating the replication.

If the operation gets stuck in the provisioning state while creating the registry replication:

* Manually delete the replication that got stuck in the provisioning state.
* Add the `Microsoft.Network/privateEndpoints/privateLinkServiceProxies/write` permission for the user identity.
* Recreate the registry replication request.

This permission check is only applicable to the registries with Private Endpoint enabled.

## Next steps

Check out the three-part tutorial series, [Geo-replication in Azure Container Registry](container-registry-tutorial-prepare-registry.md). Walk through creating a geo-replicated registry, building a container, and then deploying it by using a single `docker push` command to multiple regional Web Apps for Containers instances.

> [!div class="nextstepaction"]
> [Geo-replication in Azure Container Registry](container-registry-tutorial-prepare-registry.md)

[az-acr-replication-list]: /cli/azure/acr/replication#az-acr-replication-list
[az-acr-replication-update]: /cli/azure/acr/replication#az-acr-replication-update
