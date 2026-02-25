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

This article details how to delete or uninstall any workload orchestration resources in Azure. You can delete resources such as targets, solution templates, schema, and configuration templates.

IT users can delete resources using the Azure CLI. For more information about workload orchestration, see [Prepare your environment for workload orchestration](initial-setup-environment.md).

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Uninstall a solution

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

## Delete a solution revision

Delete a solution revision for a target if it's not installed using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration target remove-revision --resource-group "$rg" --solution-template-version "$version" --target-name "$targetName" --solution-template-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName" --solution-version 1.0.0
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target remove-revision --resource-group $rg --solution-template-version $version --target-name $targetName --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName --solution-version 1.0.0
```

***

## Delete a target

Delete a target and all its child resources using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration target delete --resource-group "$rg" --target-name "$childName"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target delete --subscription $subId --resource-group $rg --target-name $childName
```

Target deletion automatically uninstalls all running workloads (instances, solutions, namespaces, and the target itself) from the cluster. The corresponding cloud resources (Target, TargetSolution, TargetSolutionVersion and Instances) are also deleted successfully.

> [!NOTE]
> Target deletion without the --force-delete flag fails when underlying cluster is in the *stopped* state, due to cluster connectivity issues. Add the arguement `--force-delete true` to successfully delete the target along with all associated cloud resources. Workloads running on the cluster will not be impacted.

***

## Delete a solution template version

Delete solution template version across all targets even if it's deployed, using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template remove-version --resource-group "$rg" --solution-template-name "$appName1" --version "$appVersion"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template remove-version --subscription $subId --resource-group $rg --solution-template-name $appName1 --version $appVersion
```

The deletion succeeds even if the solution template version is being reference by a target solution version, which remains unaffected.
You can verify the deletion of the solution template version by listing all revisions of a solution deployed on a target.

### [Bash](#tab/bash)

```bash
az workload-orchestration target solution-revision-list --target-name "$childName" --resource-group "$rg" --solution-template-name "$appName1"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target solution-revision-list --target-name $childName --resource-group $rg --solution-template-name $appName1
```

***

## Delete a solution template

Delete a solution template across all targets using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template delete --resource-group "$rg" --solution-template-name "$appName1"
```

Deletion of solution template succeeds even if deployed instances refer to its solution template version. All associated solution template versions also get deleted automatically.

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template delete --subscription $subId --resource-group $rg --solution-template-name $appName1
```

***

## Delete a schema

Delete a schema and all its versions using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration schema delete --subscription "$subId" --resource-group "$rg" --schema-name "$schemaName"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration schema delete --subscription $subId --resource-group $rg --schema-name $schemaName
```

***

### Delete a configuration template

Delete a configuration template and all its versions using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration config-template delete --subscription "$subId" --resource-group "$rg" --config-template-name "$appConfig"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration config-template delete --subscription $subId --resource-group $rg --config-template-name $appConfig
```

***

### Delete a Context

### [Bash](#tab/bash)

```bash
az workload-orchestration context delete --resource-group "$rg" -n "$contextName"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration context delete --resource-group $rg -n $contextName
```
Context deletion succeeds even if targets are created within it. All associated site references are deleted, while the cluster and targets are unaffected. Any context related operation that is triggered while deletion or is already in progress will fail.


***

### Delete a Custom Location

### [Bash](#tab/bash)

```bash
az customlocation delete --resource-group "$rg" -n "$customLocationName"
```

### [PowerShell](#tab/powershell)

```powershell
az customlocation delete --resource-group $rg -n $customLocationName
```
All targets associated to the custom location also get deleted, and no new targets can be created. Solution revisions in Review state can be published but not deployed. Deployed solutions remain unaffected but cannot be uninstalled as the cluster is no longer connected to workload orchestration.

***

## Delete existing resources in a resource group 

To delete all resources created with workload orchestration in a resource group, see [Clean-up script](clean-up-script.md). This script allows you to clean up resources in a specified Azure resource group, including sites, targets, configurations, schemas, and solutions.
