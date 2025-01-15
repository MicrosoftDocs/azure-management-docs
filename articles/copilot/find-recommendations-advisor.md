---
title: Find your recommendations using Microsoft Copilot in Azure
description: Learn how to find your Azure Advisor recommendations using Microsoft Copilot in Azure.
ms.date: 11/08/2024
ms.topic: conceptual
ms.service: copilot-for-azure
ms.author: v-josmartin
author: jm-247-ms

---

# Find your recommendations using Microsoft Copilot in Azure

Microsoft Copilot in Azure (preview) directs you to your Azure Advisor recommendations across the Well Architected Framework pillars. The Well Architected Framework pillars include Reliability, Cost Optimization, Operational Excellence, Performance Efficiency, and Security. To learn more about the Well Architected Framework, see [Azure Well-Architected Framework](/azure/well-architected "Azure Well-Architected Framework | Microsoft Learn").

When you ask Microsoft Copilot in Azure questions about your recommendations, refine the scope of your request about recommendations by including the following information:

- Pillar
- Impact
- Subscription
- Resource selection panel

[!INCLUDE [scenario-note](includes/scenario-note.md)]

[!INCLUDE [preview-note](includes/preview-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to find recommendations. Modify these prompts based on your real-life scenarios, or try additional prompts to meet your needs.

- "Show me my reliability recommendations"
- "Show me my Advisor recommendations"
- "What are the top cost saving recommendations?"
- "How do I improve performance?"
- "Show me my security recommendations"

## Examples

You can prompt Copilot in Azure to "**Show me my reliability recommendations.**" Copilot in Azure asks if you want to pick specific resources for which to view associated recommendations. If you select **No**, Copilot in Azure returns a list of top reliability recommendations for you, including links to each recommendation page in Advisor.

If you select **Yes**, Copilot in Azure shows the **Select Resources** pane. Here, you can see all of the resources to which you have access. You can filter resources by **Subscription**, **Resource group**, **Type**, or **Location**, or search for specific resources using text. After you select one or more resources, Copilot in Azure returns a list of recommendations specific to those resources.

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Read an [Introduction to Azure Advisor](/azure/advisor/advisor-overview "Introduction to Azure Advisor | Azure Advisor | Microsoft Learn").