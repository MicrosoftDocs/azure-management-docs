---
title: Tenants, users, and roles in Azure Lighthouse scenarios
description: Understand how Microsoft Entra tenants, users, and roles can be used in Azure Lighthouse scenarios.
ms.date: 04/06/2026
ms.topic: concept-article
# Customer intent: "As an IT administrator managing multiple tenants, I want to understand how to utilize Microsoft Entra users and roles within Azure Lighthouse, so that I can efficiently delegate resource access and ensure secure management of customer environments."
---

# Tenants, users, and roles in Azure Lighthouse scenarios

Before onboarding customers for [Azure Lighthouse](../overview.md), it's important to understand how Microsoft Entra tenants, users, and roles work, and how they can be used in Azure Lighthouse scenarios.

A *tenant* is a dedicated and trusted instance of Microsoft Entra ID. Typically, each tenant represents a single organization. Azure Lighthouse enables [logical projection](architecture.md#logical-projection) of resources from one tenant to another tenant. Users in the managing tenant, such as one belonging to a service provider, can access delegated resources in a customer's tenant. [Enterprises with multiple tenants](enterprise.md) can also use Azure Lighthouse to centralize their management operations.

To achieve this logical projection, you must *onboard* a subscription (or one or more resource groups within a subscription) in the customer tenant to Azure Lighthouse. You can complete the onboarding process either [through Azure Resource Manager templates](../how-to/onboard-customer.md) or by [publishing a public or private offer to Microsoft Marketplace](../how-to/publish-managed-services-offers.md).

With either onboarding method, you need to define *authorizations*. Each authorization includes a **principalId** (a Microsoft Entra user, group, or service principal in the managing tenant) combined with a built-in role that defines the specific permissions granted for the delegated resources.

> [!NOTE]
> Unless explicitly specified, references to a "user" in the Azure Lighthouse documentation can apply to a Microsoft Entra user, group, or service principal in an authorization.

## Best practices for defining Azure Lighthouse users and roles

When creating your authorizations, follow these best practices:

- Whenever possible, assign permissions to a Microsoft Entra user group or service principal, rather than to a series of individual user accounts. By using this approach, you can add or remove access for individual users through your tenant's Microsoft Entra ID without needing to [update the delegation](../how-to/update-delegation.md) every time your individual access requirements change.
- Follow the principle of least privilege. To reduce the chance of inadvertent errors, users should have only the permissions needed to perform their specific job. For more information, see [Recommended security practices](../concepts/recommended-security-practices.md).
- Include an authorization with the [Managed Services Registration Assignment Delete Role](/azure/role-based-access-control/built-in-roles#managed-services-registration-assignment-delete-role) so that you can [remove access to the delegation](../how-to/remove-delegation.md) if needed. If you don't assign this role, only a user in the customer's tenant can remove access to delegated resources.
- Ensure that any user who needs to [view the My customers page in the Azure portal](../how-to/view-manage-customers.md) has the [Reader](/azure/role-based-access-control/built-in-roles#reader) role (or another built-in role that includes Reader access).

> [!IMPORTANT]
> To add permissions for a Microsoft Entra group, set the [**Group type**](/entra/fundamentals/concept-learn-about-groups#group-types) to **Security**. You select this option when you [create the group](/entra/fundamentals/how-to-manage-groups#create-a-basic-group-and-add-members).

## Role support for Azure Lighthouse

When you define an authorization, you assign each user account one of the [Azure built-in roles](/azure/role-based-access-control/built-in-roles). Azure Lighthouse doesn't support custom roles or [classic subscription administrator roles](/azure/role-based-access-control/classic-administrators).

Azure Lighthouse supports all [built-in roles](/azure/role-based-access-control/built-in-roles), except for the following roles:

- The [Owner](/azure/role-based-access-control/built-in-roles#owner) role isn't supported.
- The [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) role is supported, but only for the limited purpose of [assigning roles to a managed identity in the customer tenant](../how-to/deploy-policy-remediation.md#create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant). No other permissions typically granted by this role apply. If you define a user with this role, you must also specify the roles that this user can assign to managed identities.
- Roles with [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission aren't supported.
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
> When assigning roles, review the [actions](/azure/role-based-access-control/role-definitions#actions) specified for each role. Even though Azure Lighthouse doesn't support roles with [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission, some actions included in a supported role might allow access to data. This access generally occurs when data is exposed through access keys, not accessed via the user's identity. For example, the [Virtual Machine Contributor](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor) role includes the `Microsoft.Storage/storageAccounts/listKeys/action` action, which returns storage account access keys that could be used to retrieve certain customer data.

In some cases, a role that Azure Lighthouse previously supported becomes unavailable. For example, if the [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission is added to a role that previously didn't have that permission, you can't use that role when onboarding new delegations. Users who are already assigned that role can still work on previously delegated resources, but they can't perform any tasks that use the [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission.

As soon as Microsoft adds a new applicable built-in role to Azure, you can assign it when [onboarding a customer using Azure Resource Manager templates](../how-to/onboard-customer.md). There might be a delay before the newly added role becomes available in Partner Center when [publishing a managed service offer](../how-to/publish-managed-services-offers.md). Similarly, if a role becomes unavailable, you might still see it in Partner Center for a while, but you can't publish new offers that use such roles.

<a name='transferring-delegated-subscriptions-between-azure-ad-tenants'></a>

## Transferring delegated subscriptions between Microsoft Entra tenants

If you [transfer a subscription to another Microsoft Entra tenant account](/azure/cost-management-billing/manage/billing-subscription-transfer#transfer-a-subscription-to-another-microsoft-entra-tenant-account), the [registration definition and registration assignment resources](architecture.md#delegation-resources-created-in-the-customer-tenant) that the [Azure Lighthouse onboarding process](../how-to/onboard-customer.md) creates stay intact. This preservation means that the access you grant through Azure Lighthouse to managing tenants continues for that subscription (or for delegated resource groups within that subscription).

The only exception is if you transfer the subscription to a Microsoft Entra tenant to which it was previously delegated. In this case, the delegation resources for that tenant are removed and the access granted through Azure Lighthouse no longer applies, since the subscription now belongs directly to that tenant (rather than being delegated to it through Azure Lighthouse). However, if you also delegated that subscription to other managing tenants, those other managing tenants keep the same access to the subscription.

## Next steps

- Learn about [recommended security practices for Azure Lighthouse](recommended-security-practices.md).
- Onboard your customers to Azure Lighthouse, either by [using Azure Resource Manager templates](../how-to/onboard-customer.md) or by [publishing a private or public managed services offer to Microsoft Marketplace](../how-to/publish-managed-services-offers.md).
