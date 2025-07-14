---
title: Troubleshoot Arc machine extensions using Microsoft Copilot in Azure
description: Learn about scenarios where Microsoft Copilot in Azure can help troubleshoot extension issues on Arc machines
ms.date: 04/08/2025
ms.service: copilot-for-azure
ms.topic: how-to
ms.author: jenhayes
author: JnHs
# Customer intent: As an IT administrator managing Azure Arc-enabled servers, I want to troubleshoot extension failures using a guided tool, so that I can quickly identify issues and apply recommended fixes to maintain system functionality.
---

# Troubleshoot extension failures for Azure Arc-enabled servers using Microsoft Copilot in Azure

Microsoft Copilot in Azure can help you troubleshoot issues with extensions deployed on Azure Arc-enabled servers. [Extensions](/azure/azure-arc/servers/manage-vm-extensions) are small applications that provide post-deployment configuration and automation tasks, like installation of software. Many of the Azure management services available for Arc-enabled servers require extensions.

When you ask Microsoft Copilot in Azure questions about troubleshooting extensions for Arc-enabled servers, it prompts you to select the Arc-enabled server and the extension that you are interested in troubleshooting. Copilot in Azure analyzes your current configuration and associated logs, then provides a summary of the issue and recommended remediation steps. When possible, Copilot in Azure will assist you with performing actions to help remediate the issue, such as reinstalling the extension.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

- "Help me troubleshoot my failed extension on my arc-server"
- "Why is my extension on my Arc machine in a failed state?"

## Examples

When you ask Microsoft Copilot in Azure **"Help me troubleshoot my arc server extension"**, Copilot runs an analysis of your extension state and error logs. First, it asks you to confirm the affected machine:

:::image type="content" source="media/troubleshoot-arc-extension/troubleshoot-arc-extension-machine.png" alt-text="Screenshot of Microsoft Copilot in Azure prompting to select an Arc-enabled server for troubleshooting.":::

If there are multiple extensions in a failed state, you'll be prompted to select the one you're interested in. Copilot in Azure then shows details about the error message and recommended troubleshooting steps.

:::image type="content" source="media/troubleshoot-arc-extension/troubleshoot-arc-extension-error.png" alt-text="Screenshot of Microsoft Copilot in Azure providing details about an Arc extension error." lightbox="media/troubleshoot-arc-extension/troubleshoot-arc-extension-error.png":::

In some cases, reinstalling the extension may be needed to fix the issue. When possible, Copilot in Azure will offer to help you reinstall it.

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn [how to use extensions on Arc-enabled servers](/azure/azure-arc/servers/manage-vm-extensions).
- Learn more about [troubleshooting extensions on Arc-enabled servers](/azure/azure-arc/servers/troubleshoot-vm-extensions).
