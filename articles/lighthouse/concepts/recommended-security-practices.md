---
title: Recommended security practices
description: When using Azure Lighthouse, it's important to consider security and access control.
ms.date: 01/17/2025
ms.topic: concept-article
---

# Recommended security practices

When using [Azure Lighthouse](../overview.md), it's important to consider security and access control. Users in your tenant will have direct access to customer subscriptions and resource groups, so it's important to take steps that help maintain your tenant's security. We also recommend that you only enable the minimum access necessary to effectively manage your customers' resources. This topic provides recommendations to help you implement these security practices.

> [!TIP]
> These recommendations also apply to [enterprises managing multiple tenants](enterprise.md) with Azure Lighthouse.

<a name='require-azure-ad-multi-factor-authentication'></a>

## Require Microsoft Entra multifactor authentication

[Microsoft Entra multifactor authentication](/entra/identity/authentication/concept-mfa-howitworks) (also known as two-step verification) helps prevent attackers from gaining access to an account by requiring multiple authentication steps. You should require Microsoft Entra multifactor authentication for all users in your managing tenant, including users who will have access to delegated customer resources.

We recommend asking your customers to implement Microsoft Entra multifactor authentication in their tenants as well.

> [!IMPORTANT]
> Conditional access policies that are set on a customer's tenant don't apply to users who access that customer's resources through Azure Lighthouse. Only policies set on the managing tenant apply to those users. We strongly recommend requiring Microsoft Entra multifactor authentication for both the managing tenant and the managed (customer) tenant.

## Assign permissions to groups, using the principle of least privilege

To make management easier, use [Microsoft Entra groups](/entra/fundamentals/concept-learn-about-groups) for each role required to manage your customers' resources. This lets you add or remove individual users to the group as needed, rather than assigning permissions directly to each user.

> [!IMPORTANT]
> In order to add permissions for a Microsoft Entra group, the **Group type** must be set to **Security**. This option is selected when the group is created. For more information, see [Group types](/entra/fundamentals/concept-learn-about-groups#group-types).

When creating your permission structure, be sure to follow the [principle of least privilege](/entra/id-governance/scenarios/least-privileged) so that users only have the permissions needed to complete their job. Limiting permissions for users can help to reduce the chance of inadvertent errors.

For example, you may want to use a structure like this:

|Group name  |Type  |principalId  |Role definition  |Role definition ID  |
|---------|---------|---------|---------|---------|
|Architects     |User group         |\<principalId\>         |Contributor         |b24988ac-6180-42a0-ab88-20f7382dd24c  |
|Assessment     |User group         |\<principalId\>         |Reader         |acdd72a7-3385-48ef-bd42-f606fba81ae7  |
|VM Specialists     |User group         |\<principalId\>         |VM Contributor         |9980e02c-c2be-4d73-94e8-173b1dc7cf3c  |
|Automation     |Service principal name (SPN)         |\<principalId\>         |Contributor         |b24988ac-6180-42a0-ab88-20f7382dd24c  |

After you create these groups, you can assign users as needed. Only add users who truly need to have the access granted by that group.

Be sure to review group membership regularly and remove any users that are no longer necessary to include.

Keep  in mind that when you [onboard customers through a public managed service offer](../how-to/publish-managed-services-offers.md), any group (or user or service principal) that you include will have the same permissions for every customer who purchases the plan. In order to assign different groups to work with different customers, you must publish a separate private plan that is exclusive to each customer, or onboard customers individually by using Azure Resource Manager templates. For example, you could publish a public plan that has very limited access, then work with each customer directly to onboard their resources using a customized Azure Resource Template granting additional access as needed.

> [!TIP]
> You can also create *eligible authorizations* that let users in your managing tenant temporarily elevate their role. By using eligible authorizations, you can minimize the number of permanent assignments of users to privileged roles, helping to reduce security risks related to privileged access by users in your tenant. This feature has specific licensing requirements. For more information, see [Create eligible authorizations](../how-to/create-eligible-authorizations.md).

## Next steps

- Review the [security baseline information](/security/benchmark/azure/baselines/lighthouse-security-baseline) to understand how guidance from the Microsoft cloud security benchmark applies to Azure Lighthouse.
- [Deploy Microsoft Entra multifactor authentication](/entra/identity/authentication/howto-mfa-getstarted).
- Learn about [cross-tenant management experiences](cross-tenant-management-experience.md).
