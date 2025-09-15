---
title: Enable VM extensions to Arc-enabled servers from the Azure portal
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments from the Azure portal.
ms.date: 06/19/2025
ms.topic: how-to
ms.custom:
  - build-2025
  - sfi-image-nochange
# Customer intent: "As a system administrator managing hybrid environments, I want to deploy and manage VM extensions on Azure Arc-enabled servers via the Azure portal, so that I can efficiently enhance server functionality without needing command-line tools."
---

# Enable Azure VM extensions to Arc-enabled servers from the Azure portal

You can deploy, update, and uninstall [virtual machine (VM) extensions](manage-vm-extensions.md) supported by Azure Arc-enabled servers. This article shows you how to perform these tasks on a Linux or Windows hybrid machine by using the Azure portal.

> [!NOTE]
> Some extensions, such as the Azure Key Vault VM extension, don't support deployment from the Azure portal. Use the [Azure CLI](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), or an [Azure Resource Manager template](manage-vm-extensions-template.md) to deploy this extension.

## Enable extensions

You can apply VM extensions to your machine managed through Azure Arc-enabled servers by using the Azure portal:

1. In your browser, go to the [Azure portal](https://portal.azure.com).

1. In the Azure portal, go to **Machines - Azure Arc** and select your machine.

1. Under **Settings**, select **Extensions**, and then select **Add**.

1. Choose the extension that you want from the displayed extensions, or use the **Search** field to find the applicable extension. Then select **Next**.

    Depending on the extension that you selected and your requirements, you might need to provide specific configuration information. For example, to deploy the Azure Monitor agent for Windows by using a proxy, you need to provide a proxy address and authentication information.

    :::image type="content" source="media/manage-vm-extensions-portal/ama-extension-config.png" alt-text="Screenshot that shows configuration fields for the Azure Monitor agent extension.":::
  
1. After you provide the applicable configuration information, select **Review + create** to view a summary of the deployment. Then select **Create**.

> [!NOTE]
> Although multiple extensions can be batched and processed together, they're installed serially. After installation of the first extension is complete, the next extension is installed.

## List extensions installed

You can get a list of the VM extensions on your Azure Arc-enabled server from the Azure portal:

1. In the Azure portal, go to **Machines - Azure Arc** and select your machine.

1. Under **Settings**, select **Extensions**. The list of installed extensions appears.

   :::image type="content" source="media/manage-vm-extensions-portal/list-vm-extensions.png" alt-text="Screenshot showing the extensions deployed to a selected machine." lightbox="media/manage-vm-extensions-portal/list-vm-extensions.png":::

## Upgrade extensions

When a new version of a supported extension is released, you can upgrade the extension to that latest release. Extensions are upgraded by installing a newer version of the extension than the one currently installed on the machine, rather than deploying an update to the current version.

> [!TIP]
> Many VM extensions can be configured for [automatic upgrades](manage-automatic-vm-extension-upgrade.md).

When viewing an Arc-enabled server in the Azure portal, a banner appears if upgrades are available for one or more of the installed extensions.

When you view the list of installed extensions for a selected Azure Arc-enabled server, notice the column labeled **Update available**. If a newer version of an extension is released, the **Update available** value for that extension shows a value of **Yes**, along with the new version number.

Upgrading an extension to the newest version doesn't affect the configuration of that extension. When you upgrade an extension, you don't have to respecify configuration information.

You can upgrade one or multiple extensions that are eligible for an upgrade by performing the following steps in the Azure portal.

1. In the Azure portal, go to **Machines - Azure Arc** and select your machine.

1. Under **Settings**, select **Extensions**, and then review the status of extensions in the **Update available** column.

1. Perform the upgrade by using one of these methods:

   * Select an extension from the list of installed extensions. In the properties of the extension, select **Update**.

     :::image type="content" source="media/manage-vm-extensions-portal/vm-extensions-update-from-extension.png" alt-text="Screenshot showing the properties of a selected extension and the Update button.":::

   * Select the extension from the list of installed extensions. On the top of the page, select **Update**.

   * Select one or more extensions that are eligible for an upgrade from the list of installed extensions, and then select **Update**.

     :::image type="content" source="media/manage-vm-extensions-portal/vm-extensions-update-selected.png" alt-text="Screenshot showing a selected extension that is eligible for an upgrade." lightbox="media/manage-vm-extensions-portal/vm-extensions-update-selected.png":::

## Remove extensions

You can use the Azure portal to remove one or more extensions from an Azure Arc-enabled server:

1. In the Azure portal, go to **Machines - Azure Arc** and select your machine.

1. Under **Settings**, select **Extensions**, and then select an extension from the list of installed extensions.

1. Select **Uninstall**, then select **Yes** to proceed.

## Related content

* Learn how to deploy, manage, and remove VM extensions by using the [Azure CLI](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).
* Find troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).
