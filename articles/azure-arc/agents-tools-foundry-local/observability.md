---
title: Monitor Agents and Tools with Foundry Local
description: "Learn how to monitor Agents and Tools with Foundry Local using Azure Monitor, Azure Managed Grafana, and application logs."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/02/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
  - sfi-ropc-nochange
  - sfi-image-nochange
# Customer intent: As a cloud administrator or DevOps engineer, I want to monitor Agents and Tools with Foundry Local metrics using Azure Monitor and Azure Managed Grafana so that I can obtain insights into the performance and health of my edge deployment for better operational management.
---

# Monitor Agents and Tools with Foundry Local

Use [Azure Monitor](/azure/azure-monitor/fundamentals/overview) and Azure Managed Grafana to monitor the performance of **Agents and Tools with Foundry Local**. Use application logs to investigate errors.

## Prerequisites

Before you begin:

- If you want to use [Azure Managed Grafana](/azure/managed-grafana/overview) to visualize your metrics, [configure the Azure Monitor data source](https://grafana.com/docs/grafana/latest/datasources/azure-monitor/configure/).
- Review [Metrics for Agents and Tools with Foundry Local](observability-metrics.md).

## Review the metrics visualizations

Build a chart with metrics for **Agents and Tools with Foundry Local** in the Azure portal or by using Azure Managed Grafana.

#### [Azure portal](#tab/azure-portal)

1. Go to the Extensions blade in the Azure portal: **AKS cluster on Azure Local** > **Settings** > **Extensions**.

1. Select the installed **Agents and Tools with Foundry Local** extension:

   :::image type="content" source="media/observability/extensions.png" alt-text="Screenshot of the Azure portal showing the Extensions blade for an AKS cluster on Azure Local.":::

1. Go to the **Monitoring** section on the left pane and select **Metrics**.
1. Select the **Metric** drop-down list to view the available metrics and build your chart.
1. Interact with the chart by selecting **Add filter** or **Apply splitting**. Use available dimensions such as **AppId**, **Operation**, and **ResultType**. For more information, see [Use dimension filters and splitting](/azure/azure-monitor/metrics/analyze-metrics#use-dimension-filters-and-splitting).

#### [Azure Managed Grafana](#tab/azure-managed-grafana)

When [creating a new visualization](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/create-dashboard/) panel, configure as follows:

1. For **Data Source:**, choose **Azure Monitor**.

   :::image type="content" source="media/observability/grafana-data-source.png" alt-text="Screenshot showing the Azure Monitor data source configuration in Azure Managed Grafana.":::
1. Select **Select a resource**.
1. Search for and select the **Kubernetes - Azure Arc** resource where **Agents and Tools with Foundry Local** is installed.
1. Open **Advanced**.

   :::image type="content" source="media/observability/grafana-select-resource.png" alt-text="Screenshot of the Select a resource list in Azure Managed Grafana, showing Azure Arc resources.":::

1. Set the **Namespace** to `Microsoft.Kubernetes/connectedClusters/providers/extensions`.
   For example:

      :::image type="content" source="media/observability/grafana-namespace-resource-name.png" alt-text="Screenshot showing the advanced resource picker with Namespace set to Microsoft.Kubernetes/connectedClusters/providers/extensions and an extension resource name.":::

1. Update the **Resource Name** by adding the following value where you replace `Extension Name` with the name of the **Agents and Tools with Foundry Local** extension:

   `<Current Resource Name>/Microsoft.KubernetesConfiguration/<Extension Name>`

   For example, `contoso-arc-cluster/Microsoft.KubernetesConfiguration/edgeragdemo`

   If you don't know the extension name, find it in the Azure portal by going to your **Kubernetes - Azure Arc** resource > **Settings** > **Extensions**. Look for the **Name** where **Type** is `microsoft.arc.rag`.

1. Select **Apply**.
1. Set **Metric namespace** to `Microsoft.KubernetesConfiguration/extensions` in the query editor.
1. Select a metric from the **Metric** drop-down. Choose an appropriate aggregation, such as an average for duration metrics.
1. Adjust the query as needed. For example, split the chart by the **Operation** or **ResultType** dimension.

----

## Check application logs

To investigate failed requests or ingestion, check the application container's logs. You need Kubernetes access to list and get pods and read pod logs. Replace the placeholders with the extension's namespace, pod, and application container.

```bash
kubectl get pods --namespace <namespace>
kubectl logs --namespace <namespace> <pod-name> --container <application-container> --tail=100
```

Look for errors around the time of the failure. Remove identifying information and credentials before sharing logs with support.

## Configure infrastructure monitoring

To configure monitoring for AKS on Azure Local, see [Enable monitoring for Arc-enabled Kubernetes clusters](/azure/azure-monitor/containers/kubernetes-monitoring-enable-arc).

## Visualize infrastructure metrics

Visualize Kubernetes infrastructure metrics by using Azure Arc, Azure Monitor, or Azure Managed Grafana. Prometheus queries don't automatically include the extension's application metrics. For more information, see the following articles.

- Azure Arc insights: [Monitor your Kubernetes cluster performance with Container insights](/azure/azure-monitor/containers/container-insights-analyze)

- Azure Monitor workspace: [Azure Monitor metrics explorer with PromQL](/azure/azure-monitor/metrics/metrics-explorer)

- Azure workbooks: [*Query Prometheus metrics using Azure workbooks*](/azure/azure-monitor/metrics/prometheus-workbooks)

- Azure Managed Grafana: [Monitor your Azure services in Grafana](/azure/azure-monitor/visualize/grafana-plugin)

## Related content

- [Enable monitoring for Arc-enabled Kubernetes clusters](/azure/azure-monitor/containers/kubernetes-monitoring-enable-arc)
- [Configure the Azure Monitor data source](https://grafana.com/docs/grafana/latest/datasources/azure-monitor/configure/)
