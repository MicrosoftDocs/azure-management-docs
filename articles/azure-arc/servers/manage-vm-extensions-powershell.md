---
title: Enable VM Extensions Using Azure PowerShell
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments by using Azure PowerShell.
ms.date: 06/19/2025
ms.topic: how-to 
ms.custom:
  - devx-track-azurepowershell
  - build-2025
# Customer intent: As a system administrator, I want to deploy and manage VM extensions on Azure Arc-enabled servers using PowerShell, so that I can efficiently manage my hybrid cloud environment and enhance the functionality of my virtual machines.
---

# Enable Azure VM extensions by using Azure PowerShell

This article explains how to deploy, update, and uninstall [virtual machine (VM) extensions](manage-vm-extensions.md) on Azure Arc-enabled servers by using Azure PowerShell.

## Prerequisites

- A computer with Azure PowerShell. For instructions, see [Install and configure Azure PowerShell](/powershell/azure/).

- The `Az.ConnectedMachine` module. Before you use Azure PowerShell to manage VM extensions on your hybrid server managed by Azure Arc-enabled servers, you need to install this module.

  You can perform these management operations from your workstation, rather than on the Azure Arc-enabled server.

  Run the following command to install the `Az.ConnectedMachine` module:

  ```powershell
  Install-Module -Name Az.ConnectedMachine
  ```

## Enable an extension

To enable a VM extension on your Azure Arc-enabled server, use [`New-AzConnectedMachineExtension`](/powershell/module/az.connectedmachine/new-azconnectedmachineextension) with the `-Name`, `-ResourceGroupName`, `-MachineName`, `-Location`, `-Publisher`, -`ExtensionType`, and `-Settings` parameters.

This example enables the Custom Script Extension on an Azure Arc-enabled server:

```powershell
$Setting = @{ "commandToExecute" = "powershell.exe -c Get-Process" }
New-AzConnectedMachineExtension -Name "custom" -ResourceGroupName "myResourceGroup" -MachineName "myMachineName" -Location "regionName" -Publisher "Microsoft.Compute"  -Settings $Setting -ExtensionType CustomScriptExtension
```

This example enables the Microsoft Antimalware extension on an Azure Arc-enabled Windows server:

```powershell
$Setting = @{ "AntimalwareEnabled" = $true }
New-AzConnectedMachineExtension -Name "IaaSAntimalware" -ResourceGroupName "myResourceGroup" -MachineName "myMachineName" -Location "regionName" -Publisher "Microsoft.Azure.Security" -Settings $Setting -ExtensionType "IaaSAntimalware"
```

This example enables the Key Vault VM extension on an Azure Arc-enabled server:

> [!WARNING]
> Adding `\` to `"` in the settings.json file will cause `akvvm_service` to fail with the following error: `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`
>
> Although PowerShell users commonly use the `\"` sequence to escape quotation marks in other code blocks, you should avoid that formatting in the settings.json file.

```powershell
# Build settings
    $settings = @{
      secretsManagementSettings = @{
       observedCertificates = @(
        "observedCert1"
       )
      certificateStoreLocation = "myMachineName" # For Linux use "/var/lib/waagent/Microsoft.Azure.KeyVault.Store/"
      certificateStore = "myCertificateStoreName"
      pollingIntervalInS = "pollingInterval"
      }
    authenticationSettings = @{
     msiEndpoint = "http://localhost:40342/metadata/identity"
     }
    }

    $resourceGroup = "resourceGroupName"
    $machineName = "myMachineName"
    $location = "regionName"

    # Start the deployment
    New-AzConnectedMachineExtension -ResourceGroupName $resourceGroup -Location $location -MachineName $machineName -Name "KeyVaultForWindows or KeyVaultforLinux" -Publisher "Microsoft.Azure.KeyVault" -ExtensionType "KeyVaultforWindows or KeyVaultforLinux" -Setting $settings
```

This example enables the Datadog VM extension on an Azure Arc-enabled server:

```azurepowershell
$resourceGroup = "resourceGroupName"
$machineName = "machineName"
$location = "machineRegion"
$osType = "Windows" # change to Linux if appropriate
$settings = @{
    # change to your preferred Datadog site
    site = "us3.datadoghq.com"
}
$protectedSettings = @{
    # change to your Datadog API key
    api_key = "APIKEY"
}

New-AzConnectedMachineExtension -ResourceGroupName $resourceGroup -Location $location -MachineName $machineName -Name "Datadog$($osType)Agent" -Publisher "Datadog.Agent" -ExtensionType "Datadog$($osType)Agent" -Setting $settings -ProtectedSetting $protectedSettings
```

> [!TIP]
> Many other extensions are supported on Arc-enabled servers. For details, see [Virtual machine extension management with Azure Arc-enabled servers](manage-vm-extensions.md#extensions).

## List extensions installed

To get a list of the VM extensions on your Azure Arc-enabled server, use [`Get-AzConnectedMachineExtension`](/powershell/module/az.connectedmachine/get-azconnectedmachineextension) with the `-MachineName` and `-ResourceGroupName` parameters:

```powershell
Get-AzConnectedMachineExtension -ResourceGroupName myResourceGroup -MachineName myMachineName

Name    Location  PropertiesType        ProvisioningState
----    --------  --------------        -----------------
custom  westus2   CustomScriptExtension Succeeded
```

## Update an extension configuration

To reconfigure an installed extension, you can use the `Update-AzConnectedMachineExtension` cmdlet with the `-Name`, `-MachineName`, `-ResourceGroupName`, and `-Settings` parameters.

For more details, see [`Update-AzConnectedMachineExtension`](/powershell/module/az.connectedmachine/update-azconnectedmachineextension).

## Upgrade extensions

When a new version of a supported VM extension is released, you can upgrade it to that latest release. To upgrade a VM extension, use [`Update-AzConnectedExtension`](/powershell/module/az.connectedmachine/update-azconnectedextension) with the `-MachineName`, `-ResourceGroupName`, and `-ExtensionTarget` parameters.

For the `-ExtensionTarget` parameter, you need to specify the extension and the latest version available. To determine the latest version available for an extension, go to the **Extensions** page for the selected Azure Arc-enabled server in the Azure portal or run [`Get-AzVMExtensionImage`](/powershell/module/az.compute/get-azvmextensionimage). You can specify multiple extensions in a single upgrade request by providing both:

- A comma-separated list of extensions, defined by their publisher and type (separated by a period)
- The target version for each extension

You can review the version of installed VM extensions at any time by running the command [`Get-AzConnectedMachineExtension`](/powershell/module/az.connectedmachine/get-azconnectedmachineextension). The `TypeHandlerVersion` property value represents the version of the extension.

> [!TIP]
> Many VM extensions can be configured for [automatic upgrades](manage-automatic-vm-extension-upgrade.md).

## Remove extensions

To remove an installed VM extension on your Azure Arc-enabled server, use [`Remove-AzConnectedMachineExtension`](/powershell/module/az.connectedmachine/remove-azconnectedmachineextension) with the `-Name`, `-MachineName`, and `-ResourceGroupName` parameters.

## Related content

- Deploy, manage, and remove VM extensions by using the [Azure CLI](manage-vm-extensions-cli.md), the [Azure portal](manage-vm-extensions-portal.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).
- Find troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).
