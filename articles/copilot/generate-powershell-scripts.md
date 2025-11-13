---
title: Generate PowerShell scripts using Azure Copilot
description: Learn about scenarios where Azure Copilot can generate PowerShell scripts for you to customize and use.
ms.date: 04/08/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: "As a cloud developer, I want to generate PowerShell scripts using an AI assistant, so that I can automate resource management tasks in Azure more efficiently."
---

# Generate PowerShell scripts using Azure Copilot

Azure Copilot can generate [PowerShell](/powershell/azure/) scripts that you can use to create or manage resources.

When you tell Azure Copilot about a task you want to perform by using PowerShell, it provides a script with the necessary cmdlets. You'll see which placeholder values that you need to update with the actual values based on your environment.

> [!TIP]
> You can also get help from Azure Copilot [directly from your command-line interface](ai-shell-overview.md).

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to generate PowerShell scripts. Some prompts return a single cmdlet, while others provide multiple steps walking through the full scenario. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "How do I list the VMs I have running in Azure using PowerShell?"
- "Create a storage account using PowerShell."
- "How do I get all quota limits for a subscription using Azure PowerShell?"
- "Can you show me how to stop all virtual machines in a specific resource group using PowerShell?"

## Examples

In this example, the prompt "**How do I list all my resource groups using PowerShell?**" provides the cmdlet. You can ask follow-up questions to get more information.

:::image type="content" source="media/generate-powershell-scripts/powershell-list-resource-groups.png" alt-text="Screenshot of Azure Copilot providing the PowerShell cmdlet to list resource groups.":::

Similarly, if you ask "**How can I create a new resource group using PowerShell?**", you see an example cmdlet that you can customize as needed.

:::image type="content" source="media/generate-powershell-scripts/powershell-create-resource-group.png" alt-text="Screenshot of Azure Copilot providing the PowerShell cmdlet to create a new resource group.":::

You can also ask Azure Copilot for a script with multiple cmdlets. For example, you could say "**Can you help me write a script for Azure PowerShell that can be run directly, and after creating a VM, deploy an AKS cluster on it.**" Azure Copilot provides a code block that you can copy, letting you know which values to replace.

:::image type="content" source="media/generate-powershell-scripts/powershell-script.png" alt-text="Screenshot of Azure Copilot providing a PowerShell script that creates a VM and deploys an AKS cluster.":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure PowerShell](/powershell/azure/).
