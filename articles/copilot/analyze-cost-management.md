---
title: Analyze, estimate and optimize cloud costs using Microsoft Copilot in Azure
description: Learn about scenarios where Microsoft Copilot in Azure can use Microsoft Cost Management to help you manage your costs.
ms.date: 11/14/2024
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
---

# Analyze, estimate and optimize cloud costs using Microsoft Copilot in Azure

Microsoft Copilot in Azure (preview) can help you analyze, estimate and optimize cloud costs. Ask questions using natural language to get information and recommendations based on [Microsoft Cost Management](/azure/cost-management-billing/costs/overview-cost-management).

When you ask Microsoft Copilot in Azure  questions about your costs, it automatically pulls context based on the last scope that you accessed using Cost Management. If the context isn't clear, you'll be prompted to select a scope or provide more context.

For OpenAI token-based models, Microsoft Copilot in Azure can provide simulations that estimate your costs for increasing or decreasing usage. You can also get estimates for changes to your OpenAI-token based models, such as moving from GPT-35-Turbo to GPT-4.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

[!INCLUDE [preview-note](includes/preview-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use for Cost Management. Modify these prompts based on your real-life scenarios, or try additional prompts to meet your needs.

- "Summarize my cost for the last 6 months"
- "What are the top services contributing to my cost?"
- "Can you provide last month's costs broken-down by service and region?"
- "Can you provide an estimate of our expected expenses for the next 3 months?"
- "How can we reduce our costs?"
- "How much would the cost change if GPT-35-Turbo usage increased by 15%?"
- "What would be the financial effect of transitioning from GPT-35-Turbo to GPT-4?"

## Examples

When you prompt Microsoft Copilot in Azure, "**Summarize my costs for the last 6 months**," a summary of costs for the selected scope is provided. You can follow up with questions to get more granular details, such as "What was our virtual machine spending last month?"

:::image type="content" source="media/analyze-cost-management/cost-management-summarize-costs.png" alt-text="Screenshot of Microsoft Copilot in Azure providing a summary of costs.":::

:::image type="content" source="media/analyze-cost-management/cost-management-vm-costs.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing details about VM costs.":::

Next, you can ask "**How can we reduce our costs?**" Microsoft Copilot in Azure provides a list of recommendations you can take, including an estimate of the potential savings you might see.

:::image type="content" source="media/analyze-cost-management/cost-management-reduce.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing a list of recommendations to reduce costs.":::

:::image type="content" source="media/analyze-cost-management/cost-management-reduce-2.png" alt-text="Screenshot showing Microsoft Copilot in Azure continuing a list of recommendations to reduce costs.":::

To better understand cost details, you can say "**Can you break down my October costs by Meter and Product?"

:::image type="content" source="media/analyze-cost-management/cost-management-understand.png" alt-text="Screenshot showing Microsoft Copilot in Azure showing details about charges grouped by Meter and Product.":::

For a detailed look at month-to-month changes, say "**Can you compare how my costs have changed from September to October by service?"

:::image type="content" source="media/analyze-cost-management/cost-management-compare.png" alt-text="Screenshot showing Microsoft Copilot in Azure comparing costs by service from September to October 2024.
":::

You can also get an estimate of upcoming expenses by saying "**Can you forecast my cost by month for the next 3 months?"

:::image type="content" source="media/analyze-cost-management/cost-management-forecast.png" alt-text="Screenshot showing Microsoft Copilot in Azure forecsting costs for a subscription for the next three months.":::

For OpenAI token-based models, you can ask questions to estimate costs for usage changes. For example, you can say "**How much would the cost change if GPT-35-Turbo usage increased by 15?**"

:::image type="content" source="media/analyze-cost-management/cost-management-simulation-usage.png" alt-text="Screenshot showing Microsoft Copilot in Azure simulating cost changes for increased GPT-35-Turbo usage.":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [Microsoft Cost Management](/azure/cost-management-billing/costs/overview-cost-management).
