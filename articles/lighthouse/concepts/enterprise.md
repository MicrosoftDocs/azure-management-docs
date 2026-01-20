---
title: Azure Lighthouse in enterprise scenarios
description: The capabilities of Azure Lighthouse can be used to simplify cross-tenant management within an enterprise which uses multiple Microsoft Entra tenants.
ms.date: 03/28/2025
ms.topic: concept-article
# Customer intent: As an enterprise IT manager, I want to implement Azure Lighthouse for cross-tenant management, so that I can efficiently manage resources across multiple Microsoft Entra tenants while ensuring security and compliance.
---

# Azure Lighthouse in enterprise scenarios

A common scenario for [Azure Lighthouse](../overview.md) involves a service provider that manages resources in its customers' Microsoft Entra tenants. The capabilities of Azure Lighthouse can also be used to simplify cross-tenant management within an enterprise that uses multiple Microsoft Entra tenants. In this scenario, users in one of the enterprise's tenants can perform management tasks on the other tenants through Azure Lighthouse, without requiring any other service provider to be involved.

## Single vs. multiple tenants

For most organizations, management is easier with a single Microsoft Entra tenant. Having all resources within one tenant allows centralization of management tasks by designated users, user groups, or service principals within that tenant. We recommend using one tenant for your organization whenever possible.

Some organizations may need to use multiple Microsoft Entra tenants. This might be a temporary situation, as when acquisitions have taken place and a long-term tenant consolidation strategy isn't yet defined. Other times, organizations may need to maintain multiple tenants on an ongoing basis due to wholly independent subsidiaries, geographical or legal requirements, or other considerations.

In cases where a [multitenant architecture](/azure/architecture/guide/multitenant/overview) is required, Azure Lighthouse can help centralize and streamline management operations. By using Azure Lighthouse, users in one managing tenant can perform [cross-tenant management functions](cross-tenant-management-experience.md) in a centralized, scalable manner.

## Tenant management architecture

To use Azure Lighthouse in an enterprise, you need to determine which tenant will include the users who perform management operations on the other tenants. In other words, you designate one tenant as the managing tenant for the other tenants.

For example, say your organization has a single tenant that we’ll call *Tenant A*. Your organization then acquires *Tenant B* and *Tenant C*, and you have business reasons that require you to maintain them as separate tenants. However, you want to use the same policy definitions, backup practices, and security processes for all of them, with management tasks performed by the same set of users.

Since Tenant A already includes users in your organization who have been performing those tasks for Tenant A, you can designate Tenant A as the management tenant. Then you can [onboard subscriptions](../how-to/onboard-customer.md) within Tenant B and Tenant C so that they're delegated to Tenant A. During the onboarding process, you create authorizations that grant permissions to users in Tenant A, allowing them to perform management tasks across Tenant B and Tenant C.

:::image type="content" source="../media/enterprise-azure-lighthouse.jpg" alt-text="Diagram showing users in Tenant A managing resources in Tenant B and Tenant C.":::

## Security and access considerations

In most enterprise scenarios, you'll want to delegate a full subscription to Azure Lighthouse. You can also choose to delegate only specific resource groups within a subscription.

Either way, be sure to [follow the principle of least privilege](recommended-security-practices.md#assign-permissions-to-groups-using-the-principle-of-least-privilege) when defining which users can access delegated resources. Doing so helps to ensure that users only have the permissions needed to perform the required tasks and reduces the chance of inadvertent errors.

Azure Lighthouse only provides logical links between a managing tenant and managed tenants, rather than physically moving data or resources. Furthermore, the access always goes in only one direction, from the managing tenant to the managed tenants. Users and groups in the managing tenant should use multifactor authentication when performing management operations on managed tenant resources.

Enterprises with internal or external governance and compliance guardrails can use [Azure Activity logs](/azure/azure-monitor/essentials/activity-log) to meet their transparency requirements. When enterprises establish managing and managed tenant relationships, users in each tenant can view logged activity to [see actions taken by users in the managing tenant](../how-to/view-service-provider-activity.md).

For more information, see [Recommended security practices](recommended-security-practices.md).

## Onboarding considerations

Subscriptions (or resource groups within a subscription) can be onboarded to Azure Lighthouse either by deploying Azure Resource Manager templates or through Managed Services offers published to Microsoft Marketplace.

Since enterprise users typically have direct access to the enterprise’s tenants, and there's no need to market or promote a management offering, it's usually faster and more straightforward to deploy Azure Resource Manager templates. While the [onboarding guidance](../how-to/onboard-customer.md) refers to service providers and customers, enterprises can use the same processes to onboard their tenants.

If you prefer, tenants within an enterprise can be onboarded by [publishing a Managed Services offer to Microsoft Marketplace](../how-to/publish-managed-services-offers.md). To ensure that the offer is only available to the appropriate tenants, be sure that your [plans are set to private](/partner-center/marketplace-offers/create-managed-service-offer-plans#make-your-plan-private). With a private plan, you provide the subscription IDs for each tenant that you plan to onboard, and no one else will be able to get your offer.

## Microsoft Entra External ID

[Microsoft Entra External ID](/entra/external-id/customers/overview-customers-ciam) provides business-to-customer identity as a service. When you delegate a resource group through Azure Lighthouse, you can use Azure Monitor to route Microsoft Entra External ID sign-in and auditing logs to different monitoring solutions. You can retain the logs for long-term use, or integrate with third-party security information and event management (SIEM) tools to gain insights into your environment.

For more information, see [Set up Azure Monitor in external tenants](/entra/external-id/customers/how-to-azure-monitor).

## Terminology notes

For cross-tenant management within the enterprise, references to service providers in the Azure Lighthouse documentation can be understood to apply to the managing tenant within an enterprise—that is, the tenant that includes the users who will manage resources in other tenants through Azure Lighthouse. Similarly, any references to customers can be understood to apply to the tenants that are delegating resources to be managed through users in the managing tenant.

For instance, in the example described above, Tenant A can be thought of as the service provider tenant (the managing tenant) and Tenant B and Tenant C can be thought of as the customer tenants.

Continuing with that example, Tenant A users with the appropriate permissions can [view and manage delegated resources](../how-to/view-manage-customers.md) in the **My customers** page of the Azure portal. Likewise, Tenant B and Tenant C users with the appropriate permissions can [view and manage details about their delegations](../how-to/view-manage-service-providers.md) in the **Service providers** page of the Azure portal.

## Next steps

- Explore options for [resource organization in multitenant architectures](/azure/architecture/guide/multitenant/approaches/resource-organization).
- Learn about [cross-tenant management experiences](cross-tenant-management-experience.md).
- Learn more about [how Azure Lighthouse works](architecture.md).
