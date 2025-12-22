---
title: Work with Azure Load Balancer using Azure Copilot
description: Learn how Azure Copilot can help you understand and use Azure Load Balancer.
ms.date: 11/20/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
# Customer intent: "As an network administrator, I want to get recommendations for the appropriate load balancer type in Azure, so that I can configure the best solution for my networking requirements and ensure future compliance with upcoming changes."
---

# Work with Azure Load Balancer using Azure Copilot

Azure Copilot can help you work more effectively with [Azure Load Balancer](/azure/load-balancer/load-balancer-overview). You can ask Azure Copilot to help you understand what type of load balancer is right for your requirements. You can also get help if you need to upgrade from Basic Load Balancer to Standard Load Balancer.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Choose the right load balancer

Azure Copilot can recommend the best type of load balancer to fit your scenario. When you ask for help choosing a load balancer, Azure Copilot asks questions to help understand your needs, then provides recommendations. Azure Copilot also gives you the option to go directly to the **Create load balancer** experience to set up the load balancer.

### Load balancer recommendation sample prompts

Here are a few examples of the kinds of prompts you can use to get load balancer recommendations. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Which type of load balancer should I use"
- "Can you help me to choose a load balancer"
- "Which type of load balancer fit my scenario best"
- "I want to know which type of load balancer should I use"
- "Help me choose a load balancer by providing me a questionnaire"

### Load balancer recommendation example

When you say "**Which type of load balancer should I use**", Azure Copilot asks questions to better understand your scenario. After that, it provides a recommendation, along with a link to create your load balancer.

:::image type="content" source="media/work-load-balancer/load-balancer-recommendation.png" lightbox="media/work-load-balancer/load-balancer-recommendation.png" alt-text="Screenshot of Azure Copilot asking questions about a scenario and providing load balancer recommendations.":::

## Upgrade to Standard Load Balancer

As of September 30, 2025, Basic Load Balancer [is retired](https://azure.microsoft.com/updates/azure-basic-load-balancer-will-be-retired-on-30-september-2025-upgrade-to-standard-load-balancer/). If you still use Basic Load Balancer, Azure Copilot can help you understand [how to migrate from Basic to Standard](/azure/load-balancer/load-balancer-basic-upgrade-guidance) so that you can upgrade as soon as possible.

To get help upgrading to Standard Load Balancer, navigate to your Basic Load Balancer, then select the banner that appears near the top of the pane. Select one of the provided prompts to get assistance from Azure Copilot. You can also enter prompts directly, such as "**What are the benefits of a Standard SKU Load Balancer?**" or "**Help me upgrade to a Standard SKU Load Balancer**."

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- [Get tips for writing effective prompts](write-effective-prompts.md) to use with Azure Copilot.
