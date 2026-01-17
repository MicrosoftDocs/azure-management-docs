---
title: Managed Service offers in Microsoft Marketplace
description: Offer your Azure Lighthouse management services to customers through Managed Services offers in Microsoft Marketplace.
ms.date: 01/16/2026
ms.topic: concept-article
# Customer intent: As a service provider, I want to create and publish Managed Service offers in Microsoft Marketplace so that I can deliver resource management services to customers and streamline their onboarding to Azure Lighthouse.
---

# Managed Service offers in Microsoft Marketplace

You can provide resource management services to customers through [Azure Lighthouse](../overview.md) by publishing **Managed Service** offers to [Microsoft Marketplace](https://marketplace.microsoft.com). You can make an offer available to all potential customers, or publish a private offer for one or more specific customers. Since you bill customers directly for costs related to these managed services, Microsoft doesn't charge any fees.

## Understand Managed Service offers

Managed Service offers streamline the process of onboarding customers to Azure Lighthouse. When a customer purchases an offer in Microsoft Marketplace, they can specify which subscriptions and/or resource groups to onboard.

For each offer, you define the access that users in your organization will have to work on resources in the customer tenant. You set up this access through a manifest that specifies the Microsoft Entra users, groups, and service principals that can access customer resources, along with [roles that define their level of access](tenants-users-roles.md#role-support-for-azure-lighthouse).

> [!NOTE]
> Managed Service offers might not be available in Azure Government and other national clouds.

## Public and private plans

Each Managed Service offer includes one or more plans. Plans can be either private or public.

If you want to limit your offer to specific customers, publish a private plan. When you do so, customers can only purchase the plan for the specific subscription IDs that you provide. For more info, see [Private plans](/partner-center/marketplace/private-plans).

> [!NOTE]
> Private plans don't support subscriptions established through a reseller of the Cloud Solution Provider (CSP) program.

Promote your services to new customers by using public plans. These plans are usually more appropriate when you only require limited access to the customer's tenant. Once you establish a relationship with a customer, if they decide to grant your organization additional access, you can enable that access either by publishing a new private plan for that customer only, or by [onboarding them for further access using Azure Resource Manager templates](../how-to/onboard-customer.md).

If appropriate, you can include both public and private plans in the same offer.

> [!IMPORTANT]
> After you publish a plan as public, you can't change it to private. To control which customers can accept your offer and delegate resources, use a private plan. With a public plan, you can't restrict availability to certain customers or even to a certain number of customers, although you can choose to stop selling the plan completely.
>
> After a customer accepts an offer, you can [remove access to a delegation](../how-to/remove-delegation.md) only if you included an **Authorization** with the **Role Definition** set to [Managed Services Registration Assignment Delete Role](/azure/role-based-access-control/built-in-roles#managed-services-registration-assignment-delete-role) when you published the offer. You can also reach out to the  customer and ask them to [remove your access](../how-to/view-manage-service-providers.md#remove-service-provider-offers).

## Publish Managed Service offers

For more information about publishing a Managed Service offer, see [Publish a Managed Service offer to Microsoft Marketplace](../how-to/publish-managed-services-offers.md).

## Next steps

- Learn about Azure Lighthouse [architecture](architecture.md) and [cross-tenant management experiences](cross-tenant-management-experience.md).
- Learn about [Microsoft Marketplace](/partner-center/marketplace/overview).
- [Publish Managed Service offers](../how-to/publish-managed-services-offers.md) to Microsoft Marketplace.
