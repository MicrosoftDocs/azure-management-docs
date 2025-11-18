---
title: Troubleshooting agent capabilities in Agents (preview) in Azure Copilot
description: Agents (preview) in Azure Copilot lets you get troubleshooting help using intelligent agent capabilities.
author: JnHs
ms.author: jenhayes
ms.date: 11/18/2025
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use agents to help with troubleshooting, so that I can be more successful performing tasks in  my Azure tenant."
---

# Troubleshooting capabilities in Agents (preview) in Azure Copilot

[Agents (preview) in Azure Copilot](agents-preview.md) intelligently surfaces the right agent to help with your tasks. When you ask Azure Copilot for help with troubleshooting tasks, the agent functionality helps you resolve issues faster. You can quickly get help with diagnosing issues, finding solutions, and resolving problems in your Azure environment. Where possible, Azure Copilot analyzes your specific environment to run root cause diagnostics. Once the root cause is identified, Azure Copilot determines the appropriate mitigation steps, providing tailored solutions and step-by-step instructions. In many cases, Azure Copilot can even provide a one-click fix to resolve the issue for you.

If Azure Copilot can't resolve your issue, it can create a support request for you, gathering all the necessary details to help Microsoft Support assist you more effectively. You can review and confirm the details before the request is submitted.

> [!IMPORTANT]
> The functionality described in this article is only available for tenants that have access to [Agents (preview) in Azure Copilot](agents-preview.md).

## Supported resource types

Currently, Agents (preview) in Azure Copilot supports troubleshooting for all Azure resource types, and is often particularly useful for resolving issues with the following resource types:

- Azure Cosmos DB
- Azure Virtual Machines
- Azure Kubernetes Service (AKS)

## Troubleshooting sample prompts

Here are a few examples of the kinds of prompts you can use to get help with troubleshooting tasks. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. If you're not already working in the context of a resource, you may need to provide the specific resource that you want to troubleshoot.

When using these kinds of prompts, be sure to enable agent mode by selecting the icon in the chat window.

:::image type="content" source="media/agent-preview/azure-copilot-agent-mode-active.png" alt-text="Screenshot showing agent mode (preview) enabled in Azure Copilot.":::

- "Help me troubleshoot why my Cosmos DB Cassandra API is failing."
- "I’m trying to connect to Azure Cosmos DB Cassandra API from my local development machine, but I keep getting a timeout. What should I do?"
- "Help me investigate why my VM is unhealthy."
- "I can't connect to my VM, can you help me troubleshoot?"
- "Investigate the health of my pods."
- "Investigate networking issues causing pod connectivity failures."
- "Identify reasons for high CPU or memory usage in my AKS cluster."
- "Create a support request."
- "Open a support ticket for my problem"

## Current considerations and limitations

Keep in mind the following considerations and limitations when working with troubleshooting in Agents (preview) in Azure Copilot.

- Automatic mitigation of common issues isn't available for all issues or resource types. For example, one-click fixes aren't currently available to resolve issues with AKS. In these cases, Azure Copilot provides detailed instructions to help you resolve the issue, or it can create a support request for further investigation.
- Troubleshooting capabilities are based on currently available diagnostic data and predefined checks.
