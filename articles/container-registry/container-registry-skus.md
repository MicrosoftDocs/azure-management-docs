---
title: Azure Container Registry SKU Features and Limits
description: Learn about the features and limits in various SKUs of Azure Container Registry.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.date: 10/31/2023
ms.service: azure-container-registry
# Customer intent: "As a developer using Azure Container Registry, I want to understand the differences between SKU features and limits, so that I can choose the most appropriate SKU."
---

# Azure Container Registry SKU Features and Limits

Azure Container Registry (ACR) is available in multiple SKUs. These SKUs provide predictable pricing and several options for aligning to the capacity and usage patterns of your private container registry in Azure.

| SKU | Description |
| --- | ----------- |
| **Basic** | A cost-optimized entry point for developers learning about Azure Container Registry. Basic registries have the same programmatic capabilities as Standard and Premium (such as Microsoft Entra [authentication integration](container-registry-authentication.md#individual-login-with-azure-ad), [image deletion][container-registry-delete], and [webhooks][container-registry-webhook]). However, the included storage and image throughput are most appropriate for lower usage scenarios. |
| **Standard** | Standard registries offer the same capabilities as Basic, with increased included storage and image throughput. Standard registries should satisfy the needs of most production scenarios. |
| **Premium** | Premium registries provide the highest amount of included storage and concurrent operations, enabling high-volume scenarios. In addition to higher image throughput, Premium adds features such as high availability and resiliency through [geo-replication][container-registry-geo-replication] for managing a single registry across multiple regions, [private link with private endpoints](container-registry-private-link.md) to restrict access to the registry, as well as higher API concurrency and bandwidth throughput for large-scale concurrent deployments. |

The Basic, Standard, and Premium SKUs all provide the same programmatic capabilities and data plane APIs. They also all benefit from [image storage][container-registry-storage] managed entirely by Azure. Azure Container Registry recommends the Premium SKU for most scenarios to take advantage of the additional features and higher limits.

> [!NOTE]
> Some limits listed in this table can be increased by contacting [Azure Support](https://azure.microsoft.com/support/create-ticket/). See the [Request a limit increase](#request-a-limit-increase) section for details on the various limits that can be increased.

## SKU features and limits

The following table details the features and registry limits of the Basic, Standard, and Premium SKUs.

[!INCLUDE [container-instances-limits](~/reusable-content/ce-skilling/azure/includes/container-registry/container-registry-limits.md)]

## Private endpoint limits

Private endpoints are available exclusively in the Premium SKU and enable secure, private network access to your container registry. With private endpoints, you can restrict registry access to specific Azure virtual networks using Azure Private Link.

The Premium SKU supports a maximum number of private endpoints per registry. Consult the Premium SKU's limit in the [SKU features and limits table](#sku-features-and-limits).

## Registry image pull and push performance limits

Image pull and push performance is primarily affected by API concurrency, bandwidth throughput, and throttling during high-volume operations. These factors depend on your registry SKU, network configuration, and client configuration.

### API concurrency and bandwidth throughput limits

API concurrency and bandwidth throughput depend on your SKU. Higher-SKU registries support more concurrent operations and greater bandwidth for data-plane operations like listing, deleting, pushing, and pulling images.

API concurrency and bandwidth throughput during image pulls and pushes is affected by: 

* Number and size of image layers
* Reuse of layers across images in the registry
* Additional API calls required for each operation
* Scale of concurrent deployments (for example, Kubernetes deployments pulling images across multiple nodes simultaneously)

Client environment factors:

* Docker daemon or Podman configuration for concurrent operations
* Container runtime configuration, such as containerd or CRI-O concurrency settings
* Cluster configuration or cluster data plane settings

Network factors:

* Network bandwidth and latency for the network hops from clients to the registry
* Client-side network configuration (for example, firewall rules, proxy settings)
* Geographic distance to the registry (or nearest replica if [geo-replicated](container-registry-geo-replication.md))

For API operation details that happen during image push and pull, see the [Docker HTTP API V2](https://docs.docker.com/registry/spec/api/) documentation.

For troubleshooting, see [Troubleshoot registry performance](container-registry-troubleshoot-performance.md). 

#### Example

Pushing a single 133 MB `nginx:latest` image to an Azure container registry requires multiple read and write API operations for the image's five layers, all of which consume bandwidth throughput and API concurrency:

* Read operations to read the image manifest, if it exists in the registry
* Write operations to write the configuration blob of the image
* Write operations to write the image manifest

### Throttling and bandwidth constraints

During periods of high request volume, you may experience throttling with an HTTP 429 `Too many requests` error or slow bandwidth throughput. To mitigate these issues:

* Implement retry logic with exponential backoff
* Reduce the rate of concurrent requests
* Space out large-scale deployments to reduce simultaneous image pulls across multiple nodes

> [!NOTE]
> If you experience persistent API throttling or slow bandwidth throughput, see [Request a limit increase](#request-a-limit-increase) for information on contacting Azure Support to discuss increasing your registry's image pull/push performance limits.

## Registry storage limits

Registry storage is managed entirely by Azure and varies by SKU. Each SKU includes a specific amount of storage, with additional storage available at a per-GB rate.

### Example

- A Basic SKU registry includes 10 GB of storage at $0.167 per day (prices in US dollars)
- If a Basic SKU registry uses 25 GB storage, the cost is $0.167 + ($0.003/day × 15 GB) = $0.212 per day
- Additional charges may apply for networking, builds, and other operations according to [Container Registry pricing](https://azure.microsoft.com/pricing/details/container-registry/)

## Show registry usage

Use the [az acr show-usage](/cli/azure/acr#az-acr-show-usage) command in the Azure CLI, [Get-AzContainerRegistryUsage](/powershell/module/az.containerregistry/get-azcontainerregistryusage) in Azure PowerShell, or the [Registries - List Usages](/rest/api/containerregistry/) REST API, to get a snapshot of your registry's current consumption of storage and other resources, compared with the limits for that registry's SKU. Storage usage also appears on the registry's **Overview** page in the portal.

Usage information helps you make decisions about [changing the SKU](#changing-skus) when your registry nears a limit. This information also helps you [manage consumption](container-registry-best-practices.md#manage-registry-size).

> [!NOTE]
> The registry's storage usage should only be used as a guide and may not reflect recent registry operations. Monitor the registry's [StorageUsed metric](monitor-service-reference.md#container-registry-metrics) for up-to-date data.

Depending on your registry's SKU, usage information includes some or all of the following, along with the limit in that SKU:

* Storage consumed in bytes<sup>1</sup>
* Number of [webhooks](container-registry-webhook.md)
* Number of [geo-replications](container-registry-geo-replication.md) (includes the home replica)
* Number of [private endpoints](container-registry-private-link.md)
* Number of [IP access rules](container-registry-access-selected-networks.md)
* Number of [virtual network rules](container-registry-vnet.md)

<sup>1</sup>In a geo-replicated registry, storage usage is shown for the home region. Multiply by the number of replications for total storage consumed.

## Changing SKUs

You can change a registry's SKU with the Azure CLI or in the Azure portal. You can move freely between SKUs as long as the SKU you're switching to has the required maximum storage capacity.

There is no registry downtime or impact on registry operations when you move between SKUs.

### Azure CLI

To move between SKUs in the Azure CLI, use the [az acr update][az-acr-update] command. For example, to switch to Premium:

```azurecli
az acr update --name myContainerRegistry --sku Premium
```

### Azure PowerShell

To move between service SKUs in Azure PowerShell, use the [Update-AzContainerRegistry][update-azcontainerregistry] cmdlet. For example, to switch to Premium:

```azurepowershell
Update-AzContainerRegistry -ResourceGroupName myResourceGroup -Name myContainerRegistry -Sku Premium
```

### Azure portal

In the container registry **Overview** in the Azure portal, select **Update**, then select a new **SKU** from the SKU drop-down.

![Update container registry SKU in Azure portal][update-registry-sku]

## Pricing

For pricing information on each of the Azure Container Registry SKUs, see [Container Registry pricing][container-registry-pricing].

For details about pricing for data transfers, see [Bandwidth Pricing Details](https://azure.microsoft.com/pricing/details/bandwidth/).

## Request a limit increase

If you need to increase limits for your registry, contact [Azure Support](https://azure.microsoft.com/support/create-ticket/) to discuss:

- Increasing private endpoint limits
- Increasing image pull/push performance if you experience throttling, low API concurrency, or slow bandwidth
- Increasing storage limits

## Next steps

**Azure Container Registry Roadmap**

Visit the [ACR Roadmap][acr-roadmap] on GitHub to find information about upcoming features in the service.

<!-- IMAGES -->
[update-registry-sku]: ./media/container-registry-skus/update-registry-sku.png

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[container-registry-uservoice]: https://feedback.azure.com/d365community/forum/180a533d-0d25-ec11-b6e6-000d3a4f0858

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az_acr_update
[update-azcontainerregistry]: /powershell/module/az.containerregistry/update-azcontainerregistry
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-storage]: container-registry-storage.md
[container-registry-delete]: container-registry-delete.md
[container-registry-webhook]: container-registry-webhook.md
