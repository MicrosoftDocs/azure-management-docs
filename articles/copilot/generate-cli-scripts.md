---
title: Generate Azure CLI scripts using Azure Copilot
description: Learn about scenarios where Azure Copilot can generate Azure CLI scripts for you to customize and use.
ms.date: 11/20/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom: ignite-2023, ignite-2023-copilotinAzure, devx-track-azurecli, build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: "As a cloud user, I want to generate Azure CLI scripts using an AI assistant, so that I can easily create and manage resources without manual coding."
---

# Generate Azure CLI scripts using Azure Copilot

Azure Copilot can generate [Azure CLI](/cli/azure/) scripts that you can use to create or manage resources.

When you tell Azure Copilot about a task you want to perform by using Azure CLI, it provides a script with the necessary commands. You'll see which placeholder values that you need to update with the actual values based on your environment.

> [!TIP]
> You can also get help from Azure Copilot [directly from your command-line interface](ai-shell-overview.md).

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to generate Azure CLI scripts. Some prompts will return a single command, while others provide multiple steps walking through the full scenario. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Give me a CLI script to create a new storage account"
- "How do I list all my VMs using Azure CLI?"
- "Create a virtual network with two subnets using the address space of 10.0.0.0/16 using az cli"
- "I need to assign a dns name to a vm using a script"
- "How to attach a disk to a VM using az cli"
- "How to create and manage a Linux pool in Azure Batch using cli"
- "Show me how to back up and restore a web app from a backup using cli"
- "Create VNet service endpoints for Azure Database for PostgreSQL using CLI"
- "I want to create a function app with a named storage account connection using Azure CLI"
- "How to create an App Service app and deploy code to a staging environment using CLI"
- "I want to use Azure CLI to deploy and manage AKS using a private service endpoint."

## Examples

In this example, the prompt "**I want to use Azure CLI to create a new storage account**" provides an example command, along with a brief explanation of the parameters that you need to update. Alternately, you can specify these values directly in your prompt to get a more customized command.

:::image type="content" source="media/generate-cli-scripts/cli-create-storage-account.png" alt-text="Screenshot of Azure Copilot providing Azure CLI commands to create a storage account.":::

For tasks that require multiple commands, Azure Copilot can provide a script that you can copy and paste. For example, you can say "**Give me an Azure CLI script to create a function app with a named storage account connection.**" Azure Copilot provides a full script with comments for each section, along with a brief explanation of what the script does.

:::image type="content" source="media/generate-cli-scripts/cli-create-function-app.png" alt-text="Screenshot of Azure Copilot providing an Azure CLI script to create a function app with a connected storage account.":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure CLI](/azure/cli).
