---
title: Create eligible authorizations
description: When onboarding customers to Azure Lighthouse, you can let users in your managing tenant elevate their role on a just-in-time basis. 
ms.date: 04/06/2026
ms.topic: how-to
ms.custom: devx-track-arm-template
# Customer intent: "As a cloud administrator onboarding customers to Azure Lighthouse, I want to create eligible authorizations with just-in-time role elevation, so that I can control privileged access while minimizing security risks associated with permanent role assignments."
---

# Create eligible authorizations

When you onboard customers to Azure Lighthouse, you create authorizations to grant specified Azure built-in roles to users in your managing tenant. You can also create eligible authorizations that use [Microsoft Entra Privileged Identity Management (PIM)](/entra/id-governance/privileged-identity-management/pim-configure) to let users in your managing tenant temporarily elevate their role. This role elevation grants additional permissions on a just-in-time basis so that users only have those permissions for a set duration.

By creating eligible authorizations, you minimize the number of permanent assignments of users to privileged roles. This approach helps reduce security risks related to privileged access by users in your tenant.

This article explains how eligible authorizations work and how to create them when [onboarding a customer to Azure Lighthouse](onboard-customer.md).

## License requirements

Because eligible authorizations use Microsoft Entra Privileged Identity Management, the managing tenant must have a [valid Microsoft Entra ID Governance license that supports Privileged Identity management](/entra/id-governance/licensing-fundamentals#privileged-identity-management) in order to create eligible authorizations.

Any extra costs associated with an eligible role apply only during the period of time in which the user elevates their access to that role.

> [!NOTE]
> Creating eligible authorizations isn't supported in [national clouds](/entra/identity-platform/authentication-national-cloud).

## How eligible authorizations work

An eligible authorization defines a role assignment that requires the user to activate the role when they need to perform privileged tasks. When they activate the eligible role, they have the full access granted by that role for the specified period of time.

Users in the customer tenant can review all role assignments, including those in eligible authorizations, before the onboarding process.

When a user activates an eligible role, they get that elevated role on the delegated scope for a preconfigured time period, in addition to their permanent role assignments for that scope.

Administrators in the managing tenant can review all Privileged Identity Management activities by viewing the audit log in the managing tenant. Customers can view these actions in the Azure activity log for the delegated subscription.

## Eligible authorization elements

You can create an eligible authorization when onboarding customers by using Azure Resource Manager templates or by publishing a Managed Services offer to Microsoft Marketplace. Each eligible authorization must include three elements: the user, the role, and the access policy.

### User

For each eligible authorization, provide the principal ID for either an individual user or a Microsoft Entra group in the managing tenant. Along with the principal ID, provide a display name for each authorization.

If you assign an eligible authorization to a group, any member of that group can elevate their own individual access to that role, per the access policy.

You can't use eligible authorizations with service principals, since there's currently no way for a service principal account to elevate its access and use an eligible role. You also can't use eligible authorizations with `delegatedRoleDefinitionIds` that a User Access Administrator can [assign to managed identities](deploy-policy-remediation.md).

> [!NOTE]
> For each eligible authorization, be sure to also create a permanent (active) authorization for the same principal ID with a different role, such as Reader (or another Azure built-in role that includes Reader access). If you don't include a permanent authorization with Reader access, the user can't elevate their role in the Azure portal.

### Role

Each eligible authorization needs to include an [Azure built-in role](/azure/role-based-access-control/built-in-roles) that the user can use on a just-in-time basis.

The role can be any Azure built-in role that is [supported for Azure delegated resource management](../concepts/tenants-users-roles.md#role-support-for-azure-lighthouse), except for User Access Administrator.

> [!IMPORTANT]
> If you include multiple eligible authorizations that use the same role, each of the eligible authorizations must have the same access policy settings.

### Access policy

The access policy defines the multifactor authentication requirements, the length of time a user will be activated in the role before it expires, and whether approvers are required.

#### Multifactor authentication

Specify whether to require [Microsoft Entra multifactor authentication](/entra/identity/authentication/concept-mfa-howitworks) to activate an eligible role.

#### Maximum duration

Define the total length of time for which the user will have the eligible role. The minimum value is 30 minutes and the maximum is 8 hours.

#### Approvers

The approvers element is optional. If you include it, you can specify up to 10 users or user groups in the managing tenant who can approve or deny requests from a user to activate the eligible role.

You can't use a service principal account as an approver. Also, approvers can't approve their own access. If an approver is also included as the user in an eligible authorization, a different approver must grant access for them to elevate their role.

If you don't include any approvers, the user can activate the eligible role whenever they choose.

## Create eligible authorizations by using Managed Services offers

You can onboard your customer to Azure Lighthouse by publishing Managed Services offers to Microsoft Marketplace. When [creating your offers in Partner Center](publish-managed-services-offers.md), you specify whether the **Access type** for each [**Authorization**](/azure/marketplace/create-managed-service-offer-plans#authorizations) should be **Active** or **Eligible**.

When you select **Eligible**, the user in your authorization can activate the role according to the access policy you configure. You must set a maximum duration between 30 minutes and 8 hours, and specify whether you require multifactor authentication. You can also add up to 10 approvers if you choose to use them, providing a display name and a principal ID for each one.

Be sure to understand the [eligible authorization elements](#eligible-authorization-elements) when configuring your eligible authorizations in Partner Center.

## Create eligible authorizations by using Azure Resource Manager templates

You can onboard customers to Azure Lighthouse by using an [Azure Resource Manager template along with a corresponding parameters file](onboard-customer.md#create-an-azure-resource-manager-template) that you modify. The template you choose depends on whether you're onboarding an entire subscription, a resource group, or multiple resource groups within a subscription.

To include eligible authorizations when you onboard a customer, use one of the templates from the [delegated-resource-management-eligible-authorizations section of our samples repo](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-eligible-authorizations). The repository provides templates with and without approvers included, so that you can use the one that works best for your scenario.

|To onboard this (with eligible authorizations)  |Use this Azure Resource Manager template  |And modify this parameter file |
|---------|---------|---------|
|Subscription   |[subscription.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription.json)  |[subscription.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription.parameters.json)    |
|Subscription (with approvers)  |[subscription-managing-tenant-approvers.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription-managing-tenant-approvers.json)  |[subscription-managing-tenant-approvers.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription-managing-tenant-approvers.parameters.json)    |
|Resource group   |[rg.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/rg.json)  |[rg.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/rg.parameters.json)    |
|Resource group (with approvers)  |[rg-managing-tenant-approvers.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/rg-managing-tenant-approvers.json)  |[rg-managing-tenant-approvers.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/rg-managing-tenant-approvers.parameters.json)    |
|Multiple resource groups within a subscription   |[multiple-rg.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/multiple-rg.json)  |[multiple-rg.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/multiple-rg.parameters.json)    |
|Multiple resource groups within a subscription (with approvers)  |[multiple-rg-managing-tenant-approvers.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/multiple-rg-managing-tenant-approvers.json)  |[multiple-rg-managing-tenant-approvers.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/rg/multiple-rg-managing-tenant-approvers.parameters.json)    |

For example, this is the **subscription-managing-tenant-approvers.json** template, which onboards a subscription with eligible authorizations (including approvers).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "type": "string",
            "metadata": {
                "description": "Specify a unique name for your offer"
            }
        },
        "mspOfferDescription": {
            "type": "string",
            "metadata": {
                "description": "Name of the Managed Service Provider offering"
            }
        },
        "managedByTenantId": {
            "type": "string",
            "metadata": {
                "description": "Specify the tenant id of the Managed Service Provider"
            }
        },
        "authorizations": {
            "type": "array",
            "metadata": {
                "description": "Specify an array of objects, containing tuples of Azure Active Directory principalId, a Azure roleDefinitionId, and an optional principalIdDisplayName. The roleDefinition specified is granted to the principalId in the provider's Active Directory and the principalIdDisplayName is visible to customers."
            }
        },
        "eligibleAuthorizations": {
            "type": "array",
            "metadata": {
                "description": "Provide the authorizations that will have just-in-time role assignments on customer environments with support for approvals from the managing tenant"
            }
        }
    },
        "variables": {
            "mspRegistrationName": "[guid(parameters('mspOfferName'))]",
            "mspAssignmentName": "[guid(parameters('mspOfferName'))]"
        },
        "resources": [
            {
                "type": "Microsoft.ManagedServices/registrationDefinitions",
                "apiVersion": "2020-02-01-preview",
                "name": "[variables('mspRegistrationName')]",
                "properties": {
                    "registrationDefinitionName": "[parameters('mspOfferName')]",
                    "description": "[parameters('mspOfferDescription')]",
                    "managedByTenantId": "[parameters('managedByTenantId')]",
                    "authorizations": "[parameters('authorizations')]",
                    "eligibleAuthorizations": "[parameters('eligibleAuthorizations')]"
                }
            },
            {
                "type": "Microsoft.ManagedServices/registrationAssignments",
                "apiVersion": "2020-02-01-preview",
                "name": "[variables('mspAssignmentName')]",
                "dependsOn": [
                    "[resourceId('Microsoft.ManagedServices/registrationDefinitions/', variables('mspRegistrationName'))]"
                ],
                "properties": {
                    "registrationDefinitionId": "[resourceId('Microsoft.ManagedServices/registrationDefinitions/', variables('mspRegistrationName'))]"
                }
            }
        ],
        "outputs": {
            "mspOfferName": {
                "type": "string",
                "value": "[concat('Managed by', ' ', parameters('mspOfferName'))]"
            },
            "authorizations": {
                "type": "array",
                "value": "[parameters('authorizations')]"
            },
            "eligibleAuthorizations": {
                "type": "array",
                "value": "[parameters('eligibleAuthorizations')]"
            }
        }
    }
```

### Define eligible authorizations in your parameters file

The parameters file that corresponds with your deployment template defines authorizations, including eligible authorizations.

Define each of your eligible authorizations in the `eligibleAuthorizations` parameter. For example, this [subscription-managing-tenant-approvers.parameters.json sample template](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management-eligible-authorizations/subscription/subscription-managing-tenant-approvers.parameters.json)
includes one eligible authorization. It also includes the `managedbyTenantApprovers` element, which adds a `principalId` who must approve all attempts to activate the eligible roles that are defined in the `eligibleAuthorizations` element.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "value": "Relecloud Managed Services"
        },
        "mspOfferDescription": {
            "value": "Relecloud Managed Services"
        },
        "managedByTenantId": {
            "value": "<insert the managing tenant id>"
        },
        "authorizations": {
            "value": [
                { 
                    "principalId": "00000000-0000-0000-0000-000000000000",
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7",
                    "principalIdDisplayName": "PIM group"
                }
            ]
        }, 
        "eligibleAuthorizations":{
            "value": [
                {
                        "justInTimeAccessPolicy": {
                            "multiFactorAuthProvider": "Azure",
                            "maximumActivationDuration": "PT8H",
                            "managedByTenantApprovers": [ 
                                { 
                                    "principalId": "00000000-0000-0000-0000-000000000000", 
                                    "principalIdDisplayName": "PIM-Approvers" 
                                } 
                            ]
                        },
                        "principalId": "00000000-0000-0000-0000-000000000000", 
                        "principalIdDisplayName": "Tier 2 Support",
                        "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"

                }
            ]
        }
    }
}
```

Each entry within the `eligibleAuthorizations` parameter contains [three elements](#eligible-authorization-elements) that define an eligible authorization: `principalId`, `roleDefinitionId`, and `justInTimeAccessPolicy`.

`principalId` specifies the ID for the Microsoft Entra user or group to which this eligible authorization applies.

`roleDefinitionId` contains the role definition ID for an [Azure built-in role](/azure/role-based-access-control/built-in-roles) that the user will be eligible to use on a just-in-time basis. If you include multiple eligible authorizations that use the same `roleDefinitionId`, they all must have identical settings for `justInTimeAccessPolicy`.

`justInTimeAccessPolicy` specifies three elements:

- `multiFactorAuthProvider` can either be set to **Azure**, which requires authentication by using Microsoft Entra multifactor authentication, or to **None** if no multifactor authentication is required.
- `maximumActivationDuration` sets the total length of time for which the user will have the eligible role. This value must use the ISO 8601 duration format. The minimum value is PT30M (30 minutes) and the maximum value is PT8H (8 hours). For simplicity, use values in half-hour increments, such as PT6H for 6 hours or PT6H30M for 6.5 hours.
- `managedByTenantApprovers` is optional. If you include it, it must contain one or more combinations of a principalId and a principalIdDisplayName who must approve any activation of the eligible role.

For more information about these elements, see the [Eligible authorization elements](#eligible-authorization-elements) section.

## Elevation process for users

After you onboard a customer to Azure Lighthouse, the specified user (or users in any specified groups) can access the eligible roles you included.

Each user can elevate their access at any time by visiting the **My customers** page in the Azure portal, selecting a delegation, and then selecting **Manage eligible roles**. After that, they can follow the [steps to activate the role](/entra/id-governance/privileged-identity-management/pim-resource-roles-activate-your-roles) in Microsoft Entra Privileged Identity Management.

If you specify approvers, the user can't access the role until an [approver from the managing tenant](#approvers) grants approval. All of the approvers are notified when approval is requested, and the user can't use the eligible role until approval is granted. Approvers also get notified about each approval.

For more information about the approval process, see [Approve or deny requests for Azure resource roles in Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-resource-roles-approval-workflow).

Once the eligible role has been activated, the user will have that role for the full duration specified in the eligible authorization. After that time period, they will no longer be able to use that role, unless they repeat the elevation process and elevate their access again.

## Next steps

- Learn how to [onboard customers to Azure Lighthouse using ARM templates](onboard-customer.md).
- Learn how to [onboard customers using Managed Services offers](publish-managed-services-offers.md).
- Learn more about [Microsoft Entra Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-configure).
- Learn more about [tenants, users, and roles in Azure Lighthouse](../concepts/tenants-users-roles.md).
