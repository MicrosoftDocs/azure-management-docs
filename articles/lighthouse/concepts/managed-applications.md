---
title: Azure Lighthouse and Azure managed applications
description: Understand how Azure Lighthouse and Azure managed applications can be used together.
ms.date: 01/20/2026
ms.topic: concept-article
# Customer intent: As a service provider, I want to use both Azure Lighthouse and Azure managed applications, so that I can efficiently manage customer resources while maintaining control and access permissions tailored to their specific needs.
---

# Azure Lighthouse and Azure managed applications

Both [Azure managed applications](/azure/azure-resource-manager/managed-applications/overview) and [Azure Lighthouse](../overview.md) enable a service provider to access resources in the customer's tenant. It can be helpful to understand the differences in the way that they work, the scenarios that they enable, and how you can use them together.

> [!TIP]
> Though this article refers to service providers and customers, [enterprises managing multiple tenants](enterprise.md) can use the same processes and tools.

## Comparing Azure Lighthouse and Azure managed applications

This table shows some high-level differences that might affect whether you choose to use Azure Lighthouse or Azure managed applications. In some cases, you could design a solution that uses both.

|Consideration  |Azure Lighthouse  |Azure managed applications  |
|---------|---------|---------|
|Typical user     |Service providers or enterprises managing multiple tenants         |Independent Software Vendors (ISVs)         |
|Scope of cross-tenant access     |Subscriptions and/or resource groups         |Resource group (scoped to a single application)         |
|Purchasable in Microsoft Marketplace     |No (offers can be published to Microsoft Marketplace, but customers are billed separately)        |Yes         |
|IP protection |Yes (IP stays in the service provider's tenant) |Yes (If the ISV chooses to restrict customer access by using deny assignments, the managed resource group is locked to customers) |
|Deny assignments     |No         |Yes        |

### Azure Lighthouse

By using [Azure Lighthouse](../overview.md), a service provider can perform a wide range of management tasks directly on a customer's subscription or resource group. The service provider gains this access through a [logical projection](architecture.md#logical-projection) of resources. Service providers sign in to their own tenant and can access delegated resources in the customer's tenant. The customer chooses which subscriptions or resource groups to delegate to the service provider, keeps full access to those resources, and can remove the service provider's access at any time.

To use Azure Lighthouse, onboard customers by [deploying ARM templates](../how-to/onboard-customer.md) or through a [Managed Service offer in Microsoft Marketplace](managed-services-offers.md). To track your impact on customer engagements, [link your partner ID](/azure/cost-management-billing/manage/link-partner-id).

Typically, service providers use Azure Lighthouse to perform management tasks for a customer on an ongoing basis. For more information about how Azure Lighthouse works at a technical level, see [Azure Lighthouse architecture](architecture.md).

### Azure managed applications

[Azure managed applications](/azure/azure-resource-manager/managed-applications/overview) let an ISV or publisher offer cloud solutions that customers can easily deploy and use in their own subscriptions.

In a managed application, you bundle together the resources used by the application and deploy them to a resource group that the ISV or publisher manages. This "managed resource group" exists in the customer's subscription, but identities in the publisher's tenant can access it. When you publish an offer in Microsoft Partner Center, you choose whether to enable or disable management access by the publisher. You can restrict customer access by using deny assignments, or you can grant the customer full access.

Managed applications support [customized Azure portal experiences](/azure/azure-resource-manager/managed-applications/concepts-view-definition) and [integration with custom providers](/azure/azure-resource-manager/managed-applications/tutorial-create-managed-app-with-custom-provider). These options let you deliver a more customized and integrated experience, and can make it easier for customers to perform some management tasks themselves.

You can [publish managed applications to Microsoft Marketplace](/azure/marketplace/azure-app-offer-setup) as private offers for specific customers, or as public offers that multiple customers can purchase. You can also deliver managed applications to users within your organization by [publishing managed applications to your service catalog](/azure/azure-resource-manager/managed-applications/publish-service-catalog-app). You can deploy both service catalog and Marketplace instances by using ARM templates, which can include a commercial marketplace partner's unique identifier to track [customer usage attribution](/azure/marketplace/azure-partner-customer-usage-attribution).

Azure managed applications typically address a specific customer need through a turnkey solution that the service provider fully manages.

## Using Azure Lighthouse and Azure managed applications together

While Azure Lighthouse and Azure managed applications use different access mechanisms to achieve different goals, there might be scenarios where it makes sense for a service provider to use both of them with the same customer.

For example, a customer might want managed services delivered by a service provider through Azure Lighthouse, so they have visibility into the partner's actions along with continued control of their delegated subscription. However, the service provider might not want the customer to access certain resources that the service provider stores in the customer's tenant, or they might want to prevent customized actions on those resources. To meet these goals, the service provider can publish a private offer as a managed application. The managed application can include a resource group that is deployed in the customer's tenant, but that the customer can't access directly.

Customers might also be interested in managed applications from multiple service providers, whether or not they also use managed services via Azure Lighthouse from any of those service providers. Additionally, partners in the Cloud Solution Provider (CSP) program can resell certain managed applications published by other ISVs to customers that they support through Azure Lighthouse. With a wide range of options, service providers can choose the right balance to meet their customers' needs, while restricting access to resources when appropriate.

## Next steps

- Learn about [Azure managed applications](/azure/azure-resource-manager/managed-applications/overview).
- Learn how to [onboard a subscription to Azure Lighthouse](../how-to/onboard-customer.md).
- Learn about [ISV scenarios with Azure Lighthouse](isv-scenarios.md).
