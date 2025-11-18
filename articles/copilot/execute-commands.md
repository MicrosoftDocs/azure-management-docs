---
title: Execute commands using Azure Copilot
description: Learn about scenarios where Azure Copilot can help you perform tasks.
ms.date: 11/18/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
# Customer intent: As an Azure user, I want to execute commands using natural language prompts, so that I can efficiently manage my resources without manual navigation in the portal.
---

# Execute commands using Azure Copilot

Azure Copilot can help you execute individual or bulk commands on your resources. With Azure Copilot, you can save time by prompting Azure Copilot with natural language, rather than manually navigating to a resource and selecting a button in a resource's command bar.

For example, you can restart your virtual machines by using prompts like **"Restart my VM named ContosoDemo"** or **"Stop my VMs in West US 2."** Azure Copilot infers relevant resources inferred through an Azure Resource Graph query and determines the relevant command. Next, it asks you to confirm the action. Commands are never executed without your explicit confirmation. After the command is executed, you can track progress in the notification pane, just as if you manually ran the command from within the Azure portal. For faster responses, specify the resource ID of the resources that you want to run the command on.

Azure Copilot can execute many common commands on your behalf, as long as you have the permissions to perform them yourself. If Azure Copilot is unable to run a command for you, it generally provides instructions to help you perform the task yourself. To learn more about the commands you can execute with natural language for a resource or service, you can ask Azure Copilot directly. For instance, you can say **"Which commands can you help me perform on virtual machines?"**

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to execute commands. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Restart my VM named ContosoDemo"
- "Stop VMs in Europe regions
- "Restore my deleted storage account
- "Enable backup on VM named ContosoDemo"
- "Restart my web app named ContosoWebApp"
- "Start my AKS cluster"

## Examples

When you say **"Restore my deleted storage account**", Azure Copilot launches the **Restored deleted account** experience. From here, you can select the subscription and the storage account that you want to recover.

:::image type="content" source="media/execute-commands/restore-deleted-account.png" alt-text="Screenshot of Azure Copilot responding to a request to restore a deleted storage account." lightbox="media/execute-commands/restore-deleted-account.png":::

If you say **"Find the VMs running right now and stop them"**, Azure Copilot first queries to find all VMs running in your selected subscriptions. It then shows you the results and asks you to confirm that the selected VMs should be stopped. You can uncheck a box to exclude a resource from the command. After you confirm, the command is run, with progress shown in your notifications.

:::image type="content" source="media/execute-commands/stop-running-vms.png" alt-text="Screenshot showing Azure Copilot responding to a request to stop running VMs.":::

You can also specify the resource name in your prompt. When you say things like **"Restart my VM named SIMPLEVM**", Azure Copilot looks for that resource, then prompts you to confirm the operation.

:::image type="content" source="media/execute-commands/restart-vm.png" alt-text="Screenshot of Azure Copilot responding to a request to restart a VM.":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- [Get tips for writing effective prompts](write-effective-prompts.md) to use with Azure Copilot.
- Learn how to use Azure Copilot with [AI Shell](ai-shell-overview.md). 
