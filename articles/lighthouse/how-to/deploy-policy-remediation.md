---
title: Deploy a policy that can be remediated within a delegated subscription
description: To deploy policies that use a remediation task via Azure Lighthouse, you need to create a managed identity in the customer tenant.
ms.date: 01/20/2026
ms.topic: how-to
# Customer intent: "As a service provider, I want to deploy remediation policies in delegated customer subscriptions, so that I can ensure compliant resource configurations across managed tenants."
---

# Deploy a policy that can be remediated within a delegated subscription

[Azure Lighthouse](../overview.md) enables service providers to create and edit policy definitions within a delegated subscription. To deploy policies that use a [remediation task](/azure/governance/policy/how-to/remediate-resources) (that is, policies with the [deployIfNotExists](/azure/governance/policy/concepts/effect-deploy-if-not-exists) or [modify](/azure/governance/policy/concepts/effect-modify) effect), you must create a [managed identity](/azure/active-directory/managed-identities-azure-resources/overview) in the customer tenant. Azure Policy can use this managed identity to deploy the template within the policy.

This article describes the steps to enable this scenario, both when you onboard the customer to Azure Lighthouse and when you deploy the policy.

> [!TIP]
> Though this article refers to service providers and customers, [enterprises managing multiple tenants](../concepts/enterprise.md) can use the same processes.

## Create a user who can assign roles to a managed identity in the customer tenant

When you [onboard a customer to Azure Lighthouse](onboard-customer.md), you define authorizations that grant access to delegated resources in the customer tenant. Each authorization specifies a **principalId** that corresponds to a Microsoft Entra user, group, or service principal in the managing tenant, and a **roleDefinitionId** that corresponds to the [Azure built-in role](/azure/role-based-access-control/built-in-roles) that you grant.

To allow a **principalId** to assign roles to a managed identity in the customer tenant, set its **roleDefinitionId** to **User Access Administrator**. While this role isn't generally supported for Azure Lighthouse, it can be used in this specific scenario. Grant this role to the **principalId** so it can assign specific built-in roles to managed identities. These roles are defined in the **delegatedRoleDefinitionIds** property. You can include any [supported Azure built-in role](../concepts/tenants-users-roles.md#role-support-for-azure-lighthouse) except for User Access Administrator or Owner.

After the customer is onboarded, the **principalId** created in this authorization can assign these built-in roles to managed identities in the customer tenant. It doesn't have any other permissions normally associated with the User Access Administrator role.

> [!NOTE]
> You must currently use APIs, not the Azure portal, to create [role assignments](/azure/role-based-access-control/role-assignments-steps#step-5-assign-role) across tenants.

This example shows a **principalId** with the User Access Administrator role. This user can assign two built-in roles to managed identities in the customer tenant: Contributor and Log Analytics Contributor.

```json
{
    "principalId": "00000000-0000-0000-0000-000000000000",
    "principalIdDisplayName": "Policy Automation Account",
    "roleDefinitionId": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "delegatedRoleDefinitionIds": [
         "b24988ac-6180-42a0-ab88-20f7382dd24c",
         "92aaf0da-9dab-42b6-94a3-d43ce8d16293"
    ]
}
```

## Deploy policies that can be remediated

After you create the user with the necessary permissions, that user can deploy policies that use remediation tasks within delegated customer subscriptions.

For example, suppose you want to enable diagnostics on Azure Key Vault resources in the customer tenant, as shown in this [sample](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-enforce-keyvault-monitoring). A user in the managing tenant with the appropriate permissions (as described earlier) deploys an [Azure Resource Manager template](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/policy-enforce-keyvault-monitoring/enforceAzureMonitoredKeyVault.json) to enable this scenario.

You currently must use APIs to create the policy assignment to use with a delegated subscription; you can't use the Azure portal. When you create the policy assignment, set the **apiVersion** to **2019-04-01-preview** or later to include the **delegatedManagedIdentityResourceId** property. This property allows you to include a managed identity that resides in the customer tenant (in a subscription or resource group that you onboarded to Azure Lighthouse).

The following example shows a role assignment with a **delegatedManagedIdentityResourceId**.

```json
"type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2019-04-01-preview",
            "name": "[parameters('rbacGuid')]",
            "dependsOn": [
                "[variables('policyAssignment')]"
            ],
            "properties": {
                "roleDefinitionId": "[concat(subscription().id, '/providers/Microsoft.Authorization/roleDefinitions/', variables('rbacContributor'))]",
                "principalType": "ServicePrincipal",
                "delegatedManagedIdentityResourceId": "[concat(subscription().id, '/providers/Microsoft.Authorization/policyAssignments/', variables('policyAssignment'))]",
                "principalId": "[toLower(reference(concat('/providers/Microsoft.Authorization/policyAssignments/', variables('policyAssignment')), '2018-05-01', 'Full' ).identity.principalId)]"
            }
```

> [!TIP]
> A [similar sample](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-add-or-replace-tag) is available to demonstrate how to deploy a policy that adds or removes a tag (using the modify effect) to a delegated subscription.

## Next steps

- Learn about [Azure Policy](/azure/governance/policy/).
- Learn about [managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview).
