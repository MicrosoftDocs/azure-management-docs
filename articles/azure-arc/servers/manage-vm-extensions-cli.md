---
title: Enable VM Extensions Using the Azure CLI (Windows and Linux)
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments by using the Azure CLI.
ms.date: 06/19/2025
ms.topic: how-to
ms.custom:
  - devx-track-azurecli
  - build-2025
# Customer intent: As a system administrator managing hybrid cloud environments, I want to deploy and manage VM extensions using Azure CLI, so that I can efficiently automate tasks and maintain consistency across my Azure Arc-enabled servers.
---

# Enable Azure VM extensions by using the Azure CLI (Windows and Linux)

This article explains how to deploy, upgrade, update, and uninstall [virtual machine (VM) extensions](manage-vm-extensions.md) on Azure Arc-enabled servers by using the Azure CLI (Windows and Linux).

[!INCLUDE [Azure CLI Prepare your environment](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

## Install the Connected Machine extension in the Azure CLI

The `ConnectedMachine` commands aren't shipped as part of the Azure CLI. Before you use the Azure CLI to connect to Azure and manage VM extensions on your hybrid server managed by Azure Arc-enabled servers, you need to load the `ConnectedMachine` extension.

You can perform these management operations from your workstation, rather than on the Azure Arc-enabled server.

Run the following command to install the Azure CLI `ConnectedMachine` extension:

```azurecli
az extension add --name connectedmachine
```

## Enable an extension

To enable a VM extension on your Azure Arc-enabled server, use [`az connectedmachine extension create`](/cli/azure/connectedmachine/extension#az-connectedmachine-extension-create) with the `--machine-name`, `--extension-name`, `--location`, `--type`, `settings`, and `--publisher` parameters.

This example enables the Custom Script Extension on an Azure Arc-enabled server:

```azurecli
az connectedmachine extension create --machine-name "myMachineName" --name "CustomScriptExtension" --location "regionName" --type "CustomScriptExtension" --publisher "Microsoft.Compute" --settings "{\"commandToExecute\":\"powershell.exe -c \\\"Get-Process | Where-Object { $_.CPU -gt 10000 }\\\"\"}" --type-handler-version "1.10" --resource-group "myResourceGroup"
```

This example enables the Azure Key Vault VM extension on an Azure Arc-enabled server:

```azurecli
az connectedmachine extension create --resource-group "resourceGroupName" --machine-name "myMachineName" --location "regionName" --publisher "Microsoft.Azure.KeyVault" --type "KeyVaultForLinux or KeyVaultForWindows" --name "KeyVaultForLinux or KeyVaultForWindows" --settings '{"secretsManagementSettings": { "pollingIntervalInS": "60", "observedCertificates": ["observedCert1"] }, "authenticationSettings": { "msiEndpoint": "http://localhost:40342/metadata/identity" }}'
```

This example enables the Microsoft Antimalware extension on an Azure Arc-enabled Windows server:

```azurecli
az connectedmachine extension create --resource-group "resourceGroupName" --machine-name "myMachineName" --location "regionName" --publisher "Microsoft.Azure.Security" --type "IaaSAntimalware" --name "IaaSAntimalware" --settings '"{\"AntimalwareEnabled\": \"true\"}"'
```

This example enables the Datadog extension on an Azure Arc-enabled Windows server:

```azurecli
az connectedmachine extension create --resource-group "resourceGroupName" --machine-name "myMachineName" --location "regionName" --publisher "Datadog.Agent" --type "DatadogWindowsAgent" --settings '{"site": "us3.datadoghq.com"}' --protected-settings '{"api_key": "YourDatadogAPIKey" }'
```

> [!TIP]
> Many other extensions are supported on Arc-enabled servers. For details, see [Virtual machine extension management with Azure Arc-enabled servers](manage-vm-extensions.md#extensions).

## List extensions installed

To get a list of VM extensions on your Azure Arc-enabled server, use [`az connectedmachine extension list`](/cli/azure/connectedmachine/extension#az-connectedmachine-extension-list) with the `--machine-name` and `--resource-group` parameters:

```azurecli
az connectedmachine extension list --machine-name "myMachineName" --resource-group "myResourceGroup"
```

By default, the output of Azure CLI commands is in JSON (JavaScript Object Notation). To change the default output to a list or table, for example, use [az config set core.output=table](/cli/azure/reference-index). You can also add `--output` to any command for a one-time change in output format.

The following example shows the partial JSON output from the `az connectedmachine extension -list` command:

```json
[
  {
    "autoUpgradingMinorVersion": "false",
    "forceUpdateTag": null,
    "id": "/subscriptions/subscriptionId/resourceGroups/resourceGroupName/providers/Microsoft.HybridCompute/machines/SVR01/extensions/DependencyAgentWindows",
    "location": "regionName",
    "name": "DependencyAgentWindows",
    "namePropertiesInstanceViewName": "DependencyAgentWindows",
```

## Update an extension configuration

Some VM extensions require configuration settings so that you can install them on an Azure Arc-enabled server (like the Custom Script Extension). To upgrade the configuration of an extension, use [`az connectedmachine extension update`](/cli/azure/connectedmachine/extension#az-connectedmachine-extension-update).

The following example shows how to configure the Custom Script Extension:

```azurecli
az connectedmachine extension update --name "CustomScriptExtension" --type "CustomScriptExtension" --publisher "Microsoft.HybridCompute" --settings "{\"commandToExecute\":\"powershell.exe -c \\\"Get-Process | Where-Object { $_.CPU -lt 100 }\\\"\"}" --type-handler-version "1.10" --machine-name "myMachine" --resource-group "myResourceGroup"
```

## Upgrade extensions

When a new version of a supported VM extension is released, you can upgrade it to that latest release. To upgrade a VM extension, use [`az connectedmachine upgrade-extension`](/cli/azure/connectedmachine) with the `--machine-name`, `--resource-group`, and `--extension-targets` parameters.

For the `--extension-targets` parameter, you need to specify the extension and the latest version available. To determine the latest version available for an extension, go to the **Extensions** page for the selected Azure Arc-enabled server in the Azure portal or run [az vm extension image list](/cli/azure/vm/extension/image#az-vm-extension-image-list). You can specify multiple extensions in a single upgrade request by providing both:

- A comma-separated list of extensions, defined by their publisher and type (separated by a period)
- The target version for each extension

You can review the version of installed VM extensions at any time by running the command [`az connectedmachine extension list`](/cli/azure/connectedmachine/extension#az-connectedmachine-extension-list). The `typeHandlerVersion` property value represents the version of the extension.

> [!TIP]
> Many VM extensions can be configured for [automatic upgrades](manage-automatic-vm-extension-upgrade.md).

## Remove extensions

To remove an installed VM extension from your Azure Arc-enabled server, use [`az connectedmachine extension delete`](/cli/azure/connectedmachine/extension#az-connectedmachine-extension-delete) with the `--extension-name`, `--machine-name`, and `--resource-group` parameters.

## Related content

- Deploy, manage, and remove VM extensions by using [Azure PowerShell](manage-vm-extensions-powershell.md), the [Azure portal](manage-vm-extensions-portal.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).
- Find troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).
- For more information about the commands, review the [overview of the Azure CLI VM extension](/cli/azure/connectedmachine/extension).
