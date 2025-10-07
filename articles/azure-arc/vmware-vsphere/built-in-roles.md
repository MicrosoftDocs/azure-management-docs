---
title: Azure built-in roles for Azure Arc-enabled VMware vSphere
description: This article lists the Azure built-in roles for Azure Arc-enabled VMware vSphere. It lists Actions, NotActions, DataActions, and NotDataActions.
ms.services: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.topic: reference
author: Jeronika-MS
ms.author: v-gajeronika
ms.date: 07/15/2025
---

# Azure built-in roles for Azure Arc-enabled VMware vSphere

This article lists the Azure built-in roles and their permissions for Azure Arc-enabled VMware vSphere. 
Azure Arc-enabled VMware vSphere has four built-in roles:

- [Azure Arc VMware Administrator role](#azure-arc-vmware-administrator-role) 
- [Azure Arc VMware Private Cloud User](#azure-arc-vmware-private-cloud-user)
- [Azure Arc VMware Private Clouds Onboarding](#azure-arc-vmware-private-clouds-onboarding)
- [Azure Arc VMware VM Contributor](#azure-arc-vmware-vm-contributor)

If the built-in Azure roles don't match your requirements, you can [create custom roles](create-custom-roles.md) with granular permissions. 

## Azure Arc VMware Administrator role

Arc VMware VM Contributor has permission to perform all VMware vSphere actions.

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
> |Microsoft.Resources/deployments/validate/action|Validates a deployment.|
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
> |Microsoft.KubernetesConfiguration/extensions/read|Gets extention instance resource.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |microsoft.connectedvmwarevsphere/unregister/action|unregister RP.|
> |microsoft.connectedvmwarevsphere/register/action/register/action|register RP.|
> |Microsoft.ConnectedVMwarevSphere/clusters/Read|Read clusters.|
> |Microsoft.ConnectedVMwarevSphere/clusters/Write|Writes clusters.|
> |Microsoft.ConnectedVMwarevSphere/clusters/Delete|Deletes clusters.|
> |Microsoft.ConnectedVMwarevSphere/clusters/deploy/action|Deploys on cluster.|
> |Microsoft.ConnectedVMwarevSphere/datastores/Read/Write|Read datastores.|
> |Microsoft.ConnectedVMwarevSphere/datastores/Write|Writes datastores.|
> |Microsoft.ConnectedVMwarevSphere/datastores/Delete|Deletes datastores.|
>|Microsoft.ConnectedVMwarevSphere/datastores/AllocateSpace/action|Allocates on datastores.|
>|Microsoft.ConnectedVMwarevSphere/hosts/Read|Read hosts.|
>|Microsoft.ConnectedVMwarevSphere/hosts/Write|Writes hosts.|
>|Microsoft.ConnectedVMwarevSphere/hosts/Delete|Deletes hosts.|
>|Microsoft.ConnectedVMwarevSphere/hosts/deploy/action|Deploys on host.| 
> |microsoft.connectedvmwarevsphere/locations/operationstatuses/read|Read operationstatus.|
> |microsoft.connectedvmwarevsphere/locations/operationstatuses/write|Write operationstatus.|
> |Microsoft.ConnectedVMwarevSphere/locations/updateCenterOperationResults/read|Reads the status of an update center operation on virtual machines.|
> |Microsoft.ConnectedVMwarevSphere/locations/upgradeExtensionsOperationResults/read|Reads the status of an upgrade extensions operation on virtual machines.|
> |microsoft.connectedvmwarevsphere/operations/read|Read operations.|
> |Microsoft.ConnectedVMwarevSphere/resourcepools/Read|Read resourcepools.|
> |Microsoft.ConnectedVMwarevSphere/resourcepools/Write|Writes resourcepools.|
> |Microsoft.ConnectedVMwarevSphere/resourcepools/Delete|Deletes resourcepools.|
> |Microsoft.ConnectedVMwarevSphere/resourcepools/deploy/action|Deploys on resource pool.|
> |microsoft.connectedvmwarevsphere/skus/read|Get skus.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/Read|Read vcenters.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/Write|Writes vcenters.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/Delete|Deletes vcenters.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/inventoryitems/Delete|Deletes vcenter inventoryitems.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/inventoryitems/Delete|Read vcenter inventoryitems.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/inventoryitems/Write|Writes vcenter inventoryitems.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/inventoryitems/onboard/action|Project vcenters inventoryitems.
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/start/action|Start VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/restart/action|Restart VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/stop/action|Stop VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/Read|Read virtualmachineinstances.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/Write|Writes virtualmachineinstances.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/Delete|Deletes virtualmachineinstances.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/Read|Read virtualmachines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/Write|Writes virtualmachines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/Delete|Deletes virtualmachines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/start/action|Start VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/restart/action|Restart VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/stop/action|Stop VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/installPatches/action|Install patches on Azure Arc VMware machines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/assessPatches/action|Assess patches on Azure Arc VMware machines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/upgradeExtensions/action|Upgrade extensions on Azure Arc VMware machines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/extensions/Delete|Delete extension resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/extensions/Read|Gets extension resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/extensions/Write|Write extension resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/guestagents/Delete|Delete guestagent resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/guestagents/Read|Gets guestagent resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/guestagents/Write|Write guestagent resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/hybridIdentityMetadata/Delete|Deletes hybridIdentityMetadata.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/hybridIdentityMetadata/Read|Gets hybridIdentityMetadata.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/hybridIdentityMetadata/Write|Write hybridIdentityMetadata.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/Read|Read virtualmachinetemplates.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/Write|Writes virtualmachinetemplates.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/Delete|Deletes virtualmachinetemplates.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/clone/action|Clones virtualmachinetemplates.|
> |Microsoft.ConnectedVMwarevSphere/virtualnetworks/Read|Read virtualnetworks.|
> |Microsoft.ConnectedVMwarevSphere/virtualnetworks/Write|Writes virtualnetworks.|
> |Microsoft.ConnectedVMwarevSphere/virtualnetworks/Delete|Deletes virtualnetworks.|
> |Microsoft.ConnectedVMwarevSphere/virtualnetworks/join/action|Deletes virtualnetworks.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |

```json
{ 

    "id": "/providers/Microsoft.Authorization/roleDefinitions/ddc140ed-e463-4246-9145-7c664192013f", 

    "properties": { 

        "roleName": "Azure Arc VMware Administrator role ", 

        "description": "Arc VMware VM Contributor has permissions to perform all connected VMwarevSphere actions.", 

        "assignableScopes": [ 

            "/" 

        ], 

        "permissions": [ 

            { 

                "actions": [ 

                    "Microsoft.ConnectedVMwarevSphere/*", 

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

                    "Microsoft.HybridCompute/licenses/delete", 

                    "Microsoft.ExtendedLocation/customLocations/read", 

                    "Microsoft.ExtendedLocation/customLocations/deploy/action", 

                    "Microsoft.KubernetesConfiguration/extensions/read" 

                ], 

                "notActions": [], 

                "dataActions": [], 

                "notDataActions": [] 

            } 

        ] 

    } 

} 
```

## Azure Arc VMware Private Cloud User

Azure Arc VMware Private Cloud User has permissions to use the VMware resources to deploy VMs.


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
> |Microsoft.KubernetesConfiguration/extensions/read|Gets extension instance resource.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |Microsoft.ConnectedVMwarevSphere/clusters/Read|Read clusters.|
> |Microsoft.ConnectedVMwarevSphere/clusters/deploy/action|Deploys on clusters.|
> |Microsoft.ConnectedVMwarevSphere/datastores/Read|Read datastores.|
> |Microsoft.ConnectedVMwarevSphere/datastores/AllocateSpace/action|Allocates on datastores.|
> |Microsoft.ConnectedVMwarevSphere/hosts/Read|Read hosts.|
> |Microsoft.ConnectedVMwarevSphere/hosts/deploy/action|Deploys on host.|
> |Microsoft.ConnectedVMwarevSphere/resourcepools/Read|Read resourcepools.|
> |Microsoft.ConnectedVMwarevSphere/resourcepools/deploy/action|Deploys on resource pool.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/Read|Read virtualmachinetemplates.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/clone/action|Clones virtualmachinetemplates.|
> |Microsoft.ConnectedVMwarevSphere/virtualnetworks/Read|Read virtualnetworks.|
> |Microsoft.ConnectedVMwarevSphere/virtualnetworks/join/action|Deletes virtualnetworks.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |


```json
{ 

    "id": "/providers/Microsoft.Authorization/roleDefinitions/ce551c02-7c42-47e0-9deb-e3b6fc3a9a83", 

    "properties": { 

        "roleName": "Azure Arc VMware Private Cloud User", 

        "description": "Azure Arc VMware Private Cloud User has permissions to use the VMware cloud resources to deploy VMs.", 

        "assignableScopes": [ 

            "/" 

        ], 

        "permissions": [ 

            { 

                "actions": [ 

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

                    "Microsoft.ConnectedVMwarevSphere/virtualnetworks/join/action", 

                    "Microsoft.ConnectedVMwarevSphere/virtualnetworks/Read", 

                    "Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/clone/action", 

                    "Microsoft.ConnectedVMwarevSphere/virtualmachinetemplates/Read", 

                    "Microsoft.ConnectedVMwarevSphere/resourcepools/deploy/action", 

                    "Microsoft.ConnectedVMwarevSphere/resourcepools/Read", 

                    "Microsoft.ConnectedVMwarevSphere/hosts/deploy/action", 

                    "Microsoft.ConnectedVMwarevSphere/hosts/Read", 

                    "Microsoft.ConnectedVMwarevSphere/clusters/deploy/action", 

                    "Microsoft.ConnectedVMwarevSphere/clusters/Read", 

                    "Microsoft.ConnectedVMwarevSphere/datastores/allocateSpace/action", 

                    "Microsoft.ConnectedVMwarevSphere/datastores/Read", 

                    "Microsoft.ExtendedLocation/customLocations/Read", 

                    "Microsoft.ExtendedLocation/customLocations/deploy/action", 

                    "Microsoft.KubernetesConfiguration/extensions/read" 

                ], 

                "notActions": [], 

                "dataActions": [], 

                "notDataActions": [] 

            } 

        ] 

    } 

} 
```

## Azure Arc VMware Private Clouds Onboarding

Azure Arc VMware Private Clouds Onboarding role has permissions to provision all the required resources for onboard and deboard vCenter instances to Azure.


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
> |Microsoft.Resources/deployments/validate/action|Validates a deployment.|
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
> |Microsoft.KubernetesConfiguration/extensions/write|Creates or updates extension resource.|
> |Microsoft.KubernetesConfiguration/extensions/read|Gets extension instance resource.|
> |Microsoft.KubernetesConfiguration/extensions/delete|Deletes extension instance resource.|
> |Microsoft.KubernetesConfiguration/extensions/operations/read|Gets Async Operation status.|
> |Microsoft.KubernetesConfiguration/extensions/operations/read|Gets available operations of the Microsoft.KubernetesConfiguration resource provider.|
> |Microsoft.ExtendedLocation/customLocations/read|Gets an Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/write|Creates or Updates Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/deploy/action|Deploy permissions to a Custom Location resource.|
> |Microsoft.ExtendedLocation/customLocations/delete|Deletes Custom Location resource.|
> |Microsoft.ResourceConnector/appliances/read|Gets an Appliance resource.|
> |Microsoft.ResourceConnector/appliances/write|Creates or Updates Appliance resource.|
> |Microsoft.ResourceConnector/appliances/delete|Deletes Appliance resource.|
> |Microsoft.ResourceConnector/appliances/listClusterUserCredential/action|Gets an appliance cluster user credential.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/Read|Read vcenters.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/Write|Writes vcenters.|
> |Microsoft.ConnectedVMwarevSphere/vcenters/Delete|Deletes vcenters.|
> |Microsoft.BackupSolutions/VMwareApplications/read|Retrieves a list of applications.|
> |Microsoft.BackupSolutions/VMwareApplications/write|Creates an application.|
> |Microsoft.BackupSolutions/VMwareApplications/delete|Removes an application.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |

```json
{ 

    "id": "/providers/Microsoft.Authorization/roleDefinitions/67d33e57-3129-45e6-bb0b-7cc522f762fa", 

    "properties": { 

        "roleName": "Azure Arc VMware Private Clouds Onboarding", 

        "description": "Azure Arc VMware Private Clouds Onboarding role has permissions to provision all the required resources for onboard and deboard vCenter instances to Azure.", 

        "assignableScopes": [ 

            "/" 

        ], 

        "permissions": [ 

            { 

                "actions": [ 

                    "Microsoft.ConnectedVMwarevSphere/vcenters/Write", 

                    "Microsoft.ConnectedVMwarevSphere/vcenters/Read", 

                    "Microsoft.ConnectedVMwarevSphere/vcenters/Delete", 

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

                    "Microsoft.KubernetesConfiguration/extensions/Write", 

                    "Microsoft.KubernetesConfiguration/extensions/Read", 

                    "Microsoft.KubernetesConfiguration/extensions/Delete", 

                    "Microsoft.KubernetesConfiguration/operations/read", 

                    "Microsoft.KubernetesConfiguration/extensions/operations/read", 

                    "Microsoft.ExtendedLocation/customLocations/Read", 

                    "Microsoft.ExtendedLocation/customLocations/Write", 

                    "Microsoft.ExtendedLocation/customLocations/Delete", 

                    "Microsoft.ExtendedLocation/customLocations/deploy/action", 

                    "Microsoft.ResourceConnector/appliances/Read", 

                    "Microsoft.ResourceConnector/appliances/Write", 

                    "Microsoft.ResourceConnector/appliances/Delete", 

                    "Microsoft.ResourceConnector/appliances/listClusterUserCredential/action", 

                    "Microsoft.BackupSolutions/vmwareapplications/write", 

                    "Microsoft.BackupSolutions/vmwareapplications/delete", 

                    "Microsoft.BackupSolutions/vmwareapplications/read" 

                ], 

                "notActions": [], 

                "dataActions": [], 

                "notDataActions": [] 

            } 

        ] 

    } 

} 

 

{ 

    "id": "/providers/Microsoft.Authorization/roleDefinitions/67d33e57-3129-45e6-bb0b-7cc522f762fa", 

    "properties": { 

        "roleName": "Azure Arc VMware Private Clouds Onboarding", 

        "description": "Azure Arc VMware Private Clouds Onboarding role has permissions to provision all the required resources for onboard and deboard vCenter instances to Azure.", 

        "assignableScopes": [ 

            "/" 

        ], 

        "permissions": [ 

            { 

                "actions": [ 

                    "Microsoft.ConnectedVMwarevSphere/vcenters/Write", 

                    "Microsoft.ConnectedVMwarevSphere/vcenters/Read", 

                    "Microsoft.ConnectedVMwarevSphere/vcenters/Delete", 

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

                    "Microsoft.KubernetesConfiguration/extensions/Write", 

                    "Microsoft.KubernetesConfiguration/extensions/Read", 

                    "Microsoft.KubernetesConfiguration/extensions/Delete", 

                    "Microsoft.KubernetesConfiguration/operations/read", 

                    "Microsoft.KubernetesConfiguration/extensions/operations/read", 

                    "Microsoft.ExtendedLocation/customLocations/Read", 

                    "Microsoft.ExtendedLocation/customLocations/Write", 

                    "Microsoft.ExtendedLocation/customLocations/Delete", 

                    "Microsoft.ExtendedLocation/customLocations/deploy/action", 

                    "Microsoft.ResourceConnector/appliances/Read", 

                    "Microsoft.ResourceConnector/appliances/Write", 

                    "Microsoft.ResourceConnector/appliances/Delete", 

                    "Microsoft.ResourceConnector/appliances/listClusterUserCredential/action", 

                    "Microsoft.BackupSolutions/vmwareapplications/write", 

                    "Microsoft.BackupSolutions/vmwareapplications/delete", 

                    "Microsoft.BackupSolutions/vmwareapplications/read" 

                ], 

                "notActions": [], 

                "dataActions": [], 

                "notDataActions": [] 

            } 

        ] 

    } 

} 
```

## Azure Arc VMware VM Contributor

Arc VMware VM Contributor has permissions to perform all VM actions.


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
> |Microsoft.Resources/deployments/validate/action|Validates a deployment.|
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
> |Microsoft.HybridCompute/machines/licenseProfiles/read|Reads any Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/licenseProfiles/write|Installs or Updates an Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/licenseProfiles/delete|Deletes an Azure Arc extensions.|
> |Microsoft.HybridCompute/machines/hybridIdentityMetadata/read|Reads any Azure Arc licenseProfiles.|
> |Microsoft.HybridCompute/machines/patchAssessmentResults/read|Read any Azure Arc machine's Hybrid Identity Metadata.|
> |Microsoft.HybridCompute/machines/patchAssessmentResults/softwarePatches/read|Reads any Azure Arc patchAssessmentResults/softwarePatches.|
> |Microsoft.HybridCompute/machines/runcommands/read|Reads any Azure Arc runcommands.|
> |Microsoft.HybridCompute/machines/runcommands/write|Installs or Updates an Azure Arc runcommands.|
> |Microsoft.HybridCompute/machines/runcommands/delete|Deletes an Azure Arc runcommands.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/start/action|Start VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/restart/action|Restart VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/stop/action|Stop VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/Read|Read virtualmachineinstances.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/Write|Writes virtualmachineinstances.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/Delete|Deletes virtualmachineinstances.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/Read|Read virtualmachines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/Write|Writes virtualmachines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/Delete|Deletes virtualmachines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/start/action|Start VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/restart/action|Restart VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/stop/action|Stop VM.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/installPatches/action|Install patches on Azure Arc VMware machines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/assessPatches/action|Assess patches on Azure Arc VMware machines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/upgradeExtensions/action|Upgrade extensions on Azure Arc VMware machines.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/extensions/Delete|Delete extension resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/extensions/Read|Gets extension resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/extensions/Write|Writes extension resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/guestagents/Delete|Delete guestagent resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/guestagents/Read|Gets guestagent resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/guestagents/Write|Write guestagent resource.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/hybridIdentityMetadata/Delete|Deletes hybridIdentityMetadata.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/hybridIdentityMetadata/Read|Gets hybridIdentityMetadata.|
> |Microsoft.ConnectedVMwarevSphere/virtualmachines/hybridIdentityMetadata/Write|Write hybridIdentityMetadata.|
> | **NotActions** |  |
> | *none* |  |
> | **DataActions** |  |
> | *none* |  |
> | **NotDataActions** |  |
> | *none* |  |

```json
{ 

    "id": "/providers/Microsoft.Authorization/roleDefinitions/b748a06d-6150-4f8a-aaa9-ce3940cd96cb", 

    "properties": { 

        "roleName": "Azure Arc VMware VM Contributor", 

        "description": "Arc VMware VM Contributor has permissions to perform all VM actions.", 

        "assignableScopes": [ 

            "/" 

        ], 

        "permissions": [ 

            { 

                "actions": [ 

                    "Microsoft.ConnectedVMwarevSphere/virtualmachines/*", 

                    "Microsoft.ConnectedVMwarevSphere/virtualmachineinstances/*", 

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

                "notActions": [], 

                "dataActions": [], 

                "notDataActions": [] 

            } 

        ] 

    } 

} 
```

## Next steps

- [Create Custom roles](create-custom-roles.md).
- [Set up and manage self-service access](setup-and-manage-self-service-access.md).
