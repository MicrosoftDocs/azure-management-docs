---
title: Troubleshoot Arc machine extensions using Microsoft Copilot in Azure (preview)
description: Learn about scenarios where Microsoft Copilot in Azure can help troubleshoot extension issues on Arc machines
ms.date: 03/05/2025
ms.topic: how-to
---

# Troubleshoot extension failures for Aazure Arc-enabled servers using Microsoft Copilot in Azure

Microsoft Copilot in Azure (preview) can help you troubleshoot issues with extensions deployed on Azure Arc-enabled servers. [Extensions](/azure/azure-arc/servers/manage-vm-extensions) are small applications that provide post-deployment configuration and automation tasks, like installation of software. Many of the Azure management services available for Arc-enabled serversrequire extensions.

When you ask Microsoft Copilot in Azure questions about troubleshooting extensions for Arc-enabled servers, it prompts you to select the Arc-enabled server and the extension that you are interested in troubleshooting. Using this information, Copilot in Azure analyses your current configuration and associated logs. It then provides a summary of the issue and recommended remediation steps. In some cases, Copilot in Azure will assist you with performing actions, like reinstallation of the extension to help remediate the issue. 

[!INCLUDE [scenario-note](includes/scenario-note.md)]

[!INCLUDE [preview-note](includes/preview-note.md)]

## Sample prompts

- "Help me troubleshoot my failed extension on my arc-server"
- "Why is my extension on my Arc machine in a failed state?"

## Examples

When you ask Microsoft Copilot in Azure **"Help me troubleshoot my arc server extension"**, Copilot runs an analysis of your extension state and error logs. First, it asks you to confirm the affected machine:

:::image type="content" source="media/troubleshoot-arc-extensions/troubleshoot-arc-extension-machine.png" alt-text="Screenshot of Microsoft Copilot in Azure prompting to select an Arc-enabled server for troubleshooting.":::

If there are multiple extensions in a failed state, you'll be prompted to select the one you're interested in. Copilot in Azure then shows details about the error message and recommended troubleshooting steps.

:::image type="content" source="media/troubleshoot-arc-extensions/troubleshoot-arc-extension-error.png" alt-text="Screenshot of Microsoft Copilot in Azure providing details about an Arc extension error." lightbox="media/troubleshoot-arc-extensions/troubleshoot-arc-extension-error.png":::

In some cases, Copilot may be able to help you reinstall the extension.

:::image type="content" source="media/troubleshoot-arc-extensions/troubleshoot-arc-extension-reinstall.png" alt-text="Screenshot of Microsoft Copilot in Azure prompting to reinstall":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn [how to use extensions on Arc-enabled servers](/azure/azure-arc/servers/manage-vm-extensions).
- Learn more about [troubleshooting extensions on Arc-enabled servers](/azure/azure-arc/servers/troubleshoot-vm-extensions).
