---
title: Monitor Edge RAG Preview
description: "Learn how to monitor Edge RAG metrics using Azure Monitor, Azure Managed Grafana, and Azure Arc for better insights."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/13/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a cloud administrator or DevOps engineer, I want to monitor Edge RAG metrics using Azure Monitor and Azure Managed Grafana so that I can obtain insights into the performance and health of my edge deployment for better operational management.
---

# Monitor Edge RAG Preview, enabled by Azure Arc

Edge RAG publishes metrics to [Azure Monitor](/azure/azure-monitor/fundamentals/overview) to assess the performance of the deployed extension. These metrics are composed of numerical values within a sequential set of time-series data. They offer detailed insights into specific aspects of the extension at a particular point in time.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- If you want to use [Azure Managed Grafana](/azure/managed-grafana/overview) to visualize your metrics, [configure an Azure Monitor data source plug-in](/azure/azure-monitor/visualize/grafana-plugin#configure-an-azure-monitor-data-source-plug-in).
- Review [Edge RAG metrics available for monitoring](observability-metrics.md).

## Review the metrics visualizations

Build a chart with metrics for Edge RAG in the Azure portal or by using Azure Managed Grafana.

#### [Azure portal](#tab/azure-portal)

1. Go to the Extensions blade in the Azure portal: **AKS cluster on Azure Local** > **Settings** > **Extensions**.

1. Select the installed Edge RAG extension:

   :::image type="content" source="media/observability/extensions.png" alt-text="Screenshot of the Azure portal showing the Extensions blade for an AKS cluster on Azure Local.":::

1. Go to the **Monitoring** section on the left pane and select **Metrics**.

   :::image type="content" source="media/observability/extension-metrics.png" alt-text="Screenshot showing the metrics section on the extension in the Azure portal.":::

1. Select the **Metric** drop-down list to view the available metrics and build your chart.

   The following image shows an example chart illustrating three distinct metrics recorded on specific dates.

   :::image type="content" source="media/observability/example-metrics-chart.png" alt-text="Screenshot showing a chart with three distinct metrics recorded on specific dates.":::

1. Interact with the chart by selecting **Add filter** or **Apply splitting**. For more information, see [Use dimension filters and splitting](/azure/azure-monitor/metrics/analyze-metrics#use-dimension-filters-and-splitting).

#### [Azure Managed Grafana](#tab/azure-managed-grafana)

When [creating a new visualization](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/create-dashboard/) panel, configure as follows:

1. For **Data Source:**, choose **Azure Monitor**.
 
   :::image type="content" source="media/observability/grafana-data-source.png" alt-text="Screenshot showing the Azure Monitor data source configuration in Azure Managed Grafana.":::  
1. Select **Select a resource**.  
1. In the drop-down menu, go to your Azure Arc resource:  
   **Subscription > Resource Group > Azure Arc Resource**.
1. Open **Advanced**.  

   :::image type="content" source="media/observability/grafana-select-resource.png" alt-text="Screenshot showing the Azure Managed Grafana interface with the 'Select a resource' drop-down menu expanded, displaying available Azure Arc resources grouped by subscription and resource group.":::  

1. Change the **Namespace** to: **Microsoft.Kubernetes/connectedClusters/providers/extensions**  
1. Add the following to the resource name:  
  `<Current Resource Name>/Microsoft.KubernetesConfiguration/<Extension Name>`

   For example:  

      :::image type="content" source="media/observability/grafana-namespace-resource-name.png" alt-text="Screenshot showing the Azure Managed Grafana interface with the Namespace field set to 'Microsoft.Kubernetes/connectedClusters/providers/extensions' and the Resource Name field populated with a specific resource path.":::  
1. Select **Apply**  
1. Select a metric from the **"Metric"** drop-down.  
1. Adjust the query as needed. Example:  

   :::image type="content" source="media/observability/grafana-chart-example.png" alt-text="Screenshot showing an example Grafana chart displaying metrics data with multiple lines representing different metrics over time." lightbox="media/observability/grafana-chart-example.png" :::

----

## Configure Infrastructure Monitoring

To configure infrastructure monitoring, see the following articles:

- Azure CLI: [Enable monitoring for Kubernetes clusters](/azure/azure-monitor/containers/kubernetes-monitoring-enable)

- Azure portal: [Enable full monitoring with Azure portal](/azure/azure-monitor/containers/kubernetes-monitoring-enable?tabs=cli#enable-full-monitoring-with-azure-portal)

## Visualize your metrics

Visualize your metrics by going to Azure Arc, Azure Monitor, or by using Azure Managed Grafana. For more information, see the following articles.

- Azure Arc insights: [Monitor your Kubernetes cluster performance with Container insights](/azure/azure-monitor/containers/container-insights-analyze)

- Azure Monitor workspace: [Azure Monitor metrics explorer with PromQL](/azure/azure-monitor/metrics/metrics-explorer)

- Azure workbooks: [*Query Prometheus metrics using Azure workbooks*](/azure/azure-monitor/metrics/prometheus-workbooks)

- Azure Managed Grafana: [Monitor your Azure services in Grafana](/azure/azure-monitor/visualize/grafana-plugin)

## Related content

- [Enable monitoring for Kubernetes clusters](/azure/azure-monitor/containers/kubernetes-monitoring-enable)
- [Configure an Azure Monitor data source plug-in](/azure/azure-monitor/visualize/grafana-plugin#configure-an-azure-monitor-data-source-plug-in)
