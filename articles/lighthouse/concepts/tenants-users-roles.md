---
title: Tenants, users, and roles in Azure Lighthouse scenarios
description: Understand how Microsoft Entra tenants, users, and roles can be used in Azure Lighthouse scenarios.
ms.date: 03/27/2025
ms.topic: concept-article
---

# Tenants, users, and roles in Azure Lighthouse scenarios

Before onboarding customers for [Azure Lighthouse](../overview.md), it's important to understand how Microsoft Entra tenants, users, and roles work, and how they can be used in Azure Lighthouse scenarios.

A *tenant* is a dedicated and trusted instance of Microsoft Entra ID. Typically, each tenant represents a single organization. Azure Lighthouse enables [logical projection](architecture.md#logical-projection) of resources from one tenant to another tenant. This allows users in the managing tenant (such as one belonging to a service provider) to access delegated resources in a customer's tenant, or lets [enterprises with multiple tenants centralize their management operations](enterprise.md).

In order to achieve this logical projection, a subscription (or one or more resource groups within a subscription) in the customer tenant must be *onboarded* to Azure Lighthouse. This onboarding process can be done either [through Azure Resource Manager templates](../how-to/onboard-customer.md) or by [publishing a public or private offer to Azure Marketplace](../how-to/publish-managed-services-offers.md).

With either onboarding method, you'll need to define *authorizations*. Each authorization includes a **principalId** (a Microsoft Entra user, group, or service principal in the managing tenant) combined with a built-in role that defines the specific permissions that will be granted for the delegated resources.

> [!NOTE]
> Unless explicitly specified, references to a "user" in the Azure Lighthouse documentation can apply to a Microsoft Entra user, group, or service principal in an authorization.

## Best practices for defining users and roles

When creating your authorizations, we recommend the following best practices:

- In most cases, you'll want to assign permissions to a Microsoft Entra user group or service principal, rather than to a series of individual user accounts. Doing so lets you add or remove access for individual users through your tenant's Microsoft Entra ID, without having to [update the delegation](../how-to/update-delegation.md) every time your individual access requirements change.
- Follow the principle of least privilege. To reduce the chance of inadvertent errors, users should have only the permissions needed to perform their specific job. For more information, see [Recommended security practices](../concepts/recommended-security-practices.md).
- Include an authorization with the [Managed Services Registration Assignment Delete Role](/azure/role-based-access-control/built-in-roles#managed-services-registration-assignment-delete-role) so that you can [remove access to the delegation](../how-to/remove-delegation.md) if needed. If this role isn't assigned, access to delegated resources can only be removed by a user in the customer's tenant.
- Be sure that any user who needs to [view the My customers page in the Azure portal](../how-to/view-manage-customers.md) has the [Reader](/azure/role-based-access-control/built-in-roles#reader) role (or another built-in role that includes Reader access).

> [!IMPORTANT]
> In order to add permissions for a Microsoft Entra group, the **Group type** must be set to **Security**. This option is selected when the group is created. For more information, see [Create a basic group and add members using Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal).

## Role support for Azure Lighthouse

When you define an authorization, each user account must be assigned one of the [Azure built-in roles](/azure/role-based-access-control/built-in-roles). Custom roles and [classic subscription administrator roles](/azure/role-based-access-control/classic-administrators) aren't supported.

All [built-in roles](/azure/role-based-access-control/built-in-roles) are currently supported with Azure Lighthouse, with the following exceptions:

- The [Owner](/azure/role-based-access-control/built-in-roles#owner) role isn't supported.
- The [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) role is supported, but only for the limited purpose of [assigning roles to a managed identity in the customer tenant](../how-to/deploy-policy-remediation.md#create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant). No other permissions typically granted by this role will apply. If you define a user with this role, you must also specify the role(s) that this user can assign to managed identities.
- Any roles with [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission aren't supported.
- Roles that include any of the following [actions](/azure/role-based-access-control/role-definitions#actions) aren't supported:

  - Microsoft.Authorization/*
  - Microsoft.Authorization/*/write
  - Microsoft.Authorization/*/delete
  - Microsoft.Authorization/roleAssignments/write
  - Microsoft.Authorization/roleAssignments/delete
  - Microsoft.Authorization/roleDefinitions/write
  - Microsoft.Authorization/roleDefinitions/delete
  - Microsoft.Authorization/classicAdministrators/write
  - Microsoft.Authorization/classicAdministrators/delete
  - Microsoft.Authorization/locks/write
  - Microsoft.Authorization/locks/delete
  - Microsoft.Authorization/denyAssignments/write
  - Microsoft.Authorization/denyAssignments/delete

> [!IMPORTANT]
> When assigning roles, be sure to review the [actions](/azure/role-based-access-control/role-definitions#actions) specified for each role. Even though roles with [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission aren't supported, there are cases where actions included in a supported role may allow access to data. This generally occurs when data is exposed through access keys, not accessed via the user's identity. For example, the [Virtual Machine Contributor](/azure/role-based-access-control/built-in-roles) role includes the `Microsoft.Storage/storageAccounts/listKeys/action` action, which returns storage account access keys that could be used to retrieve certain customer data.

In some cases, a role that was previously supported with Azure Lighthouse may become unavailable. For example, if the [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission is added to a role that previously didn't have that permission, that role can no longer be used when onboarding new delegations. Users who had already been assigned that role will still be able to work on previously delegated resources, but they won't be able to perform any tasks that use the [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission.

As soon as a new applicable built-in role is added to Azure, it can be assigned when [onboarding a customer using Azure Resource Manager templates](../how-to/onboard-customer.md). There may be a delay before the newly added role becomes available in Partner Center when [publishing a managed service offer](../how-to/publish-managed-services-offers.md). Similarly, if a role becomes unavailable, you may still see it in Partner Center for a while, but you won't be able to publish new offers using such roles.

<a name='transferring-delegated-subscriptions-between-azure-ad-tenants'></a>

## Transferring delegated subscriptions between Microsoft Entra tenants

If a subscription is [transferred to another Microsoft Entra tenant account](/azure/cost-management-billing/manage/billing-subscription-transfer#transfer-a-subscription-to-another-azure-ad-tenant-account), the [registration definition and registration assignment resources](architecture.md#delegation-resources-created-in-the-customer-tenant) created through the [Azure Lighthouse onboarding process](../how-to/onboard-customer.md) are preserved. This means that access granted through Azure Lighthouse to managing tenants remains in effect for that subscription (or for delegated resource groups within that subscription).

The only exception is if the subscription is transferred to a Microsoft Entra tenant to which it had been previously delegated. In this case, the delegation resources for that tenant are removed and the access granted through Azure Lighthouse no longer applies, since the subscription now belongs directly to that tenant (rather than being delegated to it through Azure Lighthouse). However, if that subscription was also delegated to other managing tenants, those other managing tenants will retain the same access to the subscription.

## Next steps

- Learn about [recommended security practices for Azure Lighthouse](recommended-security-practices.md).
- Onboard your customers to Azure Lighthouse, either by [using Azure Resource Manager templates](../how-to/onboard-customer.md) or by [publishing a private or public managed services offer to Azure Marketplace](../how-to/publish-managed-services-offers.md).
