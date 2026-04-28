---
title: Uninstall the Azure Connected Machine agent
description: This article describes how to remove the Azure Connected Machine agent from Azure Arc-enabled servers.
ms.date: 04/27/2026
ms.topic: how-to
# Customer intent: As a system administrator, I want to uninstall the Azure Connected Machine agent from Azure Arc-enabled servers, so that I can remove management of these servers from Azure Arc.
---

# Uninstall the Azure Connected Machine agent from Arc-enabled servers

If you no longer want to manage a machine through Azure Arc-enabled servers, you must remove any VM extensions from the server, disconnect the agent, and uninstall the software from your server. It's important to complete all of these steps to fully remove all related software components from your system.

## Remove VM extensions

If you deployed Azure VM extensions to an Azure Arc-enabled server, you must uninstall all extensions before disconnecting the agent or uninstalling the software. Uninstalling the Azure Connected Machine agent doesn't automatically remove extensions, and these extensions aren't recognized if you reconnect the server to Azure Arc.

For guidance on how to list and remove any extensions on your Azure Arc-enabled server, see the following resources:

* [Manage VM extensions with the Azure portal](manage-vm-extensions-portal.md)
* [Manage VM extensions with Azure PowerShell](manage-vm-extensions-powershell.md#)
* [Manage VM extensions with Azure CLI](manage-vm-extensions-cli.md)

## Disconnect the server from Azure Arc

After you remove all extensions from your Arc-enabled server, the next step is to disconnect the agent. This action deletes the corresponding Azure resource for the server and clears the local state of the agent.

To disconnect the agent, run the `azcmagent disconnect` command as an administrator on the server. You're prompted to sign in with an Azure account that has permission to delete the resource in your subscription. If the resource was already deleted in Azure, pass an additional flag to clean up the local state: `azcmagent disconnect --force-local-only`.

If your Administrator and Azure accounts are different, you might encounter problems with the sign-in prompt defaulting to the Administrator account. To resolve these problems, run the `azcmagent disconnect --use-device-code` command. You're prompted to sign in with an Azure account that has permission to delete the resource.

> [!CAUTION]
> When disconnecting the agent from Arc-enabled VMs running on Azure Local, use only the `azcmagent disconnect --force-local-only` command. Using the command without the `--force-local-only` flag can cause your Arc VM on Azure Local to be deleted both from Azure and on-premises.

## Uninstall the agent

After you disconnect the agent from Azure Arc, remove the Connected Machine agent from the server.

### [Windows](#tab/windows)

Both of the following methods remove the agent, but they don't remove the *C:\Program Files\AzureConnectedMachineAgent* folder on the machine.

#### Uninstall from Control Panel

Follow these steps to uninstall the Windows agent from the machine:

1. Sign in to the computer with an account that has administrator permissions.
1. In **Control panel**, select **Programs and Features**.
1. In **Programs and Features**, select **Azure Connected Machine Agent**, select **Uninstall**, and then select **Yes**.

You can also delete the Windows agent directly from the agent setup wizard. Run the **AzureConnectedMachineAgent.msi** installer package to do so.

#### Uninstall from the command line

You can uninstall the agent manually from the Command Prompt or by using an automated method (such as a script) by following the example below. First you need to retrieve the product code, which is a GUID that is the principal identifier of the application package, from the operating system. The uninstall is performed by using the Msiexec.exe command line - `msiexec /x {Product Code}`.

1. Open the Registry Editor.

2. Under registry key `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall`, look for and copy the product code GUID.

3. Uninstall the agent using Msiexec, as in the following examples:

   * From the command line, enter the following command:

       ```dos
       msiexec.exe /x {product code GUID} /qn
       ```

   * You can perform the same steps using PowerShell:

       ```powershell
       Get-ChildItem -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall | `
       Get-ItemProperty | `
       Where-Object {$_.DisplayName -eq "Azure Connected Machine Agent"} | `
       ForEach-Object {MsiExec.exe /x "$($_.PsChildName)" /qn}
       ```

### [Linux - apt](#tab/linux-apt)

> [!NOTE]
> To uninstall the agent, you need *root* access permissions, or an account with elevated rights using sudo.

Run the following command to remove the agent:

```bash
sudo apt purge azcmagent
```

### [Linux - yum](#tab/linux-yum)

> [!NOTE]
> To uninstall the agent, you need *root* access permissions, or an account with elevated rights using sudo.

Run the following command to remove the agent:

```bash
sudo yum remove azcmagent
```

### [Linux - zypper](#tab/linux-zypper)

> [!NOTE]
> To uninstall the agent, you need *root* access permissions, or an account with elevated rights using sudo.

Run the following command to remove the agent:

```bash
sudo zypper remove azcmagent
```
---

## Remove stale server resources

If you decommission or disconnect a server without cleanly removing the agent, the resource usually remains in the Azure portal with a **Disconnected** status. Over time, these stale resources can clutter your environment.

The following PowerShell script helps you identify and delete Azure Arc-enabled servers that have been disconnected for a specified number of days.

### Prerequisites

*   **Azure PowerShell**: The `Az` module installed.
*   **Permissions**: Reader access to query via Azure Resource Graph, and Contributor or Owner (or `Microsoft.HybridCompute/machines/delete`) to delete the resources.

### The cleanup script

This script uses [Azure Resource Graph](/azure/governance/resource-graph/overview) to query `Microsoft.HybridCompute/machines` resources where the status is `Disconnected` and the `lastStatusChange` timestamp is older than the configured threshold.

Save the following code as `Cleanup-StaleArcServers.ps1`.

```powershell
<#
.SYNOPSIS
    Identifies and removes stale Azure Arc-enabled servers that have been disconnected for a specified number of days.

.DESCRIPTION
    This script queries Azure Resource Graph to find Azure Arc-enabled servers (Microsoft.HybridCompute/machines)
    that have a status of 'Disconnected' and have not updated their status for more than the specified number of days.
    Supports -WhatIf and -Confirm via SupportsShouldProcess.

.PARAMETER DaysDisconnected
    The number of days a server must be disconnected to be considered stale. Default is 60.

.PARAMETER Subscription
    Optional. One or more Subscription IDs to scope the query to.
    If not specified, queries all subscriptions the current context has access to.

.PARAMETER ManagementGroup
    Optional. A Management Group name to scope the query to.
    Cannot be used together with -Subscription.

.EXAMPLE
    .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 60 -WhatIf
    Lists Arc servers disconnected for more than 60 days across all subscriptions.

.EXAMPLE
    .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 90 -Subscription 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
    Permanently deletes Arc servers disconnected for more than 90 days in the specified subscription.

.NOTES
    Author: Microsoft
    Date: 2026-01-12
    Requires: Az.ResourceGraph, Az.Resources
#>

[CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
param (
    [int]$DaysDisconnected = 60,
    
    [ValidateNotNullOrEmpty()]
    [string[]]$Subscription,

    [ValidateNotNullOrEmpty()]
    [string]$ManagementGroup
)

# Check required modules
foreach ($mod in @('Az.ResourceGraph', 'Az.Resources')) {
    if (-not (Get-Module -Name $mod -ListAvailable -ErrorAction SilentlyContinue)) {
        Write-Error "Required module '$mod' is not installed. Run: Install-Module $mod -Scope CurrentUser"
        return
    }
}

# Check for Azure connection
try {
    $context = Get-AzContext -ErrorAction Stop
    Write-Host "Connected to Azure context: $($context.Name)" -ForegroundColor Cyan
}
catch {
    Write-Error "Not connected to Azure. Please run 'Connect-AzAccount' first."
    return
}

# Construct the KQL query
# We look for resources of type hybridcompute/machines
# Status must be Disconnected
# lastStatusChange must be older than $DaysDisconnected
$kqlQuery = @"
Resources
| where type == 'microsoft.hybridcompute/machines'
| where properties.status == 'Disconnected'
| where properties.lastStatusChange < ago($($DaysDisconnected)d)
| project id, name, resourceGroup, subscriptionId, location, status = properties.status, lastStatusChange = properties.lastStatusChange
"@

Write-Host "Searching for Arc servers disconnected for more than $DaysDisconnected days..." -ForegroundColor Yellow

# Execute Search with pagination (Search-AzGraph returns max 1000 results per call)
$staleServers = [System.Collections.Generic.List[object]]::new()
$skipToken = $null

try {
    do {
        $params = @{
            Query = $kqlQuery
            First = 1000
        }
        if ($skipToken) {
            $params['SkipToken'] = $skipToken
        }
        if ($Subscription) {
            $params['Subscription'] = $Subscription
        }
        if ($ManagementGroup) {
            $params['ManagementGroup'] = $ManagementGroup
        }

        $result = Search-AzGraph @params -ErrorAction Stop
        if ($result) {
            $staleServers.AddRange([object[]]$result)
            $skipToken = $result.SkipToken
        }
    } while ($skipToken)
}
catch {
    Write-Error "Failed to query Azure Resource Graph. Ensure resource graph module is installed and you have read permissions.`nError: $_"
    return
}

if ($staleServers.Count -eq 0) {
    Write-Host "No stale Arc servers found matching the criteria." -ForegroundColor Green
    return
}

Write-Host "Found $($staleServers.Count) stale servers." -ForegroundColor Yellow

# Process results
$successCount = 0
$failCount = 0

foreach ($server in $staleServers) {
    if ($PSCmdlet.ShouldProcess($server.id, "Delete stale Arc server '$($server.name)' (Disconnected since: $($server.lastStatusChange))")) {
        try {
            Remove-AzResource -ResourceId $server.id -Force -ErrorAction Stop
            Write-Host "Successfully deleted '$($server.name)'." -ForegroundColor Green
            $successCount++
        }
        catch {
            Write-Error "Failed to delete '$($server.name)'. Error: $_"
            $failCount++
        }
    }
}

Write-Host "`nCleanup completed. Deleted: $successCount, Failed: $failCount, Total: $($staleServers.Count)" -ForegroundColor Green
```

### How to use the script

#### Prerequisites

The script requires the following PowerShell modules:

- `Az.ResourceGraph`
- `Az.Resources`

If they're not installed, run:

```powershell
Install-Module Az.ResourceGraph, Az.Resources -Scope CurrentUser
```

#### Steps

1. Sign in to Azure

   Open your PowerShell terminal and sign in.

   ```powershell
   Connect-AzAccount
   ```

1. Run a What-If analysis

      First, run the script with the `-WhatIf` switch. This step lists the servers that meet the criteria without actually deleting them. This command checks servers that have been disconnected for 60 days or more.

   ```powershell
   .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 60 -WhatIf
   ```

   To scope the query to a specific subscription, use the `-Subscription` parameter:

   ```powershell
   .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 60 -Subscription 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' -WhatIf
   ```

   To scope the query to a management group instead, use the `-ManagementGroup` parameter:

   ```powershell
   .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 60 -ManagementGroup 'MyManagementGroup' -WhatIf
   ```

   Review the output to ensure it lists only the intended servers.

1. Perform the cleanup

   When you're confident in the list of servers to remove, run the script without the `-WhatIf` switch.

   ```powershell
   .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 60
   ```

   To clean up servers disconnected for a longer period (for example, six months), increase the day count:

   ```powershell
   .\Cleanup-StaleArcServers.ps1 -DaysDisconnected 180
   ```

   The script prompts for confirmation before each deletion. To skip prompts, add `-Confirm:$false`.

## Related content

* Learn how to [manage Connected Machine agent versions on Arc-enabled servers](manage-agent.md).
* Learn how to [configure Azure Connected Machine agent proxy settings](manage-agent-proxy-settings.md).
* Learn how to [rename an Arc-enabled server or move it to a different Azure region](manage-howto-migrate.md).
