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

You can run the following PowerShell script to clean up resources in a specified Azure resource group. The script allows you to delete resources such as sites, targets, configurations, schemas, and solutions created with workload orchestration.

```powershell
# Input Prameters
param (
    [string]$subscriptionId,
    [string]$contextSubscriptionId,
    [string]$contextName="Mehoopany-Context",
    [string]$contextResourceGroupName="Mehoopany",
    [string]$resourceGroupName,
    [string]$sgSiteNames = "",
    [bool]$deleteSite = $false,
    [bool]$deleteTarget = $false,
    [bool]$deleteConfiguration = $false,
    [bool]$deleteSchema = $false,
    [bool]$deleteConfigTemplate = $false,
    [bool]$deleteSolution = $false,
    [bool]$deleteInstance = $false,
    [bool]$deleteAks = $false,
    [bool]$deleteManagedIdentity = $false,
    [bool]$deleteMicrosoftEdge = $true,
    [bool]$deleteAll = $false
)

# If deleteAll is true, set all delete parameters to true
if ($deleteAll) {
    $deleteSite = $true
    $deleteTarget = $true
    $deleteConfiguration = $true
    $deleteSchema = $true
    $deleteConfigTemplate = $true
    $deleteSolution = $true
    $deleteInstance = $true
    $deleteAks = $true
    $deleteManagedIdentity = $true
    $deleteMicrosoftEdge = $true
}

# Import RGCleanCommon.ps1 from same directory
$scriptDir = Split-Path -Parent $MyInvocation.MyCommand.Path
$commonScriptPath = Join-Path $scriptDir "RGCleanCommon.ps1"
if (Test-Path $commonScriptPath) {
    . $commonScriptPath
} else {
    Write-Host "##[error] Common script not found at $commonScriptPath" -ForegroundColor Red
    return
}

function deleteMicrosoftEdge {
    Write-Host "##[section] Deleting Microsoft Edge resources" -ForegroundColor Yellow
    az cloud update --endpoint-resource-manager "https://eastus2euap.management.azure.com"
    $baseURI = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/microsoft.edge"

    try {
        if ($deleteTarget -eq $true) {
            deleteTargets -baseURI $baseURI -apiVersion $PublicPreviewAPIVersion -nameSpace "Microsoft.Edge"
        }
        elseif ($deleteTarget -eq $false -and $deleteInstance -eq $true) {
            Write-Host "##[section] Skipping Targets Deletion, but deleting instances" -ForegroundColor Yellow
            deleteInstances -baseURI $baseURI -apiVersion $PublicPreviewAPIVersion -nameSpace "Microsoft.Edge"    
        }
        else {
            Write-Host "##[section] Skipping Targets Deletion" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while deleting targets" -ForegroundColor Red
    }

    
    try {
        if ($deleteSolution -eq $true) {
            deleteSolutionsAndItsChildrens -baseURI $baseURI -apiVersion $PublicPreviewAPIVersion
        }
        else {
            Write-Host "##[section] Skipping Solutions Deletion" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while deleting solutions" -ForegroundColor Red
    }
    
    try {
        if ($deleteConfigTemplate -eq $true) {
            deleteConfigTemplates -baseURI $baseURI -apiVersion $PublicPreviewAPIVersion
        }
        else {
            Write-Host "##[section] Skipping Config Templates Deletion" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while deleting config templates" -ForegroundColor Red
    }

    try {
        if ($deleteConfiguration -eq $true) {
            deleteConfigurationsAndItsChildrens -baseURI $baseURI -apiVersion $ConfigRTAPIVersion
        }
        else {
            Write-Host "##[section] Skipping Configuration Deletion" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while deleting configurations" -ForegroundColor Red
    }

    try {
        if ($deleteSchema -eq $true) {
            deleteSchemas -baseURI $baseURI -apiVersion $PublicPreviewAPIVersion
        }
        else {
            Write-Host "##[section] Skipping schema Deletion" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while deleting schemas" -ForegroundColor Red
    }
    
    try {
        if ($deleteSite -eq $true) {
            # invoke if $sgSiteNames is not empty
            if ($sgSiteNames -ne "") {
                deleteSgSitesAndSiteRefsForMicrosoftEdge -sgSiteNames $sgSiteNames -contextName $contextName -contextResourceGroupName $contextResourceGroupName -contextSubscriptionId $contextSubscriptionId
            }
            else {
                Write-Host "##[section] No SG based sites provided, skipping deletion" -ForegroundColor Yellow
            }
            
            deleteRGSites -resourceGroupName $resourceGroupName -subscriptionId $subscriptionId
        }
        else {
            Write-Host "##[section] Skipping Sites Deletion" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while deleting sites" -ForegroundColor Red
    }

    try {
        removeTestCapabilities -baseURI $contextBaseUri -apiVersion $PublicPreviewAPIVersion
    } catch {
        Write-Host "##[debug] Error: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "##[debug] An error occurred while cleaning up caabilities" -ForegroundColor Red
    }
    
}

# Script starts here
if ($resourceGroupName -eq $null -or $resourceGroupName -eq "") {
    Write-Host "##[error] Please provide a resource group name with -resourceGroupName flag" -ForegroundColor Red
    return
}

if($subscriptionId -eq $null -or $subscriptionId -eq "") {
    $subscriptionId = az account show --query id -o tsv
    if ($null -eq $subscriptionId -or $subscriptionId -eq "") {
        Write-Host "##[error] Please login to Azure CLI using 'az login'" -ForegroundColor Red
        return
    } else {
        Write-Host "##[debug] Using default Subscription ID: $subscriptionId" -ForegroundColor Green
    }
}

if($contextSubscriptionId -eq $null -or $contextSubscriptionId -eq "") {
    $contextSubscriptionId = az account show --query id -o tsv
    if ($null -eq $contextSubscriptionId -or $contextSubscriptionId -eq "") {
        Write-Host "##[error] Please login to Azure CLI 'az login'" -ForegroundColor Red
        return
    } else {
        Write-Host "##[debug] Using default Context Subscription ID: $contextSubscriptionId" -ForegroundColor Green
    }
}

$currentEp = ((az cloud show) | ConvertFrom-Json ).endpoints.resourceManager

if ($deleteMicrosoftEdge -eq $true) {
    deleteMicrosoftEdge 
}
else {
    Write-Host "##[section] Skipping Microsoft.Edge Resource Deletion" -ForegroundColor Yellow
}

deleteCommon -skipAksDeletion (-not $deleteAks) -skipManagedIdentityDeletion (-not $deleteManagedIdentity) -resourceGroup $resourceGroupName -subscriptionId $subscriptionId

az cloud update --endpoint-resource-manager "$currentEp"
```

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
