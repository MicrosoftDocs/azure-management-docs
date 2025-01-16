---
title: Enable VM Extensions from the Azure Portal
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments from the Azure portal.
ms.date: 12/04/2024
ms.topic: how-to
---

# Enable Azure VM extensions from the Azure portal

This article shows you how to deploy, update, and uninstall Azure virtual machine (VM) extensions supported by Azure Arc-enabled servers. It shows you how to perform these tasks on a Linux or Windows hybrid machine by using the Azure portal.

> [!NOTE]
> The Azure Key Vault VM extension does not support deployment from the Azure portal. Use the Azure CLI, Azure PowerShell, or an Azure Resource Manager template to deploy this extension.
>
> Azure Arc-enabled servers doesn't support deploying and managing VM extensions to Azure virtual machines. For Azure VMs, see the [VM extension overview](/azure/virtual-machines/extensions/overview) article.

## Enable extensions

You can apply VM extensions to your machine managed through Azure Arc-enabled servers by using the Azure portal:

1. In your browser, go to the [Azure portal](https://portal.azure.com).

1. In the portal, go to **Machines - Azure Arc** and select your machine from the list.

1. Select **Settings** > **Extensions**, and then select **Add**.

1. Choose the extension that you want from the displayed extensions, or use the **Search** field to find the applicable extension. Then select **Next**.

    Depending on the extension that you selected, you might need to provide specific configuration information. For example, to deploy the Azure Monitor agent for Windows by using a proxy, you need to provide a proxy address and authentication information.

    :::image type="content" source="media/manage-vm-extensions/ama-extension-config.png" alt-text="Screenshot that shows configuration fields for the Azure Monitor agent extension.":::
  
1. After you provide the applicable configuration information, select **Review + create** to view a summary of the deployment. Then select **Create**.

> [!NOTE]
> Although multiple extensions can be batched and processed together, they're installed serially. After installation of the first extension is complete, the next extension is installed.

## List extensions installed

You can get a list of the VM extensions on your Azure Arc-enabled server from the Azure portal:

1. In your browser, go to the [Azure portal](https://portal.azure.com).

2. In the portal, go to **Machines - Azure Arc** and select your machine from the list.

3. Select **Settings** > **Extensions**. The list of installed extensions appears.

    :::image type="content" source="media/manage-vm-extensions/list-vm-extensions.png" alt-text="Screenshot that shows a list of virtual machine extensions deployed to a selected machine." border="true":::

## Upgrade extensions

When a new version of a supported extension is released, you can upgrade the extension to that latest release. When you go to Azure Arc-enabled servers in the Azure portal, a banner informs you that upgrades are available for one or more extensions installed on a machine.

When you view the list of installed extensions for a selected Azure Arc-enabled server, notice the column labeled **Update available**. If a newer version of an extension is released, the **Update available** value for that extension shows a value of **Yes**.

> [!NOTE]
> Although the Azure portal currently uses the word **Update** for this experience, that word does not accurately represent the behavior of the operation. Extensions are upgraded by installing a newer version of the extension that's currently installed on the machine or server.

Upgrading an extension to the newest version does not affect the configuration of that extension. You're not required to respecify configuration information for any extension that you upgrade.

:::image type="content" source="media/manage-vm-extensions-portal/vm-extensions-update-status.png" alt-text="Screenshot that shows a list of virtual machine extensions with their update status." border="true":::

You can upgrade one or multiple extensions that are eligible for an upgrade by performing the following steps in the Azure portal.

> [!NOTE]
> Currently, you can upgrade extensions only from the Azure portal. Performing this operation from the Azure CLI or an Azure Resource Manager template is not supported at this time.

1. In your browser, go to the [Azure portal](https://portal.azure.com).

2. In the portal, go to **Machines - Azure Arc** and select your hybrid machine from the list.

3. Select **Settings** > **Extensions**, and then review the status of extensions in the **Update available** column.

4. Perform the upgrade by using one of these methods:

   * Select an extension from the list of installed extensions. In the properties of the extension, select **Update**.

     :::image type="content" source="media/manage-vm-extensions-portal/vm-extensions-update-from-extension.png" alt-text="Screenshot that shows the properties of a selected extension and the Update button." border="true":::

   * Select the extension from the list of installed extensions. On the top of the page, select **Update**.

   * Select one or more extensions that are eligible for an upgrade from the list of installed extensions, and then select **Update**.

     :::image type="content" source="media/manage-vm-extensions-portal/vm-extensions-update-selected.png" alt-text="Screenshot that shows a selected extension in a list of extensions that are eligible for an upgrade." border="true":::

## Remove extensions

You can use the Azure portal to remove one or more extensions from an Azure Arc-enabled server:

1. In your browser, go to the [Azure portal](https://portal.azure.com).

2. In the portal, go to **Machines - Azure Arc** and select your hybrid machine from the list.

3. Select **Settings** > **Extensions**, and then select an extension from the list of installed extensions.

4. Select **Uninstall**. When you're prompted to verify, select **Yes** to proceed.

## Related content

* You can deploy, manage, and remove VM extensions by using the [Azure CLI](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).
* You can find troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).
