---
title: Delete Resources in Workload Orchestration
description: Learn how to delete the resources created with workload orchestration
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/24/2025
ms.custom:
  - build-2025
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
# Set force-delete argument to true if you want to delete target with installed apps
az workload-orchestration target delete --subscription "$subId" --resource-group "$rg" --target-name "$childName" --force-delete false
```

### [PowerShell](#tab/powershell)

```powershell
# Set force-delete argument to true if you want to delete target with installed apps
az workload-orchestration target delete --subscription $subId --resource-group $rg --target-name $childName --force-delete false
```

***

## Delete a solution template version

Delete solution template version across all targets if it's not deployed using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template remove-version --subscription "$subId" --resource-group "$rg" --solution-template-name "$appName1" --version "$appVersion"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template remove-version --subscription $subId --resource-group $rg --solution-template-name $appName1 --version $appVersion
```

***

## Delete a solution template

Delete a solution template across all targets if it has no versions using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template delete --subscription "$subId" --resource-group "$rg" --solution-template-name "$appName1"
```

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

## Delete existing resources in a resource group 

In the [ZIP folder](https://github.com/microsoft/AEP/blob/main/content/en/docs/Configuration%20Manager%20(Public%20Preview)/Scripts%20for%20Onboarding/Configuration%20manager%20files.zip) you downloaded as part of [Prepare your environment for workload orchestration](initial-setup-environment.md), you can find the PowerShell script `RGCleanScript.ps1` that allows you to clean up resources in a specified Azure resource group. This script is useful for removing all resources created with workload orchestration, including sites, targets, configurations, schemas, and solutions.

> [!NOTE]
> You need to have the necessary permissions to delete resources in the specified resource group. For most cases, by default your alias will have permission.

You execute the script by running the following command in PowerShell, replacing `<YourResourceGroupName>` with the name of your resource group:

```powershell
.\RGCleanScript.ps1 -resourceGroupName <YourResourceGroupName> [-skipSiteAndAddressDeletion $false] [-skipTargetDeletion $false] 
```

The script contains the following parameters, which you can set to customize the cleanup process:

| Parameter                  | Required/Optional | Type    | Description                                                                                      |
|----------------------------|-------------------|---------|--------------------------------------------------------------------------------------------------|
| `resourceGroupName`        | Required          | string  | The name of the resource group to clean.                                                         |
| `subscriptionId`           | Optional          | string  | Subscription ID for resources (For Microsoft.Edge). Defaults to the subscription shown by az CLI. |
| `contextSubscriptionId`    | Optional          | string  | Subscription ID where context is present (For Microsoft.Edge). Defaults to the subscription shown by az CLI. |
| `contextResourceGroupName` | Optional          | string  | Resource group of the Context (For Microsoft.Edge). Default is `Mehoopany`.                      |
| `contextName`              | Optional          | string  | Name of the Context (For Microsoft.Edge). Default is `Mehoopany-Context`.                        |
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
