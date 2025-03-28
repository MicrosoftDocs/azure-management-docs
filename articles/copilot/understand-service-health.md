---
title: Understand service health events and status using Microsoft Copilot in Azure
description: Learn about scenarios where Microsoft Copilot in Azure can provide information about service health events.
ms.date: 03/28/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
---

# Understand service health events and status using Microsoft Copilot in Azure

You can ask Microsoft Copilot in Azure questions to get information from [Azure Service Health](/azure/service-health/overview). This provides a quick way to find out if there are any service health events impacting your Azure subscriptions. You can also get more information about a known service health event.

[!INCLUDE [scenario-note](includes/scenario-note.md)]



## Sample prompts

Here are a few examples of the kinds of prompts you can use to get service health information. Modify these prompts based on your real-life scenarios, or try additional prompts about specific service health events.

- "Am I impacted by any service health events?"
- "Is there any outage impacting me?"
- "Can you tell me more about tracking ID {0}?"
- "Is the event with tracking ID {0} still active?"
- "What is the status of the event with tracking ID {0}?"
- "Is the incident related to (Azure service or feature) still active?"

## Examples

You can ask "**Is there any Azure outage ongoing?**" In this example, no outages or service health issues are found. If Copilot responds that there are service health issues impacting your account, you can ask further questions to get more information.

:::image type="content" source="media/understand-service-health/azure-service-health-any-issues.png" alt-text="Screenshot of Microsoft Copilot in Azure checking for active service health issues.":::

You can also ask about a specific service event to see whether it's been resolved. If you know the tracking ID, you can ask "**Is incident {0} still active?**"

:::image type="content" source="media/understand-service-health/azure-service-health-still-active.png" alt-text="Screenshot of Microsoft Copilot in Azure checking whether a service health issue is still active.":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [Azure Monitor](/azure/azure-monitor/).
