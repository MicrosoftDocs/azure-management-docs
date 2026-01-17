---
title: Publish a Managed Service offer to Microsoft Marketplace
description: Learn how to publish a Managed Service offer that onboards customers to Azure Lighthouse.
ms.date: 01/16/2026
ms.topic: how-to
# Customer intent: "As a service provider, I want to publish a Managed Service offer to Microsoft Marketplace, so that I can onboard customers and manage their subscriptions or resource groups efficiently through Azure Lighthouse."
---

# Publish a Managed Service offer to Microsoft Marketplace

This article describes how to publish a public or private Managed Service offer to [Microsoft Marketplace](https://marketplace.microsoft.com) by using the [commercial marketplace](/partner-center/marketplace-offers/overview) program in Partner Center. Customers who purchase the offer can delegate subscriptions or resource groups, so you can manage them through [Azure Lighthouse](../overview.md) according to the access you specify in the offer.

## Managed Service offer publishing requirements

You must have a valid [Microsoft Marketplace account in Partner Center](/partner-center/account-settings/create-account) to create and publish offers. If you don't have an account, the [sign-up process](https://aka.ms/joinmarketplace) guides you through the steps of creating an account in Partner Center and enrolling in the commercial marketplace program.

Per the [Managed Service offer certification requirements](/legal/marketplace/certification-policies#700-managed-services), you must have [Solutions Partner designation](/partner-center/partner-capability-score) for Infrastructure (Azure) or Security to publish a Managed Service offer.

## Decide between Managed Service offers and ARM template onboarding

If you don't want to publish a Managed Service offer to Microsoft Marketplace, or if you don't meet all the requirements, you can [onboard customers to Azure Lighthouse manually by using Azure Resource Manager templates](onboard-customer.md). Use the following table to help you determine whether to onboard customers by publishing a Managed Service offer or by using ARM templates.

|**Consideration**  |**Managed Service offer**  |**ARM templates**  |
|---------|---------|---------|
|Requires [Microsoft Marketplace account in Partner Center](/partner-center/account-settings/create-account)   |Yes         |No        |
|Requires [Solutions Partner designation](/partner-center/partner-capability-score) for Infrastructure (Azure) or Security      |Yes         |No         |
|Available to new customers through Microsoft Marketplace     |Yes     |No       |
|Can limit offer to specific customers     |Yes (only with private plans, which can't be used with subscriptions established through a reseller of the Cloud Solution Provider (CSP) program)         |Yes         |
|Can [automatically connect customers to your CRM system](/partner-center/marketplace-offers/plan-managed-service-offer#customer-leads) |Yes  |No   |
|Requires customer acceptance in Azure portal     |Yes     |No   |
|Can use automation to onboard multiple subscriptions, resource groups, or customers |No     |Yes    |
|Immediate access to new built-in roles and Azure Lighthouse features     |Not always (generally available after some delay)         |Yes         |
|Customers can review and accept updated offers in the Azure portal | Yes | No |

> [!NOTE]
> Managed Service offers might not be available in Azure Government and other national clouds.

## Create your Managed Service offer

For detailed instructions about how to create your offer, including all of the information and assets you need to provide, see [Create a Managed Service offer](/partner-center/marketplace-offers/create-managed-service-offer).

To learn about the general publishing process, review the [Microsoft Marketplace documentation](/partner-center/marketplace-offers/overview). You should also review the [Microsoft Marketplace certification policies](/legal/marketplace/certification-policies), particularly the [Managed Services](/legal/marketplace/certification-policies#700-managed-services) section.

When a customer adds your offer, they can delegate one or more subscriptions or resource groups. The delegated resources are then [onboarded to Azure Lighthouse](#customer-onboarding-process-for-managed-service-offers).

> [!IMPORTANT]
> Each plan in a Managed Service offer includes a **Manifest Details** section. In this section, you define the Microsoft Entra entities in your tenant that will have access to the delegated resource groups and subscriptions for customers who purchase that plan. It's important to be aware that permissions for any group, user, or service principal will apply to every customer who purchases the plan.
>
> To assign different groups to work with each customer, publish a separate [private plan](/partner-center/marketplace-offers/private-plans) that is exclusive to each customer. These private plans don't support subscriptions established through a reseller of the Cloud Solution Provider (CSP) program.

## Publish your Managed Service offer

When you complete all of the sections, publish the offer. After you initiate the publishing process, your offer goes through several validation and publishing steps. For more information, see [Review and publish an offer to Microsoft Marketplace](/partner-center/marketplace-offers/review-publish-offer).

You can [publish an updated version of your offer](/partner-center/marketplace-offers/update-existing-offer) at any time. For example, you might want to add a new role definition to a previously published offer. When you update the offer, customers who already added that offer see an icon in the **Service providers** page in the Azure portal to let them know an update is available. Each customer can [review the changes and choose whether to update to the new version](view-manage-service-providers.md#update-service-provider-offers).

## Customer onboarding process for Managed Service offers

After a customer adds your offer, they can [delegate one or more specific subscriptions or resource groups](view-manage-service-providers.md#delegate-resources) to onboard to Azure Lighthouse. If a customer accepts an offer but doesn't delegate any resources, they see a note at the top of the **Service provider offers** section of the **Service providers** page in the Azure portal.

> [!IMPORTANT]
> Delegation must be done by an account in the customer's tenant who has a role with the `Microsoft.Authorization/roleAssignments/write`, `Microsoft.Authorization/roleAssignments/delete`, and `Microsoft.Authorization/roleAssignments/read` permissions, such as [Owner](/azure/role-based-access-control/built-in-roles#owner), for the subscription to onboard (or which contains the resource groups to onboard). To find users who can delegate the subscription, a user in the customer's tenant can select the subscription in the Azure portal, open **Access control (IAM)**, and [view all users with the Owner role](/azure/role-based-access-control/role-assignments-list-portal#list-owners-of-a-subscription).

When the customer delegates a subscription (or one or more resource groups within a subscription), the process automatically registers the **Microsoft.ManagedServices** resource provider for that subscription. Users in your tenant can access the delegated resources according to the authorizations that you defined in your offer.

> [!NOTE]
> To delegate more subscriptions or resource groups to the same offer at a later time, the customer must [manually register the **Microsoft.ManagedServices** resource provider](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider) on each subscription before delegating.

If you publish an updated version of your offer, the customer can [review the changes in the Azure portal and accept the new version](view-manage-service-providers.md#update-service-provider-offers).

## Next steps

- Learn about [Microsoft Marketplace](/partner-center/marketplace-offers/overview).
- Learn about [cross-tenant management experiences in Azure Lighthouse](../concepts/cross-tenant-management-experience.md).
- [View and manage customers](view-manage-customers.md) by going to **My customers** in the Azure portal.
