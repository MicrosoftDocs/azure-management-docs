---
title: Mitigate Data Exfiltration with Dedicated Data Endpoints
description: Azure Container Registry is introducing dedicated data endpoints available to mitigate data-exfiltration concerns.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 02/24/2026
# Customer intent: As a security specialist, I want to implement dedicated data endpoints in Azure Container Registry, so that I can minimize data exfiltration risks and ensure secure access to storage resources.
---
# Mitigate data exfiltration with dedicated data endpoints in Azure Container Registry

Dedicated data endpoints in Azure Container Registry enable tightly scoped client firewall rules to specific registries, minimizing data exfiltration concerns.

Dedicated data endpoints feature is available for registries in the **Premium** [service tier](includes/container-registry-quickstart-sku.md). For pricing information, see [container-registry-pricing.](https://azure.microsoft.com/pricing/details/container-registry/)

Pulling content from a registry involves two endpoints:

*Registry endpoint*, often referred to as the login URL, used for authentication and content discovery. A command like docker pulls `contoso.azurecr.io/hello-world` makes a REST request that authenticates and negotiates the layers, which represent the requested artifact.
*Data endpoints* serve blobs representing content layers.

:::image type="content" source="./media/dedicated-data-endpoints/endpoints.png" alt-text="Diagram to illustrate endpoints.":::

The registry service manages the data endpoint storage accounts. Benefits of these managed storage accounts include load balancing, contentious content splitting, multiple copies for higher concurrent content delivery, and multi-region support with [geo-replication](container-registry-geo-replication.md).

## Azure Private Link virtual network support

[Azure Private Link virtual network support](container-registry-private-link.md) enables private endpoints for the managed registry service from Azure Virtual Networks. In this case, both the registry and data endpoints are accessible from within the virtual network, using private IPs.

Once the managed registry service and storage accounts are both secured to access from within the virtual network, then the public endpoints are removed.

:::image type="content" source="./media/dedicated-data-endpoints/v-net.png" alt-text="Diagram to illustrate virtual network support.":::

However, virtual network connection isn’t always an option.

> [!IMPORTANT]
> [Azure Private Link](container-registry-private-link.md) is the most secure way to control network access between clients and the registry, as network traffic is limited to the Azure Virtual Network using private IP addresses. When Private Link isn’t an option, dedicated data endpoints can provide secure knowledge in what resources are accessible from each client.

## Client firewall rules and data exfiltration risks

Client firewall rules limit access to specific resources. The firewall rules apply while connecting to a registry from on-premises hosts, IoT devices, custom build agents. These rules also apply when Private Link support isn't an option.

:::image type="content" source="./media/dedicated-data-endpoints/client-firewall-0.png" alt-text="Diagram to illustrate client firewall rules.":::

For example, you might create a rule with a wildcard for all storage accounts, which raises data exfiltration concerns. A bad actor could deploy code that would be capable of writing to the storage account.

:::image type="content" source="./media/dedicated-data-endpoints/client-firewall-2.png" alt-text="Diagram to illustrate client data exfiltration risks.":::

To address these concerns and mitigate the risk of data exfiltration, use the dedicated data endpoint feature of Azure Container Registry.

## Dedicated data endpoints

Dedicated data endpoints help retrieve layers from the Azure Container Registry service, with fully qualified domain names representing the registry domain.

As any registry could be geo-replicated, a regional pattern is used: `[registry].[region].data.azurecr.io`.

For the Contoso example, multiple regional data endpoints are added supporting the local region with a nearby replica.

With dedicated data endpoints, the bad actor is blocked from writing to other storage accounts.

:::image type="content" source="./media/dedicated-data-endpoints/contoso-example-0.png" alt-text="Diagram to illustrate contoso example with dedicated data endpoints.":::

To enable dedicated data endpoints, see [Configure rules to access an Azure Container Registry behind a firewall](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints).

## Next steps

* Learn how to access an Azure container registry from behind a [firewall rule](container-registry-firewall-access-rules.md).
* Connect Azure Container Registry using [Azure Private Link](container-registry-private-link.md).
