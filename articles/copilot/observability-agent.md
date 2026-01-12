---
title: Observability agent capabilities in Agents (preview) in Azure Copilot
description: Agents (preview) in Azure Copilot helps investigate Azure Monitor alerts and provide observability using intelligent agent capabilities.
author: JnHs
ms.author: jenhayes
ms.date: 1/12/2025
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use agents to help investigate Azure Monitor alerts, so that I can be more successful performing tasks in  my Azure tenant."
---

# Observability capabilities in Agents (preview) in Azure Copilot

[Agents (preview) in Azure Copilot](agents-preview.md) intelligently surfaces the right agent to help with your tasks. When you ask Azure Copilot for help with investigating Azure Monitor alerts, Azure Copilot creates an Azure Monitor issue and starts an [investigation](/azure/azure-monitor/aiops/aiops-issue-and-investigation-overview). If no default Azure Monitor Workspace is configured for the subscription (or one is not passed as context in the prompt), Azure Copilot attempts to configure one for you. Azure Copilot investigates the alert and provides a summary of the issue and its findings, including possible explanations and potential remediation steps. You can view more details by following a link to the Azure Monitor issue it created.

> [!NOTE]
> To use investigation capabilities, you must have the Contributor, Monitoring Contributor, or Issue Contributor role on the Azure Monitor Workspace. For more information, see [Use Azure Monitor issues and investigations (preview)](/azure/azure-monitor/aiops/aiops-issue-and-investigation-how-to).

You can ask for help understanding alerts when viewing an alert instance in the Azure portal. If you aren't currently viewing the alert, provide the Azure resource ID for the alert so that Azure Copilot knows which alert to investigate.

> [!IMPORTANT]
> The functionality described in this article is only available for tenants that have access to [Agents (preview) in Azure Copilot](agents-preview.md).

## Observability sample prompts

Here are a few examples of the kinds of prompts you can use to get help with observability tasks. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. When using these kinds of prompts, be sure to enable agent mode by selecting the icon in the chat window.

:::image type="content" source="media/agent-preview/azure-copilot-agent-mode-active.png" alt-text="Screenshot showing agent mode (preview) enabled in Azure Copilot.":::

When you have the alert ID, you can use prompts like these:

- "Start an investigation for my alert: `/subscriptions/SUB_ID/resourcegroups/RESOURCE_GROUP/providers/microsoft.insights/components/COMPONENT_NAME/providers/Microsoft.AlertsManagement/alerts/ALERT_ID`"
- "Can you help with this alert? The ID is `/subscriptions/SUB_ID/resourcegroups/RESOURCE_GROUP/providers/microsoft.insights/components/COMPONENT_NAME/providers/Microsoft.AlertsManagement/alerts/ALERT_ID`"
- "Troubleshoot this alert: `/subscriptions/SUB_ID/resourcegroups/RESOURCE_GROUP/providers/microsoft.insights/components/COMPONENT_NAME/providers/Microsoft.AlertsManagement/alerts/ALERT_ID`"

When viewing a specific alert in the Azure portal, you can ask more general questions without needing to provide the alert ID. For example:

- "Can you help investigate this alert?"
- "Can you help troubleshoot this?"

For example, when viewing an alert, you can say "**Start an investigation for this alert**." Azure Copilot begins an investigation. Select **Show activity** to view progress and reasoning as Azure Copilot works on your request.

:::image type="content" source="media/observability-agent/observability-agent-investigate-alert.png" alt-text="Screenshot of Azure Copilot beginning an alert investigation with detailed reasoning.":::

When the investigation is complete, Azure Copilot provides a summary of its findings with steps you can take to remediate the issue, along with links where you can find more information.

:::image type="content" source="media/observability-agent/observability-agent-steps.png" alt-text="Screenshot of Azure Copilot providing steps to investigate and remediate an alert.":::

## Current considerations and limitations

Keep in mind the following considerations and limitations when working with observability in Agents (preview) in Azure Copilot.

- Agent capabilities are currently scoped to alerts coming from Application Insights components. Support for other alert types isn't currently available.
- While the agent can perform investigations and recommend remediations, it can't perform remediation steps suggested by the investigation results.
