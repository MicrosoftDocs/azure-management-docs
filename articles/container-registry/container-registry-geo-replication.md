---
title: Geo-replicate Azure Container Registry to Multiple Regions
description: Learn how to create and manage a geo-replicated Azure container registry to serve multiple regions efficiently with the Premium service tier.
author: rayoef
ms.topic: how-to
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 10/31/2023
# Customer intent: "As a DevOps engineer, I want to configure geo-replication for my container registry, so that I can efficiently serve multiple Azure regions with reduced latency and improved performance while managing a single registry across all regions."
---
# Geo-replication in Azure Container Registry

Companies that want a local presence or a hot backup choose to run services from multiple Azure regions. As a best practice, placing a container registry in each region where images are run allows network-close operations, enabling fast, reliable image layer transfers. Geo-replication enables an Azure container registry to function as a single registry, serving multiple regions with multi-primary regional registries. 

A geo-replicated registry provides the following benefits:

* Single registry, image, and tag names can be used across multiple regions
* Improve performance and reliability of regional deployments with network-close registry access
* Reduce data transfer costs by pulling image layers from a local, replicated registry in the same or nearby region as your container host
* Single management of a registry across multiple regions
* Registry resilience if a regional outage occurs

> [!NOTE]
> * If you need to maintain copies of container images in more than one Azure container registry, Azure Container Registry also supports [image import](container-registry-import-images.md). For example, in a DevOps workflow, you can import an image from a development registry to a production registry without needing to use Docker commands.
> * If you want to move a registry to a different Azure region, instead of geo-replicating the registry, see [Manually move a container registry to another region](manual-regional-move.md).

## Prerequisites

* The user requires the following permissions (at the registry level) to create/delete replications:

    | Permission | Description |
    |---|---|
    | Microsoft.ContainerRegistry/registries/write | Create a replication |
    | Microsoft.ContainerRegistry/registries/replications/write | Delete a replication |

## Example use case
Contoso runs a public presence website located across the US, Canada, and Europe. To serve these markets with local and network-close content, Contoso runs [Azure Kubernetes Service (AKS)](/azure/aks/) clusters in West US, East US, Canada Central, and West Europe. The website application, deployed as a Docker image, utilizes the same code and image across all regions. Content local to that region is retrieved from a database, which is provisioned uniquely in each region. Each regional deployment has its unique configuration for resources like the local database.

The development team is located in Seattle, WA, and utilizes the West US data center.

![Pushing to multiple registries](media/container-registry-geo-replication/before-geo-replicate.png)<br />*Pushing to multiple registries*

Prior to using the geo-replication features, Contoso had a US-based registry in West US, with an additional registry in West Europe. To serve these different regions, the development team pushed images to two different registries.

```bash
docker push contoso.azurecr.io/public/products/web:1.2
docker push contosowesteu.azurecr.io/public/products/web:1.2
```
![Pulling from multiple registries](media/container-registry-geo-replication/before-geo-replicate-pull.png)<br />*Pulling from multiple registries*

Typical challenges of multiple registries include:

* All the East US, West US, and Canada Central clusters pull from the West US registry, incurring egress fees as each of these remote container hosts pull images from West US data centers.
* The development team must push images to West US and West Europe registries.
* The development team must configure and maintain each regional deployment with image names referencing the local registry.
* Registry access must be configured for each region.

## Benefits of geo-replication

![Pulling from a geo-replicated registry](media/container-registry-geo-replication/after-geo-replicate-pull.png)

The geo-replication feature of Azure Container Registry has the following benefits:

* Manage a single registry across all regions: `contoso.azurecr.io`
* Manage a single configuration of image deployments as all regions use the same image URL: `contoso.azurecr.io/public/products/web:1.2`
* Push to a single registry while ACR automatically manages the geo-replication. ACR only replicates unique layers, reducing data transfer across regions. 
* Configure regional [webhooks](container-registry-webhook.md) to notify you of events in specific replicas.
* Provide a highly available registry that is resilient to regional outages.

Azure Container Registry also supports [availability zones](zone-redundancy.md) to create a resilient and high availability Azure container registry within an Azure region. The combination of availability zones for redundancy within a region, and geo-replication across multiple regions, enhances both the reliability and performance of a registry.

## Configure geo-replication

Configuring geo-replication is as easy as clicking regions on a map. You can also manage geo-replication using tools including the [az acr replication](/cli/azure/acr/replication) commands in the Azure CLI, or deploy a registry enabled for geo-replication with an [Azure Resource Manager template](https://azure.microsoft.com/resources/templates/container-registry-geo-replication/).

Geo-replication is a feature of [Premium registries](container-registry-skus.md). If your registry isn't yet Premium, you can change from Basic and Standard to Premium in the [Azure portal](https://portal.azure.com):

![Switching service tiers in the Azure portal](media/container-registry-skus/update-registry-sku.png)

To configure geo-replication for your Premium registry, sign in to the [Azure portal](https://portal.azure.com).

Navigate to your Azure Container Registry, and select **Replications**:

![Replications in the Azure portal container registry UI](media/container-registry-geo-replication/registry-services.png)

A map is displayed showing all current Azure Regions:

 ![Region map in the Azure portal](media/container-registry-geo-replication/registry-geo-map.png)

* Blue hexagons represent current replicas
* Green hexagons represent possible replica regions
* Gray hexagons represent Azure regions not yet available for replication

To configure a replica, select a green hexagon, then select **Create**:

 ![Create replication UI in the Azure portal](media/container-registry-geo-replication/create-replication.png)

To configure additional replicas, select the green hexagons for other regions, then click **Create**.

ACR begins syncing images across the configured replicas. Once complete, the portal reflects *Ready*. The replica status in the portal doesn't automatically update. Use the refresh button to see the updated status.

## Considerations for using a geo-replicated registry

* Each region in a geo-replicated registry is independent once set-up. Azure Container Registry SLAs apply to each geo-replicated region.
* For every push or pull image operation on a geo-replicated registry, Azure Traffic Manager in the background sends a request to the registry's closest location in the region to maintain network latency.
* After you push an image or tag update to the closest region, it takes some time for Azure Container Registry to replicate the manifests and layers to the remaining regions you opted into. Larger images take longer to replicate than smaller ones. Images and tags are synchronized across the replication regions with an eventual consistency model.
* To manage workflows that depend on push updates to a geo-replicated registry, we recommend that you configure [webhooks](container-registry-webhook.md) to respond to the push events. You can set up regional webhooks within a geo-replicated registry to track push events as they complete across the geo-replicated regions.
* To serve blobs representing content layers, Azure Container Registry uses data endpoints. You can enable [dedicated data endpoints](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints) for your registry in each of your registry's geo-replicated regions. These endpoints allow configuration of tightly scoped firewall access rules. For troubleshooting purposes, you can optionally [disable routing to a replication](#temporarily-disable-routing-to-replication) while maintaining replicated data.
* If you configure a [private link](container-registry-private-link.md) for your registry using private endpoints in a virtual network, dedicated data endpoints in each of the geo-replicated regions are enabled by default. 

## Considerations for high availability

* For high availability and resiliency, we recommend creating a registry in a region that supports enabling [zone redundancy](zone-redundancy.md). Enabling zone redundancy in each replica region is also recommended.
* If an outage occurs in the registry's home region (the region where it was created) or one of its replica regions, a geo-replicated registry remains available for data plane operations such as pushing or pulling container images. 
* If the registry's home region becomes unavailable, you may be unable to carry out registry management operations, including configuring network rules, enabling availability zones, and managing replicas.
* To plan for high availability of a geo-replicated registry encrypted with a [customer-managed key](tutorial-enable-customer-managed-keys.md) stored in an Azure key vault, review the guidance for key vault [failover and redundancy](/azure/key-vault/general/disaster-recovery-guidance).

## Delete a replica

After you've configured a replica for your registry, you can delete it at any time if it's no longer needed. Delete a replica using the Azure portal or other tools such as the [az acr replication delete](/cli/azure/acr/replication#az-acr-replication-delete) command in the Azure CLI.

To delete a replica in the Azure portal:

1. Navigate to your Azure Container Registry and select **Replications**.
1. Select the name of a replica and select **Delete**. Confirm that you want to delete the replica.

To use the Azure CLI to delete a replica of *myregistry* in the East US region:

```azurecli
az acr replication delete --name eastus --registry myregistry
```

## Geo-replication pricing

Geo-replication is a feature of the [Premium service tier](container-registry-skus.md) of Azure Container Registry. When you replicate a registry to your desired regions, you incur Premium registry fees for each region.

In the preceding example, Contoso consolidated two registries down to one, adding replicas to East US, Canada Central, and West Europe. Contoso would pay four times Premium per month, with no additional configuration or management. Each region now pulls their images locally, improving performance and reliability without network egress fees from the West US to Canada and the East US.

## Troubleshoot push operations with geo-replicated registries
 
A Docker client that pushes an image to a geo-replicated registry may not push all image layers and its manifest to a single replicated region. This may occur because Azure Traffic Manager routes registry requests to the network-closest replicated registry. If the registry has two *nearby* replication regions, image layers and the manifest could be distributed to the two sites, and the push operation fails when the manifest is validated. This problem occurs because of the way the DNS name of the registry is resolved on some Linux hosts. This issue doesn't occur on Windows, which provides a client-side DNS cache.
 
If this problem occurs, one solution is to apply a client-side DNS cache such as `dnsmasq` on the Linux host. This helps ensure that the registry's name is resolved consistently. If you're using a Linux VM in Azure to push to a registry, see options in [DNS Name Resolution options for Linux virtual machines in Azure](/azure/virtual-machines/linux/azure-dns).

To optimize DNS resolution to the closest replica when pushing images, configure a geo-replicated registry in the same Azure regions as the source of the push operations, or the closest region when working outside of Azure.

### Temporarily disable routing to replication

To troubleshoot operations with a geo-replicated registry, you might want to temporarily disable Traffic Manager routing to one or more replications. Starting in Azure CLI version 2.8, you can configure a `--region-endpoint-enabled` option (preview) when you create or update a replicated region. When you set a replication's `--region-endpoint-enabled` option to `false`, Traffic Manager no longer routes docker push or pull requests to that region. By default, routing to all replications is enabled, and data synchronization across all replications takes place whether routing is enabled or disabled.

To disable routing to an existing replication, first run [az acr replication list][az-acr-replication-list] to list the replications in the registry. Then, run [az acr replication update][az-acr-replication-update] and set `--region-endpoint-enabled false` for a specific replication. For example, to configure the setting for the *westus* replication in *myregistry*:

```azurecli
# Show names of existing replications
az acr replication list --registry --output table

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

## Creating replication for a Private Endpoint enabled registry

When creating a new registry replication for the primary registry enabled with Private Endpoint, we recommend validating that the User Identity has valid Private Endpoint creation permissions. Otherwise, the operation gets stuck in the provisioning state while creating the replication.

Follow the below steps if you got stuck in the provisioning state while creating the registry replication:

- Manually delete the replication that got stuck in the provisioning state.
- Add the `Microsoft.Network/privateEndpoints/privateLinkServiceProxies/write` permission for the User Identity.
- Recreate the registry replication request.

This permission check is only applicable to the registries with Private Endpoint enabled.

## Next steps

Check out the three-part tutorial series, [Geo-replication in Azure Container Registry](container-registry-tutorial-prepare-registry.md). Walk through creating a geo-replicated registry, building a container, and then deploying it with a single `docker push` command to multiple regional Web Apps for Containers instances.

> [!div class="nextstepaction"]
> [Geo-replication in Azure Container Registry](container-registry-tutorial-prepare-registry.md)

[az-acr-replication-list]: /cli/azure/acr/replication#az_acr_replication_list
[az-acr-replication-update]: /cli/azure/acr/replication#az_acr_replication_update
