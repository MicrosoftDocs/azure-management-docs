---
title: Optimization agent capabilities in Agents (preview) in Azure Copilot
description: Agents (preview) in Azure Copilot lets you complete optimization tasks using intelligent agent capabilities.
author: JnHs
ms.author: jenhayes
ms.date: 11/18/2025
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use agents to help with optimization tasks, so that I can be more successful performing tasks in  my Azure tenant."
---

# Optimization capabilities in Agents (preview) in Azure Copilot

[Agents (preview) in Azure Copilot](agents-preview.md) intelligently surfaces the right agent to help with your tasks. When you ask Azure Copilot for help with optimization tasks, you can get help taking actions to reduce costs and carbon emissions for your Azure resources while maintaining performance.

> [!IMPORTANT]
> The functionality described in this article is only available for tenants that have access to [Agents (preview) in Azure Copilot](agents-preview.md).

Azure Copilot provides detailed optimization recommendations and helps you understand the reasons behind them. It can provide alternative options for you to compare. To help you implement the recommended changes, Azure Copilot can generate Azure CLI or PowerShell scripts that you can deploy.

To help you understand and act on optimization, Azure Copilot can generate charts that show the expected results of applying recommendations.

:::image type="content" source="media/optimization-agent/azure-copilot-optimization-recommendation.png" alt-text="Screenshot of Azure Copilot providing an optimization recommendation.":::

:::image type="content" source="media/optimization-agent/azure-copilot-optimization-recommendation-chart.png" alt-text="Screenshot of Azure Copilot generating a chart to show expected results of applying an optimization recommendation.":::

## Supported resource types

Currently, Agents (preview) in Azure Copilot supports optimization recommendations for the following resource types:

- Virtual machines (VMs)
- Virtual Machine Scale Sets (VMSS)

## Use optimization agent capabilities

You can get started with optimization recommendations in several ways:

- From the Azure Copilot chat pane, ask for optimization recommendations for your subscription or specific resources. For the best results, specify the subscription GUID or ARM Resource URI in your prompt, rather than the resource names, unless the resource is clear from the current context. Be sure to enable agent mode by selecting the icon in the chat window.
- In [operations center (preview)](/azure/operations/overview), top recommendations for your subscriptions are shown. Select **Optimize with Copilot** next to a recommendation to explore further.
- From the [Azure Advisor cost recommendations page](/azure/advisor/advisor-reference-cost-recommendations) or [operations center (preview)](/azure/operations/overview)recommendations page, select **Optimize** next to an impacted VM or VMSS resource.

## Optimization sample prompts

Here are a few examples of the kinds of prompts you can use to get help with optimization. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. For the best results, specify the subscription GUID or ARM Resource URI in your prompt. Once you're already working on a subscription or resource, you can refer to it by name in further questions.

When using these kinds of prompts, be sure to enable agent mode by selecting the icon in the chat window.

:::image type="content" source="media/agent-preview/azure-copilot-agent-mode-active.png" alt-text="Screenshot showing agent mode (preview) enabled in Azure Copilot.":::

- "Show me the top five cost-saving opportunities for subscription `aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e`."
- "Show me cost saving recommendations for `subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/acss-demo/providers/microsoft.compute/virtualmachines/ContosoVM1`."
- "Explain the recommendation for ContosoVM1."
- "Is there an alternate recommendation for ContosoVM1?"
- "Generate a PowerShell script to apply the recommended optimizations for ContosoVM1."
- "Can you provide a CLI script to apply those optimizations?"
- "Summarize total potential cost and carbon reduction from all active recommendations."

## Current considerations and limitations

Keep in mind the following considerations and limitations when asking for optimization recommendations with Agents (preview) in Azure Copilot.

- Azure Copilot can generate scripts that you can run to optimize your resources, but it can't execute any actions to change resources.
- Azure Copilot can't automatically create or update budgets and alerts.
- By default, when asking for recommendations across a subscription, the response will show five items. You can ask for a specific number of top recommendations, with a maximum of ten items per query.
- If the context isn't already clear, you may need to provide the subscription GUID or ARM Resource URI for the resource you want to optimize, rather than the resource name.
