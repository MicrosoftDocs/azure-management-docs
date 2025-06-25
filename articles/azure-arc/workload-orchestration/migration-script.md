---
title: Migrate Existing Target Resources to General Availability
description: Learn how to migrate existing target resources in Azure Arc workload orchestration to the general availability (GA) version.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: reference
ms.date: 06/24/2025
---

# Migrate existing target resources to the general availability 

If you used workload orchestration in preview, you need to run a migration script to update the existing target resources to the general availability (GA) version. For more information about the GA release, see [Release Notes for Workload Orchestration](release-notes.md#june-2025-ga-release).

## Prerequisites

Install Azure CLI `resource-graph` extension. If you don't have it installed, run the following command:

```bash
az extension add --name resource-graph
```

## Migration script

You can run the following PowerShell script to migrate your workload orchestration environment to GA. The script updates the `contextId` property of your existing targets to point to the new API version of workload orchestration.

```powershell
param (
    [Parameter(Mandatory = $true)]
    [string]
    $location
)

$namespace = "microsoft.edge"

function List-Contexts() {
    ,(az graph query -q "resources | where type =~ '$namespace/contexts'" -o json | ConvertFrom-JSON).data
}

function Summarize-Targets-By-ResourceGroup() {   
    ,(az graph query --first 1000 -q "resources | where type =~ '$namespace/targets' | where location =~ '$location' | summarize count = count() by subscriptionId, resourceGroup" -o json | ConvertFrom-JSON).data
}

function List-Targets() {
    param (
        [string]$subscriptionId,
        [string]$resourceGroup,
        [string]$apiVersion = "2025-01-01-preview"
    )

    $scope = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/$namespace/targets"
    $result = az rest --method get --uri $scope`?api-version=$apiVersion --output json 2>&1
    if ($LASTEXITCODE -ne 0) {
        throw "GET $scope?api-version=$apiVersion failed with: $result"
    }
    
    ,($result | ConvertFrom-Json).value
}

function Patch-Target() {
    param (
        [string]$id,
        [string]$delta,
        [string]$apiVersion
    )

    $result = (az rest --header Content-Type=application/json --method PATCH --uri $id`?api-version=$apiVersion --body $delta 2>&1)
    if ($LASTEXITCODE -ne 0) {
        throw "PATCH $id?api-version=$apiVersion failed with: $result"
    }
    
    $result | ConvertFrom-JSON 
}

function updateTargetsWithContextId {
    $contexts = List-Contexts
    Write-Host "Found $($contexts.Length) Contexts" -ForegroundColor Cyan
    
    if ($contexts.Length -eq 0) {
        Write-Host "[ERROR] No Contexts found. Unsupported" -ForegroundColor Red
        exit 1
    }

    if ($contexts.Length -gt 1) {
        Write-Host "[ERROR] Greater than 1 Context found. Unsupported" -ForegroundColor Red
        exit 1
    }
    
    $contextId = $contexts[0].id
    Write-Host "Found Context at $contextId" -ForegroundColor Cyan
    
    $subscriptionResourceGroups = Summarize-Targets-By-ResourceGroup
    foreach ($item in $subscriptionResourceGroups) {
        $subscriptionId = $item.subscriptionId
        $resourceGroup = $item.resourceGroup
        
        try {
            $targets = List-Targets -subscriptionId $subscriptionId -resourceGroup $resourceGroup -apiVersion 2025-06-01
            Write-Host "Migration Complete for Subscription $subscriptionId Resource Group $resourceGroup"
            continue
        }
        catch {
            Write-Host "Migrating targets for Subscription $subscriptionId Resource Group $resourceGroup"
        }

        $targets = List-Targets -subscriptionId $subscriptionId -resourceGroup $resourceGroup -apiVersion 2025-01-01-preview
        Write-Host "Found $($targets.Length) targets in Subscription $subscriptionId Resource Group $resourceGroup"
        foreach ($target in $targets) {
            if ($target.location -ne $location) {
                # Write-Host "Skipped target $($target.name) for Subscription $subscriptionId Resource Group $resourceGroup - Location $($target.location) not matched"
                continue
            } 
        
            try {
                Patch-Target -id $target.id -delta "{'properties': {'contextId': '$contextId'}}" -apiVersion 2025-06-01
                Write-Host "Migrated target $($target.name) for Subscription $subscriptionId Resource Group $resourceGroup"
            }
            catch {
                $errorMessage = $_.Exception.Message
                Write-Host "Fail to Migrate $($target.name) for Subscription $subscriptionId Resource Group $resourceGroup - $errorMessage"
            }
        }
        Write-Host "Migration Complete for Subscription $subscriptionId Resource Group $resourceGroup"
    }
}

$tenantId = ((az account show) | ConvertFrom-Json ).tenantId
Write-Host "Migrating $namespace/targets Resources in Tenant $tenantId" -ForegroundColor Cyan
updateTargetsWithContextId
Write-Host "Migration Complete" -ForegroundColor Cyan
```

Run the script in PowerShell with the `-location` parameter set to the Azure region where your workload orchestration environment is deployed. For example, in the `eastus` region, you run the script as follows:

```powershell
.\WOGAMigration.ps1 -location eastus
```