---
title: Automatic extension upgrade for Azure Arc-enabled servers
description: Learn how to enable automatic extension upgrades for your Azure Arc-enabled servers.
ms.topic: concept-article
ms.date: 05/09/2025
---

# Automatic extension upgrade for Azure Arc-enabled servers

You can enable automatic extension upgrade for Azure Arc-enabled servers that have supported [VM extensions](manage-vm-extensions.md) installed. Automatic extension upgrades reduce the amount of operational overhead by scheduling the installation of new extension versions when they become available. The Azure Connected Machine agent takes care of upgrading the extension, preserving its settings along the way.

## How does automatic extension upgrade work?

With automatic extension upgrade, existing extension versions deployed on your Arc-enabled servers are replaced with the newer one after the extension publisher provides a new version.

By default, all extensions are opted into the automatic upgrade feature. However, only those [extensions currently supported for this feature](#supported-extensions) will receive automatic upgrades. You can choose to opt out of automatic upgrades for each extension at any time.

### Availability-first updates

The availability-first model for platform-orchestrated updates ensures that availability configurations in Azure are respected across multiple availability levels.

For a group of Arc-enabled servers undergoing an update, the Azure platform orchestrates updates following the model described in the [Automation Extension Upgrade](/azure/virtual-machines/automatic-extension-upgrade#availability-first-updates). However, there are some notable differences between Arc-enabled servers and Azure VMs:

**Across regions:**

- Geo-paired regions aren't applicable.

**Within a region:**

- Availability zones aren't applicable.
- Machines are batched on a best effort basis to avoid concurrent updates for all machines registered with Arc-enabled servers in a subscription.

### Automatic rollback and retries

If an extension upgrade fails, Azure tries to repair the extension by performing the following actions:

1. The Azure Connected Machine agent will automatically reinstall the last known good version of the extension to attempt to restore functionality.
1. If the rollback is successful, the extension status shows **Succeeded**, and the extension will be added to the automatic upgrade queue again. The next upgrade attempt can be as soon as the next hour, and attempts will continue until the upgrade is successful.
1. If the rollback fails, the extension status shows **Failed**, and the extension will no longer function. In this case, you must [remove](manage-vm-extensions-cli.md#remove-extensions) and [reinstall](manage-vm-extensions-cli.md#enable-an-extension) the extension to restore functionality.

If you experience ongoing issues with an automatic extension upgrade, you can [disable automatic extension upgrade](#manage-automatic-extension-upgrade) to prevent the system from trying again while you troubleshoot the issue. You can then [enable automatic extension upgrade](#manage-automatic-extension-upgrade) again when you're ready.

### Extension upgrades with multiple extensions

A machine managed by Arc-enabled servers can have multiple extensions with automatic extension upgrade enabled. The same machine can also have other extensions without automatic extension upgrade enabled.

If multiple extension upgrades are available for a machine, the upgrades might be batched together, but each extension upgrade is applied individually. A failure on one extension doesn't impact upgrading any other extensions. For example, if two extensions are scheduled for an upgrade, and the first extension upgrade fails, the second extension will still be upgraded.

### Timing of automatic extension upgrades

When a new version of a VM extension is published, it becomes available for installation and manual upgrade on Arc-enabled servers. For servers that have that extension installed with automatic extension upgrade enabled, it could take up to eight weeks for every server with that extension to get the automatic upgrade. Upgrades are issued in batches across Azure regions and subscriptions, so you might see the extension get upgraded on some of your servers before others.

Extension versions fixing critical security vulnerabilities are rolled out much faster. These automatic upgrades happen using a specialized rollout process, where each server with that extension will be upgraded within three weeks. Azure determines which extension versions should be rolled out the fastest to help ensure servers are protected.

You can always upgrade any extension immediately by following the guidance to manually upgrade extensions using the [Azure portal](manage-vm-extensions-portal.md#upgrade-extensions), [Azure PowerShell](manage-vm-extensions-powershell.md#upgrade-extensions) or [Azure CLI](manage-vm-extensions-cli.md#upgrade-extensions).

## Supported extensions

Automatic extension upgrade supports the following extensions:

- Azure Monitor Agent - Linux and Windows
- Dependency agent – Linux and Windows
- Azure Security agent - Linux and Windows
- Key Vault Extension - Linux only
- Azure Update Manager - Linux and Windows
- Azure Automation Hybrid Runbook Worker - Linux and Windows
- Azure extension for SQL Server - Linux and Windows

Extensions that don't currently support automatic extension upgrade are still configured to enable automatic upgrades by default once the feature is available for those extensions. This setting has no effect until the extension publisher chooses to support automatic upgrades.

## Manage automatic extension upgrade

Automatic extension upgrade is enabled by default when you install extensions on Azure Arc-enabled servers. To enable automatic upgrades for an existing extension, you can use Azure CLI or Azure PowerShell to set the `enableAutomaticUpgrade` property on the extension to `true`. You'll need to repeat this process for every extension where you'd like to enable or disable automatic upgrades.

### [Azure portal](#tab/azure-portal)

Use the following steps to configure automatic extension upgrades in using the Azure portal:

1. In the [Azure portal](https://portal.azure.com), navigate to **Machines - Azure Arc**.
1. Select the applicable server.
1. In the service menu, under **Settings**, select **Extensions**

   :::image type="content" source="media/manage-automatic-vm-extension-upgrade/portal-navigation-extensions.png" alt-text="Screenshot of an Azure Arc-enabled server in the Azure portal showing where to navigate to extensions." border="true":::

1. The **Automatic upgrade** column in the table shows whether upgrades are enabled, disabled, or not supported for each extension. To turn on automatic upgrade for supported extensions,  select the checkbox next to the extensions and then select **Enable automatic upgrade** to turn on the feature. Likewise, to turn off automatic upgrade for certain extensions, select their checkboxes and then select **Disable automatic upgrade**.

### [Azure CLI](#tab/azure-cli)

To check the status of automatic extension upgrade for all extensions on an Arc-enabled server, run the following command:

```azurecli
az connectedmachine extension list --resource-group resourceGroupName --machine-name machineName --query "[].{Name:name, AutoUpgrade:properties.enableAutoUpgrade}" --output table
```

Use the [az connectedmachine extension update](/cli/azure/connectedmachine/extension) command to enable automatic upgrades on an extension:

```azurecli
az connectedmachine extension update \
    --resource-group resourceGroupName \
    --machine-name machineName \
    --name extensionName \
    --enable-auto-upgrade true
```

To disable automatic upgrades, set the `--enable-auto-upgrade` parameter to `false`, as shown below:

```azurecli
az connectedmachine extension update \
    --resource-group resourceGroupName \
    --machine-name machineName \
    --name extensionName \
    --enable-auto-upgrade false
```

### [Azure PowerShell](#tab/azure-powershell)

To check the status of automatic extension upgrade for all extensions on an Arc-enabled server, run the following command:

```azurepowershell
Get-AzConnectedMachineExtension -ResourceGroup resourceGroupName -MachineName machineName | Format-Table Name, EnableAutomaticUpgrade
```

To enable automatic upgrades for an extension using Azure PowerShell, use the [Update-AzConnectedMachineExtension](/powershell/module/az.connectedmachine/update-azconnectedmachineextension) cmdlet:

```azurepowershell
Update-AzConnectedMachineExtension -ResourceGroup resourceGroupName -MachineName machineName -Name extensionName -EnableAutomaticUpgrade
```

To disable automatic upgrades, set `-EnableAutomaticUpgrade:$false` as shown in the example below:

```azurepowershell
Update-AzConnectedMachineExtension -ResourceGroup resourceGroupName -MachineName machineName -Name extensionName -EnableAutomaticUpgrade:$false
```

> [!TIP]
> The cmdlets above come from the [Az.ConnectedMachine](/powershell/module/az.connectedmachine) PowerShell module. You can install this PowerShell module with `Install-Module Az.ConnectedMachine` on your computer or in Azure Cloud Shell.

---

## Check automatic extension upgrade history

You can use the Azure Activity Log to identify extensions that were automatically upgraded. You can find the Activity Log tab on individual Azure Arc-enabled server resources, resource groups, and subscriptions. Extension upgrades are identified by the `Upgrade Extensions on Azure Arc machines (Microsoft.HybridCompute/machines/upgradeExtensions/action)` operation.

To view automatic extension upgrade history, search for the **Azure Activity Log** in the Azure portal. Select **Add filter** and choose the Operation filter. For the filter criteria, search for "Upgrade Extensions on Azure Arc machines" and select that option. You can optionally add a second filter for **Event initiated by** and set "Azure Regional Service Manager" as the filter criteria to only see automatic upgrade attempts and exclude upgrades manually initiated by users.

:::image type="content" source="media/manage-automatic-vm-extension-upgrade/azure-activity-log-extension-upgrade.png" alt-text="Azure Activity Log showing attempts to automatically upgrade extensions on Azure Arc-enabled servers." border="true":::

## Next steps

- You can deploy, manage, and remove VM extensions using the [Azure CLI](manage-vm-extensions-cli.md), [PowerShell](manage-vm-extensions-powershell.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).

- Troubleshooting information can be found in the [Troubleshoot VM extensions guide](troubleshoot-vm-extensions.md).
