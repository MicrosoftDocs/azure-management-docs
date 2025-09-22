---
title: View and manage service providers
description: Customers can view info about Azure Lighthouse service providers, service provider offers, and delegated resources in the Azure portal.
ms.date: 09/22/2025
ms.topic: how-to
# Customer intent: As a cloud administrator, I want to view and manage Azure Lighthouse service providers and their offers in the Azure portal, so that I can control access and delegate resources effectively while ensuring compliance with my organization's resource management policies.
---

# View and manage service providers

Customers can visit **Service providers** in the [Azure portal](https://portal.azure.com) for control and visibility of their service providers who use [Azure Lighthouse](../overview.md). In **Service providers**, customers can delegate specific resources, review new or updated offers, remove service provider access, and more.

To access **Service providers** in the Azure portal, enter "Service providers" in the search box in the Azure portal page header. You can also navigate to **Azure Lighthouse** in the Azure portal, then select **View service provider offers**.

> [!NOTE]
> To view **Service providers**, a user in the customer's tenant must have the [Reader built-in role](/azure/role-based-access-control/built-in-roles#reader) (or another built-in role which includes Reader access).
>
> To add or update offers, delegate resources, and remove offers, the user must have a role with the `Microsoft.Authorization/roleAssignments/write`, `Microsoft.Authorization/roleAssignments/delete`, and `Microsoft.Authorization/roleAssignments/read` permissions, such as [Owner](/azure/role-based-access-control/built-in-roles#owner).

**Service providers** only shows information about the service providers that have access to the customer's subscriptions or resource groups through Azure Lighthouse. It doesn't show information about any additional service providers who don't use Azure Lighthouse.

## View service provider details

To view details about the current service providers who use Azure Lighthouse to work on the customer's tenant, select **Service provider offers** from the service menu of **Service providers**.

Each offer shows the service provider's name and the offer associated with it. Select an offer to view a description and other details, including the role assignments that the service provider was granted.

In the **Delegations** column for an offer you can see how many subscriptions and/or resource groups were delegated to the service provider. The service provider can work on these subscriptions and/or resource groups according to the access levels specified in the offer.

## Add service provider offers

You can add a new service provider offer in **Service provider offers**.

To add an offer from the marketplace, select **Add offer** in the command bar, then choose **Add via marketplace**. To view [Managed Service offers](../concepts/managed-services-offers.md) that were published specifically for a customer, select **Private products**. Otherwise, search for a public offer. When you find the offer you're interested in, select it to review details. To add the offer, select **Create**, then provide any necessary information.

To add an offer from a template, select **Add offer** in the command bar, then choose **Add via template**. The **Upload Offer Template** pane appears, allowing you to upload a template from your service provider and onboard your subscription (or resource group). For detailed steps, see [Deploy the Azure Resource Manager template](onboard-customer.md#deploy-the-azure-resource-manager-template).

## Update service provider offers

After a customer adds an offer, a service provider may publish an updated version of the same offer to Azure Marketplace, such as to add a new role definition. If a new version of the offer is published, an "update" icon appears in the row for that offer. Select this icon to see the changes in the current version of the offer.

 ![Update offer icon](../media/update-offer.jpg)

After reviewing the changes, you can choose to update to the new version. When you do so, the authorizations and other settings specified in the new version apply to any subscriptions and/or resource groups that were previously delegated for that offer.

## Remove service provider offers

You can remove a service provider offer at any time by selecting the trash can icon in the row for that offer.

Once you confirm the deletion, that service provider can no longer access the resources that were formerly delegated for that offer.

> [!IMPORTANT]
> If a subscription has two or more offers from the same service provider, removing one of them could cause some service provider users to lose the access granted via the other delegations. This problem only occurs when the same user and role are included in multiple delegations, then one of the delegations is removed. To restore access, the [onboarding process](onboard-customer.md) should be repeated for the offers that you don't want to remove.

## Delegate resources

Before a service provider can access and manage a customer's resources, one or more specific subscriptions and/or resource groups must be delegated. When a customer adds an offer without delegating any resources, a note appears at the top of the **Service provider offers** section. The service provider can't work on any resources in the customer's tenant until the delegation is completed.

To delegate subscriptions or resource groups:

1. Check the box for the row containing the service provider, offer, and name. Then select **Delegate resources** at the top of the screen.
1. In the **Offer details** section of the **Delegate resources** pane, review the details about the service provider and offer. To review role assignments for the offer, select **Click here to see the details of the selected offer**.
1. In the **Delegate** section, select **Delegate subscriptions** or **Delegate resource groups**.
1. Choose the subscriptions and/or resource groups you'd like to delegate for this offer, then select **Add**.
1. Select the checkbox to confirm that you want to grant this service provider access to these resources, then select **Delegate**.

## View delegations

Delegations represent an association of specific customer resources (subscriptions and/or resource groups) with role assignments that grant permissions to the service provider for those resources. To view delegation details, select **Delegations** from the service menu.

Filters at the top of the pane let you sort and group your delegation information. You can also filter by specific service providers, offers, or keywords.

> [!NOTE]
> When [viewing role assignments for the delegated scope in the Azure portal](/azure/role-based-access-control/role-assignments-list-portal#list-role-assignments-at-a-scope) or via APIs, customers can't see role assignments for users from the service provider tenant who have access through Azure Lighthouse. Similarly, users in the service provider tenant can't see role assignments for users in a customer's tenant, regardless of the role they've been assigned.
>
> [Classic administrator](/azure/role-based-access-control/classic-administrators) assignments in a customer tenant may be visible to users in the managing tenant, or the other way around, because classic administrator roles don't use the Resource Manager deployment model.

## Audit and restrict delegations in your environment

Customers may want to review all subscriptions and/or resource groups that are delegated to Azure Lighthouse, or place restrictions on the tenants to which they can be delegated. These options are especially useful for customers with a large number of subscriptions, or who have many users who perform management tasks.

We provide an [Azure Policy built-in policy definition](/azure/governance/policy/samples/built-in-policies#lighthouse) to [audit delegation of scopes to a managing tenant](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/Delegations_Audit.json). You can assign this policy to a management group that includes all of the subscriptions that you want to audit. When you check for compliance with this policy, any delegated subscriptions and/or resource groups within that management group are shown in a noncompliant state. You can then review the results and confirm that there are no unexpected delegations.

Another [built-in policy definition](/azure/governance/policy/samples/built-in-policies#lighthouse) lets you [restrict delegations to specific managing tenants](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/AllowCertainManagingTenantIds_Deny.json). This policy can be assigned to a management group that includes all subscriptions for which you want to limit delegations. After the policy is deployed, any attempts to delegate a subscription to a tenant outside of the ones you specify are denied.

For more information about how to assign a policy and view compliance state results, see [Quickstart: Create a policy assignment](/azure/governance/policy/assign-policy-portal).

## Next steps

- Learn more about [Azure Lighthouse](../overview.md).
- Learn how to [audit service provider activity](view-service-provider-activity.md).
- Learn how service providers can [view and manage customers](view-manage-customers.md) in the Azure portal.
- Learn how [enterprises managing multiple tenants](../concepts/enterprise.md) can use Azure Lighthouse to consolidate their management experience.
