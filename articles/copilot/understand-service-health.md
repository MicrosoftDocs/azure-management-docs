---
title: Understand service health events and status using Microsoft Copilot in Azure
description: Learn about scenarios where Microsoft Copilot in Azure can provide information about service health events.
ms.date: 04/24/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: "As an Azure user, I want to inquire about current service health events and status, so that I can quickly assess if my resources are impacted and understand any ongoing maintenance or security issues affecting my system."
---

# Understand service health events and status using Microsoft Copilot in Azure

You can ask Microsoft Copilot in Azure questions to get information from [Azure Service Health](/azure/service-health/overview) and [Azure Resource Health](/azure/service-health/resource-health-overview). This tool provides a quick way to find out if there are any service health events impacting your Azure subscriptions. You can also get more information about a known service health event, planned maintenance, and security or health advisories.

Copilot in Azure can be helpful for all Microsoft customers who use Service health and Resource health to assess their system health.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to get service health information. Modify these prompts based on your real-life scenarios, or try other prompts about specific service health events.

- "Am I being impacted by any service health events?"
- "Is there any outage impacting me?"
- "Can you tell me more about tracking ID {0}?"
- "Is the event with tracking ID {0} still active?"
- "Do I have any impacted resources from event {0}?"
- "Any Post Incident Reviews?"
- "How many active Azure security advisories?"
- "How many security advisories in the last 10 days?"
- "How many Azure retirements?"
- "Are there any active planned maintenance events?"
- "How many Azure health advisories over the last 20 days?" 
- "Can you share health information of resource <br>``/subscriptions/testSub/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testResource?`

## Examples

You can ask "**Is there any Azure outage ongoing?**" In this example, no outages or service health issues are found. If Copilot responds that there are service health issues impacting your account, you can ask further questions to get more information.

:::image type="content" source="media/understand-service-health/azure-service-health-any-issues.png" alt-text="Screenshot of Microsoft Copilot in Azure checking for active service health issues.":::

You can also ask about a specific service event to see whether it's resolved. If you know the tracking ID, you can ask "**Is incident {0} still active?**"

:::image type="content" source="media/understand-service-health/azure-service-health-still-active.png" alt-text="Screenshot of Microsoft Copilot in Azure checking whether a service health issue is still active.":::

To find out about current or past planned maintenance events, use prompts like "**Any planned maintenance events?**" or "**How many planned maintenance events in the last 20 days?**"

:::image type="content" source="media/understand-service-health/azure-service-health-planned-maintenance.png" alt-text="Screenshot of Microsoft Copilot in Azure checking if there are any planned maintenance events.":::

:::image type="content" source="media/understand-service-health/azure-service-health-planned-maintenance-in-last-days.png" alt-text="Screenshot of Microsoft Copilot in Azure checking if there are any planned maintenance events in the last 20 days.":::

You can also ask about current and past security advisories. For example, "**How many active security advisories?**" or "**How many security advisories in the last 50 days?**"

:::image type="content" source="media/understand-service-health/azure-service-health-how-many-security-advisories.png" alt-text="Screenshot of Microsoft Copilot in Azure checking about any security advisories.":::

:::image type="content" source="media/understand-service-health/azure-service-health-how-many-security-advisories-in-last-days.png" alt-text="Screenshot of Microsoft Copilot in Azure checking to see if there have been any security advisories in the last 50 days.":::

To check whether any Post Incident Reviews (PIRs) are available, ask "**Are there any PIRs?**"

:::image type="content" source="media/understand-service-health/azure-service-health-any-pir.png" alt-text="Screenshot of Microsoft Copilot responding with data and link to click.":::

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [Azure Monitor](/azure/azure-monitor/).
