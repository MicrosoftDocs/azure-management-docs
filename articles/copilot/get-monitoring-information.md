---
title: Get information about Azure Monitor metrics and logs using Azure Copilot
description: Learn about scenarios where Azure Copilot can provide information about Azure Monitor metrics and logs.
ms.date: 04/14/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
ms.collection: ce-skilling-ai-copilot
# Customer intent: As a cloud operations manager, I want to query Azure Monitor metrics, logs, and alerts using AI assistance, so that I can quickly gather insights and make informed decisions about resource performance and issues.
---

# Get information about Azure Monitor metrics, logs, and alerts using Azure Copilot

You can ask Azure Copilot questions about metrics and logs collected by [Azure Monitor](/azure/azure-monitor/), and about Azure Monitor alerts.

When you ask Azure Copilot for this information, it automatically pulls context when possible, based on the current conversation or on the page you're viewing in the Azure portal. If the context of a query isn't clear, you'll be prompted to specify the resource for which you want information.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Answer questions about Azure Monitor platform metrics

Use Azure Copilot to ask questions about your Azure Monitor metrics. When asked about metrics for a particular resource, Azure Copilot generates a graph, summarizes the results, and allows you to further explore the data in Metrics Explorer. When asked about what metrics are available, Azure Copilot describes the platform metrics available for the given resource type.

### Platform metrics sample prompts

Here are a few examples of the kinds of prompts you can use to get information about Azure Monitor platform metrics. Modify these prompts based on your real-life scenarios, or try additional prompts to get different kinds of information.

- "What platform metrics are available for my VM?"
- "Show me the memory usage trend for my VM over the last 4 hours"
- "Show trends for network bytes in over the last day"
- "Give me a chart of os disk latency statistics for the last week"

### Platform metrics examples

When you're working with a resource, you can say "**Show me available metrics for this resource**." Azure Copilot lets you know which metrics are available.

:::image type="content" source="media/get-monitoring-information/monitor-available-metrics.png" alt-text="Screenshot of Azure Copilot listing which metrics are available for a resource.":::

You can then follow up by asking to see a particular metric for a set period of time.

:::image type="content" source="media/get-monitoring-information/monitor-show-metric.png" alt-text="Screenshot of Azure Copilot showing request metrics for a resource.":::

## Answer questions about Azure Monitor logs

When asked about logs for a particular resource, Azure Copilot generates an example KQL expression and allows you to further explore the data in Azure Monitor logs. This capability is available for all customers using Log Analytics, and can be used in the context of a particular Azure Kubernetes Service (AKS) cluster that uses Azure Monitor logs.

To get details about your container logs, navigate to your AKS cluster. In the service menu, under **Monitoring**, select **Logs**. From there, ask Copilot for information about your container logs.

### Logs sample prompts

Here are a few examples of the kinds of prompts you can use to get information about Azure Monitor logs for an AKS cluster. Modify these prompts based on your real-life scenarios, or try additional prompts to get different kinds of information.

- "Are there any errors in container logs?"
- "Show logs for the last day of pod <provide_pod_name> under namespace <provide_namespace>"
- "Show me container logs that include word 'error' for the last day for namespace 'xyz'"
- "Check in logs which containers keep restarting"
- "Show me all Kubernetes events"

## Answer questions about Azure Monitor alerts

Use Azure Copilot to ask questions about your Azure Monitor alerts. When asked about alerts, Azure Copilot summarizes the list of alerts, their severity, and allows you to further explore the data in the alerts page. 

### Alerts sample prompts

Here are a few examples of the kinds of prompts you can use to get information about Azure Monitor alerts. Modify these prompts based on your real-life scenarios, or try additional prompts to get different kinds of information.

- "Are there any alerts for my resource?"
- "Tell me more about these alerts. How many critical alerts are there?"
- "Show me all the alerts in my resource group"
- "List all the alerts for the subscription"
- "Show me all alerts triggered during the last 24 hours"

### Alerts examples

You can say "**Catch me up on alerts** to get an overview of active alerts across your resources. Azure Copilot shares the types of alerts, along with details about the top alerts that are currently active.

:::image type="content" source="media/get-monitoring-information/monitor-active-alerts.png" alt-text="Screenshot of Azure Copilot summarizing all active alerts.":::

To narrow it down, you can say "**Show me all alerts triggered during the last 24 hours**."

:::image type="content" source="media/get-monitoring-information/monitor-recent-alerts.png" alt-text="Screenshot of Azure Copilot summarizing active alerts in the past 24 hours.":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure Monitor](/azure/azure-monitor/) and [how to use it with AKS clusters](/azure/aks/monitor-aks).
