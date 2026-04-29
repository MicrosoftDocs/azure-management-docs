---
title: Delete Resources in Workload Orchestration
description: Learn how to delete the resources created with workload orchestration
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 06/24/2025
ms.custom:
  - build-2025
# Customer intent: As an IT administrator, I want to delete workload orchestration resources using CLI commands, so that I can manage and free up my cloud environment effectively.
---


# Delete resources in workload orchestration

This article details how to delete or uninstall any workload orchestration resources in Azure, and its cascading impact on other dependent resources, if any. It also introduces a script to clean up multiple resources in a specified Azure resource group, with the option to selectively delete specific resource types.

## Prerequisites

- Set up the required resources for workload orchestration. If you haven't, refer to [Set up workload orchestration](setup-wo.md).

## Delete resources manually

### Uninstall a solution

Any installed solution can be uninstalled from a target using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration target uninstall --resource-group "$rg"  --target-name "$targetName" --solution-template-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target uninstall --resource-group $rg --target-name $targetName --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName
```

***

### Delete a solution revision

Delete a solution revision for a target if it's not installed using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration target remove-revision --resource-group "$rg" --solution-template-version "$version" --target-name "$targetName" --solution-template-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName" --solution-version 1.0.0
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target remove-revision --resource-group $rg --solution-template-version $version --target-name $targetName --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName --solution-version 1.0.0
```

***

### Delete a target

Delete a target and all its child resources using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration target delete --resource-group "$rg" --target-name "$childName"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target delete --subscription $subId --resource-group $rg --target-name $childName
```
***

Target deletion automatically uninstalls all running workloads (instances, solutions, namespaces, and the target itself) from the cluster. The corresponding cloud resources (Target, TargetSolution, TargetSolutionVersion and Instances) are also deleted successfully.

A deleted target can be recreated with the same name successfully. Once the target is re-linked to the relevant common config templates, users can once again start deploying solutions post re-configuring the parameter values.

> [!NOTE]
> Target deletion without the --force-delete flag fails when underlying cluster is in the *stopped* state, due to cluster connectivity issues. Add the arguement `--force-delete true` to successfully delete the target along with all associated cloud resources. Workloads running on the cluster will not be impacted.

***

### Delete a solution template version

Delete solution template version across all targets even if it's deployed, using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template remove-version --resource-group "$rg" --solution-template-name "$appName1" --version "$appVersion"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template remove-version --subscription $subId --resource-group $rg --solution-template-name $appName1 --version $appVersion
```
***

The deletion succeeds even if the solution template version is being referenced by a target solution version, which remains unaffected.
You can verify the deletion of the solution template version by listing all revisions of a solution deployed on a target.

#### [Bash](#tab/bash)

```bash
az workload-orchestration target solution-revision-list --target-name "$childName" --resource-group "$rg" --solution-template-name "<solution-version-id>"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target solution-revision-list --target-name $childName --resource-group $rg --solution-template-name "<solution-version-id>"
```

***

### Delete a solution template

Delete a solution template across all targets using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template delete --resource-group "$rg" --solution-template-name "$appName1"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template delete --subscription $subId --resource-group $rg --solution-template-name $appName1
```

***

Deletion of solution template succeeds even if deployed instances refer to its solution template version. All associated solution template versions also get deleted automatically. Any solutions already reviewed or published, referring to the deleted solution template, can still be deployed. No running workloads are impacted by this deletion.

Any deleted solution template can be recreated successfully with the same name, referencing the same or a different schema. However, solution parameters need to be re-configured to deploy solutions based on the template.

***

### Delete a schema

Delete a schema and all its versions using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration schema delete --subscription "$subId" --resource-group "$rg" --schema-name "$schemaName"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration schema delete --subscription $subId --resource-group $rg --schema-name $schemaName
```

Deleting a schema removes all associated schema versions, and any solution templates referencing the schema cannot be reviewed.


***

### Delete a configuration template

Delete a configuration template and all its versions using the following command:

#### [Bash](#tab/bash)

```bash
az workload-orchestration config-template delete --subscription "$subId" --resource-group "$rg" --config-template-name "$appConfig"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration config-template delete --subscription $subId --resource-group $rg --config-template-name $appConfig
```

Deleting a configuration template removes all its versions and associated hierarchy linkages. A deleted template can be recreated successfully with the same name, referencing the same or a different schema. However, it needs to be linked to hierarchies or targets and configured again. If the recreated configuration template references a different schema and a solution template inherits that schema, the `az workload-orchestration target review` command is expected to fail since the schema properties have changed.


### Delete a Context

#### [Bash](#tab/bash)

```bash
az workload-orchestration context delete --resource-group "$rg" -n "$contextName"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration context delete --resource-group $rg -n $contextName
```
***

Context deletion succeeds even if targets are created within it. All associated site references are deleted, while the cluster and targets are unaffected. Any context related operation that is triggered while deletion or is already in progress will fail.


### Delete a Custom Location

#### [Bash](#tab/bash)

```bash
az customlocation delete --resource-group "$rg" -n "$customLocationName"
```

#### [PowerShell](#tab/powershell)

```powershell
az customlocation delete --resource-group $rg -n $customLocationName
```
***

All targets associated to the custom location also get deleted, and no new targets can be created. Solution revisions in Review state can be published but not deployed. Deployed solutions remain unaffected but cannot be uninstalled as the cluster is no longer connected to workload orchestration.

### Delete a Resource Group

Deleting a resource group results in deletion of all resources within the same.

***

## Delete resources using scripts

To delete all resources created as part of workload orchestration in a resource group, please refer to the [Clean-up script](clean-up-script.md). This script allows you to clean up resources in a specified Azure resource group, including sites, targets, configurations, schemas, and solutions, offering you the flexibility to filter by resource type.

### Run the clean-up script 

You can find the PowerShell script `RGCleanScript.ps1` in the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). Execute the script by running the following command in PowerShell, replacing `<YourResourceGroupName>` with the name of your resource group. You can also specify which resources to delete by using the optional parameters.

```powershell
# Clean only specific resources (safe by default - nothing is deleted unless explicitly specified)
.\RGCleanScript.ps1 -resourceGroupName <YourResourceGroupName> [-deleteSite $true] [-deleteTarget $true] 

# Clean all resources at once
.\RGCleanScript.ps1 -resourceGroupName <YourResourceGroupName> -deleteAll $true

# Clean all deployed instances
.\RGCleanScript.ps1 -resourceGroupName <YourResourceGroupName> [-deleteInstance $true]
```

The `RGCleanScript.ps1` script contains the following parameters, which you can set to customize the cleanup process:

| Parameter                  | Required/Optional | Type    | Description                                                                                      |
|----------------------------|-------------------|---------|--------------------------------------------------------------------------------------------------|
| `resourceGroupName`        | Required          | string  | The name of the resource group to clean.                                                         |
| `subscriptionId`           | Optional          | string  | Subscription ID for resources (For Microsoft.Edge). Defaults to the subscription shown by `az cli`. |
| `contextSubscriptionId`    | Optional          | string  | Subscription ID where context is present (For Microsoft.Edge). Defaults to `az cli subscription`.   |
| `contextResourceGroupName` | Optional          | string  | Resource group of the Context (For Microsoft.Edge).                    |
| `contextName`              | Optional          | string  | Name of the Context (For Microsoft.Edge).                      |
| `deleteSite`               | Optional          | bool    | Delete site resources. Default is `false`.                                                       |
| `deleteTarget`             | Optional          | bool    | Delete target resources. Default is `false`.                                                     |
| `deleteConfiguration`      | Optional          | bool    | Delete CM created configuration resources. Default is `false`.                                   |
| `deleteSchema`             | Optional          | bool    | Delete schema/dynamic schema resources. Default is `false`.                                      |
| `deleteConfigTemplate`     | Optional          | bool    | Delete user created config template resources. Default is `false`.                               |
| `deleteSolution`           | Optional          | bool    | Delete solution template resources. Default is `false`.                                          |
| `deleteInstance`           | Optional          | bool    | Delete application instances. Default is `false`.                                                |
| `deleteAks`                | Optional          | bool    | Delete AKS cluster resources. Default is `false`.                                                |
| `deleteManagedIdentity`    | Optional          | bool    | Delete managed identity resources. Default is `false`.                                           |
| `deleteMicrosoftEdge`      | Optional          | bool    | Delete Microsoft Edge resources. Default is `false`.                                             |
| `deleteAll`                | Optional          | bool    | Delete all resources (sets all delete parameters to true). Default is `false`.                   |


