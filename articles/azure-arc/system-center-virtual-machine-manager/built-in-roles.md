---
title: Azure built-in roles for Azure Arc-enabled SCVMM
description: This article lists the Azure built-in roles for Azure Arc-enabled SCVMM. It lists Actions, NotActions, DataActions, and NotDataActions.
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.topic: generated-reference
author: jyothisuri
manager: akashdubey
ms.author: jsuri
ms.date: 05/08/2025
ms.custom:
  - generated
  - build-2025
# Customer intent: As a cloud administrator, I want to understand the built-in role definitions for Azure Arc-enabled SCVMM, so that I can manage permissions effectively and ensure secure operations across virtual machine instances.
---

# Azure built-in roles for Azure Arc-enabled SCVMM

This article lists the Azure built-in roles and their permissions for Azure Arc-enabled SCVMM. 
Azure Arc-enabled SCVMM has four built-in roles:

- [Azure Arc SCVMM Administrator role](#azure-arc-scvmm-administrator-role) 
- [Azure Arc SCVMM Private Cloud User](#azure-arc-scvmm-private-cloud-user)
- [Azure Arc SCVMM Private Clouds Onboarding](#azure-arc-scvmm-private-clouds-onboarding)
- [Azure Arc SCVMM VM Contributor](#azure-arc-scvmm-vm-contributor)

If the built-in Azure roles doesn’t match your requirements, you can [create custom roles](/azure/azure-arc/system-center-virtual-machine-manager/create-custom-roles) with granular permissions. 

## Azure Arc SCVMM Administrator role

Arc SCVMM VM Administrator has permission to perform all SCVMM actions.


> [!div class="mx-tableFixed"]
> | Actions | Description |
> | --- | --- |
> |Microsoft.Authorization/classicAdministrators/read|Reads the administrators for the subscription.|
> |Microsoft.Authorization/classicAdministrators/operationstatuses/read|Gets the administrator opreation statuses of the subscription.|
> |Microsoft.Authorization/denyAssignments/read|Get information about a deny assignment.|
> |Microsoft.Authorization/diagnosticSettingsCategories/read|Get the information about diagnostic settings categories.|
> |Microsoft.Authorization/diagnosticSettings/read|Read the information about diagnostics settings.|
> |Microsoft.Authorization/roleEligibilityScheduleInstances/read|Gets the role eligibility schedule instances at given scope.|
> |Microsoft.Authorization/locks/read|Gets locks at the specified scope.|
> |Microsoft.Authorization/operations/read|Gets the list of operations.|
> |Microsoft.Authorization/permissions/read|Lists all the permissions the caller has at a given scope.|
> |Microsoft.Authorization/policyAssignments/read|Get information about a policy assignment.|
> |Microsoft.Authorization/policyAssignments/ privateLinkAssociations/read|Get information about private link association.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/read|Get information about resource management private link.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnections/read|Get information about private endpoint connection.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnectionProxies/read|Get information about private endpoint connection proxy.|
> |Microsoft.Authorization/policyDefinitions/read|Get information about a policy definition.|
> |Microsoft.Authorization/policyDefinitions/versions/read|Get information about a policy definition version.|
> |Microsoft.Authorization/policyEnrollments/read|Get information about a policy enrollment.|
> |Microsoft.Authorization/policyExemptions/read|Get information about a policy exemption.|
> |Microsoft.Authorization/policySetDefinitions/read|Get information about a policy set definition.|
> |Microsoft.Authorization/policySetDefinitions/versions/read|Get information about a policy set definition version.|
> |Microsoft.Authorization/providerOperations/read|Get operations for all resource providers which can be used in role definitions.|
> |Microsoft.Authorization/roleAssignments/read|Get information about a role assignment.|
> |Microsoft.Authorization/roleAssignmentSchedules/read|Gets the role assignment schedules at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleInstances/read|Gets the role assignment schedule instances at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleRequests/read|Gets the role assignment schedule requests at given scope.|
> |Microsoft.Authorization/roleDefinitions/read|Get information about a role definition.|
> |Microsoft.Authorization/roleEligibilitySchedules/read|Gets the role eligibility schedules at given scope.|
> |Microsoft.Authorization/roleEligibilityScheduleRequests/read|Gets the role eligibility schedule requests at given scope.|
> |Microsoft.Authorization/roleManagementPolicies/read|Get Role management policies|
> |Microsoft.Authorization/roleManagementPolicyAssignments/read|Get role management policy assignments
> |Microsoft.Insights/AlertRules/Write|Create or update a classic metric alert.|
> |Microsoft.Insights/AlertRules/Delete|Delete a classic metric alert.|
> |Microsoft.Insights/AlertRules/Read|Read a classic metric alert.|
> |Microsoft.Insights/AlertRules/Activated/Action|Classic metric alert activated.|
> |Microsoft.Insights/AlertRules/Resolved/Action|Classic metric alert resolved.|
> |Microsoft.Insights/AlertRules/Throttled/Action|Classic metric alert rule throttled.|
> |Microsoft.Insights/AlertRules/Incidents/Read|Read a classic metric alert incident.|
> |Microsoft.Resources/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/deployments/delete|Deletes a deployment.|
> |Microsoft.Resources/deployments/cancel/action|Cancels a deployment.|
> |Microsoft.Resources/deployments/validate/action|Validates an deployment.|
> |Microsoft.Resources/deployments/whatIf/action|Predicts template deployment changes.|
> |Microsoft.Resources/deployments/exportTemplate/action|Export template for a deployment.|
> |Microsoft.Resources/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/read|Gets the list of subscriptions.|
> |Microsoft.Resources/subscriptions/resourceGroups/read|Gets or lists resource groups.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/operationresults/read|Get the subscription operation results.|
> |Microsoft.ResourceHealth/AvailabilityStatuses/read|Gets the availability statuses for all resources in the specified scope.|
> |Microsoft.HybridCompute/operations/read|Read all Operations for Azure Arc for Servers.|
> |Microsoft.HybridCompute/osType/agentVersions/read|Read all Azure Connected Machine Agent versions available.|
> |Microsoft.HybridCompute/osType/agentVersions/latest/read|Read the latest Azure Connected Machine Agent version.|
> |Microsoft.HybridCompute/licenses/read|Reads any Azure Arc licenses.|
> |Microsoft.HybridCompute/licenses/write|Installs or Updates an Azure Arc licenses.|
> |Microsoft.HybridCompute/licenses/delete|Deletes an Azure Arc licenses.|
> |Microsoft.HybridCompute/locations/operationresults/read|Reads the status of an operation on Microsoft.HybridCompute Resource Provider.|
> |Microsoft.HybridCompute/locations/operationstatus/read|Reads the status of an operation on Microsoft.HybridCompute Resource Provider.|
> |Microsoft.HybridCompute/locations/updateCenterOperationResults/read|Reads the status of an update center operation on machines.|
> |Microsoft.HybridCompute/machines/read|Read any Azure Arc machines.|
> |Microsoft.HybridCompute/machines/write|Writes an Azure Arc machines.|
> |Microsoft.HybridCompute/machines/delete|Deletes an Azure Arc machines.|
> |Microsoft.HybridCompute/machines/UpgradeExtensions/action|Upgrades Extensions on Azure Arc machines.|
> |Microsoft.HybridCompute/machines/assessPatches/action|Assesses any Azure Arc machines to get missing software patches.|
> |Microsoft.HybridCompute/machines/installPatches/action|Installs patches on any Azure Arc machines.|
> |Microsoft.HybridCompute/machines/patchInstallationResults/read|Reads any Azure Arc patchInstallationResults.|
> |Microsoft.HybridCompute/machines/patchInstallationResults/softwarePatches/read|Reads any Azure Arc patchInstallationResults/softwarePatches.|
> |Microsoft.HybridCompute/machines/extensions/read|Reads any Azure Arc extensions|
> |Microsoft.HybridCompute/machines/extensions/write|Installs or Updates an Azure Arc extensions|
> |Microsoft.HybridCompute/machines/extensions/delete|Deletes an Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/licenseProfiles/read|Reads any Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/licenseProfiles/write|Installs or Updates an Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/licenseProfiles/delete|Deletes an Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/hybridIdentityMetadata/read|Read any Azure Arc machines's Hybrid Identity Metadata|
> |Microsoft.HybridCompute/machines/patchAssessmentResults/read|Reads any Azure Arc patchAssessmentResults.|
> |Microsoft.HybridCompute/machines/patchAssessmentResults/softwarePatches/read|Reads any Azure Arc patchAssessmentResults/softwarePatches.|
> |Microsoft.HybridCompute/machines/runcommands/read|Reads any Azure Arc runcommands.|
> |Microsoft.HybridCompute/machines/runcommands/write|Installs or Updates an Azure Arc runcommands.|
> |Microsoft.HybridCompute/machines/runcommands/delete|Deletes an Azure Arc runcommands.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |Microsoft.SCVMM/unregister/action|unregister RP.|
> |Microsoft.SCVMM/register/action|register RP.|
> |Microsoft.SCVMM/availabilitySets/Read|Read availabilitySets.|
> |Microsoft.SCVMM/availabilitySets/Write|Writes availabilitySets.|
> |Microsoft.SCVMM/availabilitySets/Delete|Deletes availabilitySets.|
> |Microsoft.SCVMM/clouds/Read|Read clouds.|
> |Microsoft.SCVMM/clouds/Write|Writes clouds.|
> |Microsoft.SCVMM/clouds/Delete|Deletes clouds.|
> |Microsoft.SCVMM/clouds/deploy/action|Deploy on resource pool.|
> |Microsoft.SCVMM/locations/operationstatuses/read|Read operationstatus.|
> |Microsoft.SCVMM/locations/operationstatuses/write|Write operationstatus.|
> |Microsoft.SCVMM/operations/read|Read operations.|
> |Microsoft.SCVMM/skus/read|Get skus.|
> |Microsoft.SCVMM/virtualMachineInstances/read|Retrieves information about a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/write|The operation to create or update a virtual machine instance. Please note some properties can be set only during virtual machine instance creation.|
> |Microsoft.SCVMM/virtualMachineInstances/delete|The operation to delete a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/stop/action|The operation to power off (stop) a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/start/action|The operation to start a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/restart/action|The operation to restart a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/createCheckpoint/action|Creates a checkpoint in virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/deleteCheckpoint/action|Deletes a checkpoint in virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/restoreCheckpoint/action|Restores to a checkpoint in virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/guestAgents/read|Implements GuestAgent GET method.|
> |Microsoft.SCVMM/virtualMachineInstances/guestAgents/write|Create Or Update GuestAgent.|
> |Microsoft.SCVMM/virtualMachineInstances/guestAgents/delete|Implements GuestAgent DELETE method.|
> |Microsoft.SCVMM/virtualMachineInstances/hybridIdentityMetadata/read|Implements HybridIdentityMetadata GET method.|
> |Microsoft.SCVMM/virtualmachines/Delete|Deletes virtualmachines.|
> |Microsoft.SCVMM/virtualmachinetemplates/Read|Read virtualmachinetemplates.|
> |Microsoft.SCVMM/virtualmachinetemplates/Write|Writes virtualmachinetemplates.|
> |Microsoft.SCVMM/virtualmachinetemplates/Delete|Deletes virtualmachinetemplates.|
> |Microsoft.SCVMM/virtualmachinetemplates/clone/action|Clones virtualmachinetemplates.|
> |Microsoft.SCVMM/virtualnetworks/Read|Read virtualnetworks.|
> |Microsoft.SCVMM/virtualnetworks/Write|Writes virtualnetworks.|
> |Microsoft.SCVMM/virtualnetworks/Delete|Deletes virtualnetworks.|
> |Microsoft.SCVMM/virtualnetworks/join/action|Join virtual network.|
> |Microsoft.SCVMM/vmmservers/Read|Read vmmservers.|
> |Microsoft.SCVMM/vmmservers/Write|Writes vmmservers.|
> |Microsoft.SCVMM/vmmservers/Delete|Deletes vmmservers.|
> |Microsoft.SCVMM/vmmservers/inventoryitems/Delete|Deletes vmmserver inventoryitems.|
> |Microsoft.SCVMM/vmmservers/inventoryitems/Read|Read vmmserver inventoryitems.|
> |Microsoft.SCVMM/vmmservers/inventoryitems/Write|Writes vmmservers inventoryitems.|
> |Microsoft.SCVMM/vmmservers/inventoryitems/onboard/action|Onboards vmmservers inventoryitems.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |

```json
{
    "id": "/providers/Microsoft.Authorization/roleDefinitions/a92dfd61-77f9-4aec-a531-19858b406c87",
    "properties": {
        "roleName": "Azure Arc ScVmm Administrator role",
        "description": "Arc ScVmm VM Administrator has permissions to perform all ScVmm actions.",
        "assignableScopes": [
            "/"
        ],
        "permissions": [
            {
                "actions": [
                    "Microsoft.ScVmm/*",
                    "Microsoft.Insights/AlertRules/Write",
                    "Microsoft.Insights/AlertRules/Delete",
                    "Microsoft.Insights/AlertRules/Read",
                    "Microsoft.Insights/AlertRules/Activated/Action",
                    "Microsoft.Insights/AlertRules/Resolved/Action",
                    "Microsoft.Insights/AlertRules/Throttled/Action",
                    "Microsoft.Insights/AlertRules/Incidents/Read",
                    "Microsoft.Resources/deployments/read",
                    "Microsoft.Resources/deployments/write",
                    "Microsoft.Resources/deployments/delete",
                    "Microsoft.Resources/deployments/cancel/action",
                    "Microsoft.Resources/deployments/validate/action",
                    "Microsoft.Resources/deployments/whatIf/action",
                    "Microsoft.Resources/deployments/exportTemplate/action",
                    "Microsoft.Resources/deployments/operations/read",
                    "Microsoft.Resources/deployments/operationstatuses/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/write",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read",
                    "Microsoft.ResourceHealth/availabilityStatuses/read",
                    "Microsoft.Authorization/*/read",
                    "Microsoft.Resources/subscriptions/read",
                    "Microsoft.Resources/subscriptions/resourceGroups/read",
                    "Microsoft.Resources/subscriptions/operationresults/read",
                    "Microsoft.ExtendedLocation/customLocations/Read",
                    "Microsoft.ExtendedLocation/customLocations/deploy/action",
                    "Microsoft.HybridCompute/machines/read",
                    "Microsoft.HybridCompute/machines/write",
                    "Microsoft.HybridCompute/machines/delete",
                    "Microsoft.HybridCompute/machines/UpgradeExtensions/action",
                    "Microsoft.HybridCompute/machines/assessPatches/action",
                    "Microsoft.HybridCompute/machines/installPatches/action",
                    "Microsoft.HybridCompute/machines/extensions/read",
                    "Microsoft.HybridCompute/machines/extensions/write",
                    "Microsoft.HybridCompute/machines/extensions/delete",
                    "Microsoft.HybridCompute/operations/read",
                    "Microsoft.HybridCompute/locations/operationresults/read",
                    "Microsoft.HybridCompute/locations/operationstatus/read",
                    "Microsoft.HybridCompute/machines/patchAssessmentResults/read",
                    "Microsoft.HybridCompute/machines/patchAssessmentResults/softwarePatches/read",
                    "Microsoft.HybridCompute/machines/patchInstallationResults/read",
                    "Microsoft.HybridCompute/machines/patchInstallationResults/softwarePatches/read",
                    "Microsoft.HybridCompute/locations/updateCenterOperationResults/read",
                    "Microsoft.HybridCompute/machines/hybridIdentityMetadata/read",
                    "Microsoft.HybridCompute/osType/agentVersions/read",
                    "Microsoft.HybridCompute/osType/agentVersions/latest/read",
                    "Microsoft.HybridCompute/machines/runcommands/read",
                    "Microsoft.HybridCompute/machines/runcommands/write",
                    "Microsoft.HybridCompute/machines/runcommands/delete",
                    "Microsoft.HybridCompute/machines/licenseProfiles/read",
                    "Microsoft.HybridCompute/machines/licenseProfiles/write",
                    "Microsoft.HybridCompute/machines/licenseProfiles/delete",
                    "Microsoft.HybridCompute/licenses/read",
                    "Microsoft.HybridCompute/licenses/write",
                    "Microsoft.HybridCompute/licenses/delete"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

## Azure Arc SCVMM Private Cloud User

Azure Arc SCVMM Private Cloud User has permissions to use the SCVMM resources to deploy VMs.


> [!div class="mx-tableFixed"]
> | Actions | Description |
> | --- | --- |
> |Microsoft.Authorization/classicAdministrators/read|Reads the administrators for the subscription.|
> |Microsoft.Authorization/classicAdministrators/operationstatuses/read|Gets the administrator operation statuses of the subscription.|
> |Microsoft.Authorization/denyAssignments/read|Get information about a deny assignment.|
> |Microsoft.Authorization/diagnosticSettingsCategories/read|Get the information about diagnostic settings categories.|
> |Microsoft.Authorization/diagnosticSettings/read|Read the information about diagnostics settings.|
> |Microsoft.Authorization/roleEligibilityScheduleInstances/read|Gets the role eligibility schedule instances at given scope.|
> |Microsoft.Authorization/locks/read|Gets locks at the specified scope.|
> |Microsoft.Authorization/operations/read|Gets the list of operations.|
> |Microsoft.Authorization/permissions/read|Lists all the permissions the caller has at a given scope.|
> |Microsoft.Authorization/policyAssignments/read|Get information about a policy assignment.|
> |Microsoft.Authorization/policyAssignments/privateLinkAssociations/read|Get information about private link association.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/read|Get information about resource management private link.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnections/read|Get information about private endpoint connection.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnectionProxies/read|Get information about private endpoint connection proxy.|
> |Microsoft.Authorization/policyDefinitions/read|Get information about a policy definition.|
> |Microsoft.Authorization/policyDefinitions/versions/read|Get information about a policy definition version.|
> |Microsoft.Authorization/policyEnrollments/read|Get information about a policy enrollment.|
> |Microsoft.Authorization/policyExemptions/read|Get information about a policy exemption.|
> |Microsoft.Authorization/policySetDefinitions/read|Get information about a policy set definition.|
> |Microsoft.Authorization/policySetDefinitions/versions/read|Get information about a policy set definition version.|
> |Microsoft.Authorization/providerOperations/read|Get operations for all resource providers which can be used in role definitions.|
> |Microsoft.Authorization/roleAssignments/read|Get information about a role assignment.|
> |Microsoft.Authorization/roleAssignmentSchedules/read|Gets the role assignment schedules at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleInstances/read|Gets the role assignment schedule instances at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleRequests/read|Gets the role assignment schedule requests at given scope.|
> |Microsoft.Authorization/roleDefinitions/read|Get information about a role definition.|
> |Microsoft.Authorization/roleEligibilitySchedules/read|Gets the role eligibility schedules at given scope.|
> |Microsoft.Authorization/roleEligibilityScheduleRequests/read|Gets the role eligibility schedule requests at given scope.|
> |Microsoft.Authorization/roleManagementPolicies/read|Get Role management policies.|
> |Microsoft.Authorization/roleManagementPolicyAssignments/read|Get role management policy assignments.|
> |Microsoft.Insights/AlertRules/Write|Create or update a classic metric alert.|
> |Microsoft.Insights/AlertRules/Delete|Delete a classic metric alert.|
> |Microsoft.Insights/AlertRules/Read|Read a classic metric alert.|
> |Microsoft.Insights/AlertRules/Activated/Action|Classic metric alert activated.|
> |Microsoft.Insights/AlertRules/Resolved/Action|Classic metric alert resolved.|
> |Microsoft.Insights/AlertRules/Throttled/Action|Classic metric alert rule throttled.|
> |Microsoft.Insights/AlertRules/Incidents/Read|Read a classic metric alert incident.|
> |Microsoft.Resources/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/deployments/delete|Deletes a deployment.|
> |Microsoft.Resources/deployments/cancel/action|Cancels a deployment.|
> |Microsoft.Resources/deployments/validate/action|Validates an deployment.|
> |Microsoft.Resources/deployments/whatIf/action|Predicts template deployment changes.|
> |Microsoft.Resources/deployments/exportTemplate/action|Export template for a deployment.|
> |Microsoft.Resources/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/read|Gets the list of subscriptions.|
> |Microsoft.Resources/subscriptions/resourceGroups/read|Gets or lists resource groups.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/operationresults/read|Get the subscription operation results.|
> |Microsoft.ResourceHealth/AvailabilityStatuses/read|Gets the availability statuses for all resources in the specified scope.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/enabledresourcetypes/read|Gets EnabledResourceTypes for a Custom Location resource.|
> |Microsoft.SCVMM/clouds/Read|Read clouds.|
> |Microsoft.SCVMM/clouds/deploy/action|Deploy on resource pool.|
> |Microsoft.SCVMM/virtualmachinetemplates/Read|Read virtualmachinetemplates.|
> |Microsoft.SCVMM/virtualmachinetemplates/clone/action|Clones virtualmachinetemplates.|
> |Microsoft.SCVMM/virtualnetworks/Read|Read virtualnetworks.|
> |Microsoft.SCVMM/virtualnetworks/join/action|Join virtual network.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |


```json
{
    "id": "/providers/Microsoft.Authorization/roleDefinitions/c0781e91-8102-4553-8951-97c6d4243cda",
    "properties": {
        "roleName": "Azure Arc ScVmm Private Cloud User",
        "description": "Azure Arc ScVmm Private Cloud User has permissions to use the ScVmm resources to deploy VMs.",
        "assignableScopes": [
            "/"
        ],
        "permissions": [
            {
                "actions": [
                    "Microsoft.Insights/AlertRules/Write",
                    "Microsoft.Insights/AlertRules/Delete",
                    "Microsoft.Insights/AlertRules/Read",
                    "Microsoft.Insights/AlertRules/Activated/Action",
                    "Microsoft.Insights/AlertRules/Resolved/Action",
                    "Microsoft.Insights/AlertRules/Throttled/Action",
                    "Microsoft.Insights/AlertRules/Incidents/Read",
                    "Microsoft.Resources/deployments/read",
                    "Microsoft.Resources/deployments/write",
                    "Microsoft.Resources/deployments/delete",
                    "Microsoft.Resources/deployments/cancel/action",
                    "Microsoft.Resources/deployments/validate/action",
                    "Microsoft.Resources/deployments/whatIf/action",
                    "Microsoft.Resources/deployments/exportTemplate/action",
                    "Microsoft.Resources/deployments/operations/read",
                    "Microsoft.Resources/deployments/operationstatuses/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/write",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read",
                    "Microsoft.ResourceHealth/availabilityStatuses/read",
                    "Microsoft.Authorization/*/read",
                    "Microsoft.Resources/subscriptions/read",
                    "Microsoft.Resources/subscriptions/resourceGroups/read",
                    "Microsoft.Resources/subscriptions/operationresults/read",
                    "microsoft.scvmm/virtualnetworks/join/action",
                    "microsoft.scvmm/virtualnetworks/Read",
                    "microsoft.scvmm/virtualmachinetemplates/clone/action",
                    "microsoft.scvmm/virtualmachinetemplates/Read",
                    "microsoft.scvmm/clouds/deploy/action",
                    "microsoft.scvmm/clouds/Read",
                    "Microsoft.ExtendedLocation/customLocations/Read",
                    "Microsoft.ExtendedLocation/customLocations/deploy/action",
                    "Microsoft.ExtendedLocation/customLocations/enabledresourcetypes/read"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

## Azure Arc SCVMM Private Clouds Onboarding

Azure Arc SCVMM Private Clouds Onboarding role has permissions to provision all the required resources for onboard and  deboard VMM server instances to Azure.


> [!div class="mx-tableFixed"]
> | Actions | Description |
> | --- | --- |
> |Microsoft.Authorization/classicAdministrators/read|Reads the administrators for the subscription.|
> |Microsoft.Authorization/classicAdministrators/operationstatuses/read|Gets the administrator operation statuses of the subscription.|
> |Microsoft.Authorization/denyAssignments/read|Get information about a deny assignment.|
> |Microsoft.Authorization/diagnosticSettingsCategories/read|Get the information about diagnostic settings categories.|
> |Microsoft.Authorization/diagnosticSettings/read|Read the information about diagnostics settings.|
> |Microsoft.Authorization/roleEligibilityScheduleInstances/read|Gets the role eligibility schedule instances at given scope.|
> |Microsoft.Authorization/locks/read|Gets locks at the specified scope.|
> |Microsoft.Authorization/operations/read|Gets the list of operations.|
> |Microsoft.Authorization/permissions/read|Lists all the permissions the caller has at a given scope.|
> |Microsoft.Authorization/policyAssignments/read|Get information about a policy assignment.|
> |Microsoft.Authorization/policyAssignments/privateLinkAssociations/read|Get information about private link association.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/read|Get information about resource management private link.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnections/read|Get information about private endpoint connection.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnectionProxies/read|Get information about private endpoint connection proxy.|
> |Microsoft.Authorization/policyDefinitions/read|Get information about a policy definition.|
> |Microsoft.Authorization/policyDefinitions/versions/read|Get information about a policy definition version.|
> |Microsoft.Authorization/policyEnrollments/read|Get information about a policy enrollment.|
> |Microsoft.Authorization/policyExemptions/read|Get information about a policy exemption.|
> |Microsoft.Authorization/policySetDefinitions/read|Get information about a policy set definition.|
> |Microsoft.Authorization/policySetDefinitions/versions/read|Get information about a policy set definition version.|
> |Microsoft.Authorization/providerOperations/read|Get operations for all resource providers which can be used in role definitions.|
> |Microsoft.Authorization/roleAssignments/read|Get information about a role assignment.|
> |Microsoft.Authorization/roleAssignmentSchedules/read|Gets the role assignment schedules at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleInstances/read|Gets the role assignment schedule instances at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleRequests/read|Gets the role assignment schedule requests at given scope.|
> |Microsoft.Authorization/roleDefinitions/read|Get information about a role definition.|
> |Microsoft.Authorization/roleEligibilitySchedules/read|Gets the role eligibility schedules at given scope.|
> |Microsoft.Authorization/roleEligibilityScheduleRequests/read|Gets the role eligibility schedule requests at given scope.|
> |Microsoft.Authorization/roleManagementPolicies/read|Get Role management policies.|
> |Microsoft.Authorization/roleManagementPolicyAssignments/read|Get role management policy assignments.|
> |Microsoft.Insights/AlertRules/Write|Create or update a classic metric alert.|
> |Microsoft.Insights/AlertRules/Delete|Delete a classic metric alert.|
> |Microsoft.Insights/AlertRules/Read|Read a classic metric alert.|
> |Microsoft.Insights/AlertRules/Activated/Action|Classic metric alert activated.|
> |Microsoft.Insights/AlertRules/Resolved/Action|Classic metric alert resolved.|
> |Microsoft.Insights/AlertRules/Throttled/Action|Classic metric alert rule throttled.|
> |Microsoft.Insights/AlertRules/Incidents/Read|Read a classic metric alert incident.|
> |Microsoft.Resources/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/deployments/delete|Deletes a deployment.|
> |Microsoft.Resources/deployments/cancel/action|Cancels a deployment.|
> |Microsoft.Resources/deployments/validate/action|Validates an deployment.|
> |Microsoft.Resources/deployments/whatIf/action|Predicts template deployment changes.|
> |Microsoft.Resources/deployments/exportTemplate/action|Export template for a deployment.|
> |Microsoft.Resources/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/read|Gets the list of subscriptions.|
> |Microsoft.Resources/subscriptions/resourceGroups/read|Gets or lists resource groups.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/operationresults/read|Get the subscription operation results.|
> |Microsoft.ResourceHealth/AvailabilityStatuses/read|Gets the availability statuses for all resources in the specified scope.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |Microsoft.SCVMM/vmmservers/Read|Read vmmservers.|
> |Microsoft.SCVMM/vmmservers/Write|Writes vmmservers.|
> |Microsoft.SCVMM/vmmservers/Delete|Deletes vmmservers.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |

```json
{
    "id": "/providers/Microsoft.Authorization/roleDefinitions/6aac74c4-6311-40d2-bbdd-7d01e7c6e3a9",
    "properties": {
        "roleName": "Azure Arc ScVmm Private Clouds Onboarding",
        "description": "Azure Arc ScVmm Private Clouds Onboarding role has permissions to provision all the required resources for onboard and deboard vmm server instances to Azure.",
        "assignableScopes": [
            "/"
        ],
        "permissions": [
            {
                "actions": [
                    "microsoft.scvmm/vmmservers/Read",
                    "microsoft.scvmm/vmmservers/Write",
                    "microsoft.scvmm/vmmservers/Delete",
                    "Microsoft.Insights/AlertRules/Write",
                    "Microsoft.Insights/AlertRules/Delete",
                    "Microsoft.Insights/AlertRules/Read",
                    "Microsoft.Insights/AlertRules/Activated/Action",
                    "Microsoft.Insights/AlertRules/Resolved/Action",
                    "Microsoft.Insights/AlertRules/Throttled/Action",
                    "Microsoft.Insights/AlertRules/Incidents/Read",
                    "Microsoft.Resources/deployments/read",
                    "Microsoft.Resources/deployments/write",
                    "Microsoft.Resources/deployments/delete",
                    "Microsoft.Resources/deployments/cancel/action",
                    "Microsoft.Resources/deployments/validate/action",
                    "Microsoft.Resources/deployments/whatIf/action",
                    "Microsoft.Resources/deployments/exportTemplate/action",
                    "Microsoft.Resources/deployments/operations/read",
                    "Microsoft.Resources/deployments/operationstatuses/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/write",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read",
                    "Microsoft.ResourceHealth/availabilityStatuses/read",
                    "Microsoft.Authorization/*/read",
                    "Microsoft.Resources/subscriptions/read",
                    "Microsoft.Resources/subscriptions/resourceGroups/read",
                    "Microsoft.Resources/subscriptions/operationresults/read",
                    "Microsoft.ExtendedLocation/customLocations/Read",
                    "Microsoft.ExtendedLocation/customLocations/deploy/action"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

## Azure Arc SCVMM VM Contributor

Arc SCVMM VM Contributor has permissions to perform all VM actions.


> [!div class="mx-tableFixed"]
> | Actions | Description |
> | --- | --- |
> |Microsoft.Authorization/classicAdministrators/read|Reads the administrators for the subscription.|
> |Microsoft.Authorization/classicAdministrators/operationstatuses/read|Gets the administrator operation statuses of the subscription.|
> |Microsoft.Authorization/denyAssignments/read|Get information about a deny assignment.|
> |Microsoft.Authorization/diagnosticSettingsCategories/read|Get the information about diagnostic settings categories.|
> |Microsoft.Authorization/diagnosticSettings/read|Read the information about diagnostics settings.|
> |Microsoft.Authorization/roleEligibilityScheduleInstances/read|Gets the role eligibility schedule instances at given scope.|
> |Microsoft.Authorization/locks/read|Gets locks at the specified scope.|
> |Microsoft.Authorization/operations/read|Gets the list of operations.|
> |Microsoft.Authorization/permissions/read|Lists all the permissions the caller has at a given scope.|
> |Microsoft.Authorization/policyAssignments/read|Get information about a policy assignment.|
> |Microsoft.Authorization/policyAssignments/privateLinkAssociations/read|Get information about private link association.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/read|Get information about resource management private link.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnections/read|Get information about private endpoint connection.|
> |Microsoft.Authorization/policyAssignments/resourceManagementPrivateLinks/privateEndpointConnectionProxies/read|Get information about private endpoint connection proxy.|
> |Microsoft.Authorization/policyDefinitions/read|Get information about a policy definition.|
> |Microsoft.Authorization/policyDefinitions/versions/read|Get information about a policy definition version.|
> |Microsoft.Authorization/policyEnrollments/read|Get information about a policy enrollment.|
> |Microsoft.Authorization/policyExemptions/read|Get information about a policy exemption.|
> |Microsoft.Authorization/policySetDefinitions/read|Get information about a policy set definition.|
> |Microsoft.Authorization/policySetDefinitions/versions/read|Get information about a policy set definition version.|
> |Microsoft.Authorization/providerOperations/read|Get operations for all resource providers which can be used in role definitions.|
> |Microsoft.Authorization/roleAssignments/read|Get information about a role assignment.|
> |Microsoft.Authorization/roleAssignmentSchedules/read|Gets the role assignment schedules at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleInstances/read|Gets the role assignment schedule instances at given scope.|
> |Microsoft.Authorization/roleAssignmentScheduleRequests/read|Gets the role assignment schedule requests at given scope.|
> |Microsoft.Authorization/roleDefinitions/read|Get information about a role definition.|
> |Microsoft.Authorization/roleEligibilitySchedules/read|Gets the role eligibility schedules at given scope.|
> |Microsoft.Authorization/roleEligibilityScheduleRequests/read|Gets the role eligibility schedule requests at given scope.|
> |Microsoft.Authorization/roleManagementPolicies/read|Get Role management policies.|
> |Microsoft.Authorization/roleManagementPolicyAssignments/read|Get role management policy assignments.|
> |Microsoft.Insights/AlertRules/Write|Create or update a classic metric alert.|
> |Microsoft.Insights/AlertRules/Delete|Delete a classic metric alert.|
> |Microsoft.Insights/AlertRules/Read|Read a classic metric alert.|
> |Microsoft.Insights/AlertRules/Activated/Action|Classic metric alert activated.|
> |Microsoft.Insights/AlertRules/Resolved/Action|Classic metric alert resolved.|
> |Microsoft.Insights/AlertRules/Throttled/Action|Classic metric alert rule throttled.|
> |Microsoft.Insights/AlertRules/Incidents/Read|Read a classic metric alert incident.|
> |Microsoft.Resources/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/deployments/delete|Deletes a deployment.|
> |Microsoft.Resources/deployments/cancel/action|Cancels a deployment.|
> |Microsoft.Resources/deployments/validate/action|Validates an deployment.|
> |Microsoft.Resources/deployments/whatIf/action|Predicts template deployment changes.|
> |Microsoft.Resources/deployments/exportTemplate/action|Export template for a deployment.|
> |Microsoft.Resources/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/read|Gets the list of subscriptions.|
> |Microsoft.Resources/subscriptions/resourceGroups/read|Gets or lists resource groups.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/read|Gets or lists deployments.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/write|Creates or updates an deployment.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read|Gets or lists deployment operations.|
> |Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read|Gets or lists deployment operation statuses.|
> |Microsoft.Resources/subscriptions/operationresults/read|Get the subscription operation results.|
> |Microsoft.ResourceHealth/AvailabilityStatuses/read|Gets the availability statuses for all resources in the specified scope.|
> |Microsoft.HybridCompute/operations/read|Read all Operations for Azure Arc for Servers.|
> |Microsoft.HybridCompute/osType/agentVersions/read|Read all Azure Connected Machine Agent versions available.|
> |Microsoft.HybridCompute/osType/agentVersions/latest/read|Read the latest Azure Connected Machine Agent version.|
> |Microsoft.HybridCompute/licenses/read|Reads any Azure Arc licenses.|
> |Microsoft.HybridCompute/licenses/write|Installs or Updates an Azure Arc licenses.|
> |Microsoft.HybridCompute/licenses/delete|Deletes an Azure Arc licenses.|
> |Microsoft.HybridCompute/locations/operationresults/read|Reads the status of an operation on Microsoft.HybridCompute Resource Provider.|
> |Microsoft.HybridCompute/locations/operationstatus/read|Reads the status of an operation on Microsoft.HybridCompute Resource Provider.|
> |Microsoft.HybridCompute/locations/updateCenterOperationResults/read|Reads the status of an update center operation on machines.|
> |Microsoft.HybridCompute/machines/read|Read any Azure Arc machines.|
> |Microsoft.HybridCompute/machines/write|Writes an Azure Arc machines.|
> |Microsoft.HybridCompute/machines/delete|Deletes an Azure Arc machines.|
> |Microsoft.HybridCompute/machines/UpgradeExtensions/action|Upgrades Extensions on Azure Arc machines.|
> |Microsoft.HybridCompute/machines/assessPatches/action|Assesses any Azure Arc machines to get missing software patches.|
> |Microsoft.HybridCompute/machines/installPatches/action|Installs patches on any Azure Arc machines.|
> |Microsoft.HybridCompute/machines/patchInstallationResults/read|Reads any Azure Arc patchInstallationResults.|
> |Microsoft.HybridCompute/machines/patchInstallationResults/softwarePatches/read|Reads any Azure Arc patchInstallationResults/softwarePatches.|
> |Microsoft.HybridCompute/machines/extensions/read|Reads any Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/extensions/write|Installs or Updates an Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/extensions/delete|Deletes an Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/licenseProfiles/read|Reads any Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/licenseProfiles/write|Installs or Updates an Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/licenseProfiles/delete|Deletes an Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/hybridIdentityMetadata/read|Read any Azure Arc machines's Hybrid Identity Metadata.|
> |Microsoft.HybridCompute/machines/patchAssessmentResults/read|Reads any Azure Arc patchAssessmentResults.|
> |Microsoft.HybridCompute/machines/patchAssessmentResults/softwarePatches/read|Reads any Azure Arc patchAssessmentResults/softwarePatches.|
> |Microsoft.HybridCompute/machines/runcommands/read|Reads any Azure Arc runcommands.|
> |Microsoft.HybridCompute/machines/runcommands/write|Installs or Updates an Azure Arc runcommands.|
> |Microsoft.HybridCompute/machines/runcommands/delete|Deletes an Azure Arc runcommands.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |Microsoft.SCVMM/virtualMachineInstances/read|Retrieves information about a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/write|The operation to create or update a virtual machine instance. Please note some properties can be set only during virtual machine instance creation.|
> |Microsoft.SCVMM/virtualMachineInstances/delete|The operation to delete a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/stop/action|The operation to power off (stop) a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/start/action|The operation to start a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/restart/action|The operation to restart a virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/createCheckpoint/action|Creates a checkpoint in virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/deleteCheckpoint/action|Deletes a checkpoint in virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/restoreCheckpoint/action|Restores to a checkpoint in virtual machine instance.|
> |Microsoft.SCVMM/virtualMachineInstances/guestAgents/read|Implements GuestAgent GET method.|
> |Microsoft.SCVMM/virtualMachineInstances/guestAgents/write|Create Or Update GuestAgent.|
> |Microsoft.SCVMM/virtualMachineInstances/guestAgents/delete|Implements GuestAgent DELETE method.|
> |Microsoft.SCVMM/virtualMachineInstances/hybridIdentityMetadata/read|Implements HybridIdentityMetadata GET method.|
> |Microsoft.SCVMM/virtualmachines/Delete|Deletes virtualmachines.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |

```json
{
    "id": "/providers/Microsoft.Authorization/roleDefinitions/e582369a-e17b-42a5-b10c-874c387c530b",
    "properties": {
        "roleName": "Azure Arc ScVmm VM Contributor",
        "description": "Arc ScVmm VM Contributor has permissions to perform all VM actions.",
        "assignableScopes": [
            "/"
        ],
        "permissions": [
            {
                "actions": [
                    "microsoft.scvmm/virtualmachines/*",
                    "microsoft.scvmm/virtualMachineInstances/*",
                    "Microsoft.Insights/AlertRules/Write",
                    "Microsoft.Insights/AlertRules/Delete",
                    "Microsoft.Insights/AlertRules/Read",
                    "Microsoft.Insights/AlertRules/Activated/Action",
                    "Microsoft.Insights/AlertRules/Resolved/Action",
                    "Microsoft.Insights/AlertRules/Throttled/Action",
                    "Microsoft.Insights/AlertRules/Incidents/Read",
                    "Microsoft.Resources/deployments/read",
                    "Microsoft.Resources/deployments/write",
                    "Microsoft.Resources/deployments/delete",
                    "Microsoft.Resources/deployments/cancel/action",
                    "Microsoft.Resources/deployments/validate/action",
                    "Microsoft.Resources/deployments/whatIf/action",
                    "Microsoft.Resources/deployments/exportTemplate/action",
                    "Microsoft.Resources/deployments/operations/read",
                    "Microsoft.Resources/deployments/operationstatuses/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/write",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read",
                    "Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read",
                    "Microsoft.ResourceHealth/availabilityStatuses/read",
                    "Microsoft.Authorization/*/read",
                    "Microsoft.Resources/subscriptions/read",
                    "Microsoft.Resources/subscriptions/resourceGroups/read",
                    "Microsoft.Resources/subscriptions/operationresults/read",
                    "Microsoft.ExtendedLocation/customLocations/Read",
                    "Microsoft.ExtendedLocation/customLocations/deploy/action",
                    "Microsoft.HybridCompute/machines/read",
                    "Microsoft.HybridCompute/machines/write",
                    "Microsoft.HybridCompute/machines/delete",
                    "Microsoft.HybridCompute/machines/UpgradeExtensions/action",
                    "Microsoft.HybridCompute/machines/assessPatches/action",
                    "Microsoft.HybridCompute/machines/installPatches/action",
                    "Microsoft.HybridCompute/machines/extensions/read",
                    "Microsoft.HybridCompute/machines/extensions/write",
                    "Microsoft.HybridCompute/machines/extensions/delete",
                    "Microsoft.HybridCompute/operations/read",
                    "Microsoft.HybridCompute/locations/operationresults/read",
                    "Microsoft.HybridCompute/locations/operationstatus/read",
                    "Microsoft.HybridCompute/machines/patchAssessmentResults/read",
                    "Microsoft.HybridCompute/machines/patchAssessmentResults/softwarePatches/read",
                    "Microsoft.HybridCompute/machines/patchInstallationResults/read",
                    "Microsoft.HybridCompute/machines/patchInstallationResults/softwarePatches/read",
                    "Microsoft.HybridCompute/locations/updateCenterOperationResults/read",
                    "Microsoft.HybridCompute/machines/hybridIdentityMetadata/read",
                    "Microsoft.HybridCompute/osType/agentVersions/read",
                    "Microsoft.HybridCompute/osType/agentVersions/latest/read",
                    "Microsoft.HybridCompute/machines/runcommands/read",
                    "Microsoft.HybridCompute/machines/runcommands/write",
                    "Microsoft.HybridCompute/machines/runcommands/delete",
                    "Microsoft.HybridCompute/machines/licenseProfiles/read",
                    "Microsoft.HybridCompute/machines/licenseProfiles/write",
                    "Microsoft.HybridCompute/machines/licenseProfiles/delete",
                    "Microsoft.HybridCompute/licenses/read",
                    "Microsoft.HybridCompute/licenses/write",
                    "Microsoft.HybridCompute/licenses/delete"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

## Next steps

- [Create Custom roles](/azure/azure-arc/system-center-virtual-machine-manager/create-custom-roles).
- [Enable inventory resources](/azure/azure-arc/system-center-virtual-machine-manager/enable-scvmm-inventory-resources).
- [Set up and manage self-service access](/azure/azure-arc/system-center-virtual-machine-manager/set-up-and-manage-self-service-access-scvmm).
