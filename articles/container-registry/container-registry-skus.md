---
title: Azure Container Registry SKU Features and Limits
description: Learn about the features and limits in various SKUs of Azure Container Registry.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.date: 02/10/2026
ms.service: azure-container-registry
# Customer intent: "As a developer using Azure Container Registry, I want to understand the differences between SKU features and limits, so that I can choose the most appropriate SKU."
---

# Azure Container Registry SKU features and limits

Azure Container Registry is available in multiple SKUs. These SKUs, also known as pricing plans or tiers, support predictable pricing and align to different capacity and usage patterns of your private container registry in Azure.

When you create a registry, you select a **Pricing Plan** that determines the features and limits of your registry. Choose the plan that aligns with your expected usage patterns, such as the number of images, storage needs, and performance requirements.

Azure Container Registry offers three **Pricing Plan** options: Basic, Standard, and Premium. Each SKU offers a different set of features and limits to accommodate various scenarios, from development and testing to production workloads.

| SKU | Description |
| --- | ----------- |
| **Basic** | A cost-optimized entry point for developers learning about Azure Container Registry. Basic registries have most of the same capabilities as Standard and Premium registries, such as Microsoft Entra [authentication integration](container-registry-authentication.md#individual-login-with-azure-ad), [image deletion][container-registry-delete], and [webhooks][container-registry-webhook]. However, the included storage and image throughput are most appropriate for lower usage scenarios, and some features aren't available. |
| **Standard** | Standard registries offer the same capabilities as Basic, with increased included storage and image throughput. Standard registries satisfy the needs of many production scenarios. |
| **Premium** | Premium registries provide the highest amount of included storage and concurrent operations, enabling high-volume scenarios. In addition to higher image throughput, Premium adds features such as [geo-replication][container-registry-geo-replication] for high availability through managing a single registry across multiple regions, [private link with private endpoints](container-registry-private-link.md) to restrict access to the registry, and higher API concurrency and bandwidth throughput for large-scale concurrent deployments. |

Each SKU includes a specific amount of free storage, with additional storage available at a per-GB rate. Each SKU also has a different maximum storage limit.

The Basic, Standard, and Premium SKUs all provide the same programmatic capabilities and data plane APIs. They also all benefit from [image storage][container-registry-storage] managed entirely by Azure. However, the Premium SKU enables a wider range of features and has higher limits.

## SKU features and limits

The following table details the features and registry limits of the Basic, Standard, and Premium SKUs.

[!INCLUDE [container-instances-limits](~/reusable-content/ce-skilling/azure/includes/container-registry/container-registry-limits.md)]

> [!NOTE]
> You can increase some limits listed in this table by contacting [Azure Support](https://azure.microsoft.com/support/create-ticket/). For example, you can request an increase to private endpoint limits, image push and pull performance due to throttling or bandwidth constraints, or general storage limits.

For pricing information on each of the Azure Container Registry SKUs, see [Container Registry pricing][container-registry-pricing]. For details about pricing for data transfers, see [Bandwidth pricing](https://azure.microsoft.com/pricing/details/bandwidth/).

## Registry image pull and push performance limits

API concurrency, bandwidth throughput, and throttling during high-volume operations primarily affect image pull and push performance. Your registry SKU, network configuration, and client configuration determine these factors.

### API concurrency and bandwidth throughput limits

Your SKU determines API concurrency and bandwidth throughput. Higher SKUs support more concurrent operations and greater bandwidth for data-plane operations like listing, deleting, pushing, and pulling images.

The following factors affect API concurrency and bandwidth throughput during image pulls and pushes:

* Number and size of image layers
* Reuse of layers across images in the registry
* Additional API calls required for each operation
* Scale of concurrent deployments, such as Kubernetes deployments pulling images across multiple nodes simultaneously

The following client environment factors affect performance:

* Docker daemon or Podman configuration for concurrent operations
* Container runtime configuration, such as containerd or CRI-O concurrency settings
* Cluster configuration or cluster data plane settings

The following network factors affect performance:

* Network bandwidth and latency for the network hops from clients to the registry
* Client-side network configuration, such as firewall rules and proxy settings
* Geographic distance to the registry or nearest replica if [geo-replicated](container-registry-geo-replication.md)

For more information about API operations that happen during image push and pull, see the [Docker HTTP API V2](https://docs.docker.com/registry/spec/api/) documentation. For help troubleshooting, see [Troubleshoot registry performance](container-registry-troubleshoot-performance.md).

### Throttling and bandwidth constraints

During periods of high request volume, you might encounter throttling with an HTTP 429 `Too many requests` error or slow bandwidth throughput. To mitigate these problems:

* Implement retry logic with exponential backoff.
* Reduce the rate of concurrent requests.
* Space out large-scale deployments to reduce simultaneous image pulls across multiple nodes.

> [!NOTE]
> If you experience persistent API throttling or slow bandwidth throughput, consider [updating your registry's SKU](#change-registry-sku) to a higher one. You can also contact Azure support to [request a limit increase](https://azure.microsoft.com/support/create-ticket/).

## Show registry usage

Usage information helps you make decisions about [changing the SKU](#change-registry-sku) when your registry nears a limit, and helps you [manage consumption](container-registry-best-practices.md#manage-registry-size).

To get a snapshot of your registry's current consumption of storage and other resources, compared with the limits for that registry's SKU, check the **Overview** page of your registry in the Azure portal. You can also use APIs such as [az acr show-usage](/cli/azure/acr#az-acr-show-usage) (Azure CLI), [Get-AzContainerRegistryUsage](/powershell/module/az.containerregistry/get-azcontainerregistryusage) (Azure PowerShell), or [Registries - List Usages](/rest/api/container-registry/registries/list-usages) (REST API).

> [!NOTE]
> The registry's storage usage might not reflect all recent registry operations. Monitor the registry's [`StorageUsed` metric](monitor-service-reference.md#metrics) for up-to-date data.

Depending on your registry's SKU, usage information includes some or all of the following, along with the limit in that SKU:

* Storage consumed in bytes
* Number of [webhooks](container-registry-webhook.md)
* Number of [geo-replications](container-registry-geo-replication.md) (includes the home replica)
* Number of [private endpoints](container-registry-private-link.md)
* Number of [IP access rules](container-registry-access-selected-networks.md)
* Number of [virtual network rules](container-registry-vnet.md)

In a geo-replicated registry, storage usage is shown for the home region. Multiply by the number of replicas for the total amount of storage.

## Change registry SKU

You can change a registry's SKU in the Azure portal or by using [Azure CLI](/cli/azure/acr#az-acr-update) or [Azure PowerShell](/powershell/module/az.containerregistry/update-azcontainerregistry). You can move freely between SKUs as long as the SKU you're switching to has the required maximum storage capacity.

When you change a registry's SKU, there's no downtime or impact on registry operations. However, if you move from Premium to a lower SKU, features specific to premium are disabled. In some cases, you need to remove resources related to these features before you can switch SKUS. For example, you must delete any geo-replications or connected registries before you can switch from Premium to Standard or Basic.

To change SKUs in the Azure portal, go to your container registry. In the service menu, under **Settings**, select **Properties**. Change the option for **Pricing plan**, and then select **Save**.

To change SKUs by using the Azure CLI, use the [az acr update][az-acr-update] command. For example, to switch to Premium:

```azurecli
az acr update --name myContainerRegistry --sku Premium
```

To change SKUs by using Azure PowerShell, use the [Update-AzContainerRegistry][update-azcontainerregistry] cmdlet. For example, to switch to Premium:

```azurepowershell
Update-AzContainerRegistry -ResourceGroupName myResourceGroup -Name myContainerRegistry -Sku Premium
```

## Related content

For information about upcoming Azure Container Registry features, see the [Roadmap][acr-roadmap] on GitHub.

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[update-azcontainerregistry]: /powershell/module/az.containerregistry/update-azcontainerregistry
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-storage]: container-registry-storage.md
[container-registry-delete]: container-registry-delete.md
[container-registry-webhook]: container-registry-webhook.md
