---
title: Use Grafana dashboards in Azure Container Registry
description: Use prebuilt and customizable Grafana dashboards for Azure Container Registry directly in the Azure portal.
ms.topic: how-to
author: jeburke
ms.author: jeburke
ms.date: 10/08/2026
ms.service: azure-container-registry
ai-usage: ai-assisted
# Customer intent: As a registry operator, I want to use customizable Grafana dashboards to analyze registry metrics so that I can understand usage patterns, monitor performance, and troubleshoot issues.
---

# Use Grafana dashboards in Azure Container Registry

Azure Container Registry (ACR) provides an integrated metrics visualization experience enabled by [Azure Managed Grafana](/azure/managed-grafana/overview). Use prebuilt or customizable Grafana dashboards to understand registry usage patterns, monitor performance, identify throttling, track storage and data transfer, and troubleshoot operational issues.

Access this experience from the **Dashboards with Grafana** page in your container registry. You don't need to deploy or manage a separate Grafana instance. From the Azure portal, you can:

- Start with Azure-managed dashboards for common ACR monitoring scenarios.
- Create and edit dashboards by adding panels, modifying queries, and applying client-side transformations.
- Save and share dashboards as Azure resources governed by [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview).
- Automate dashboard deployment with [Azure Resource Manager templates](/azure/azure-resource-manager/templates/overview) or [Bicep](/azure/azure-resource-manager/bicep/overview).
- Import compatible dashboards from the Grafana community.
- Open Grafana **Explore** from the registry's integrated portal experience to investigate data and add results to dashboards.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/).
- An [Azure container registry](container-registry-get-started-portal.md).
- Permissions to read ACR monitoring data and create resources in the target subscription and resource group. After you save a dashboard, use Azure RBAC to assign access to the dashboard resource.

## Open the Grafana experience in Azure Container Registry

1. In the Azure portal, open your container registry.
2. On the resource menu, under **Monitoring**, select **Dashboards with Grafana**.

The gallery lists Azure-managed dashboards and dashboards saved for the current registry. It automatically filters the gallery to dashboards associated with Azure Container Registry. You can't change this filter when you access the gallery from a registry.

## Start with a prebuilt dashboard

Azure provides dashboards for common ACR monitoring scenarios. The gallery includes the **Azure Container Registry - Observability** dashboard, which provides an overview of data transfer, transaction trends, throttling, and storage consumption. You can analyze this data by dimensions such as API name and region.

To view the dashboard, select its name in the gallery.

:::image type="content" source="media/dashboards-with-grafana.png" alt-text="Screenshot of the Azure Container Registry Observability Grafana dashboard showing registry metrics and charts.":::

## Create, edit, and save dashboards

You can customize an Azure-managed dashboard or create a dashboard from scratch.

- To edit an Azure-managed dashboard, open it and select **Edit**. Modify its panels, queries, and transformations.
- To save your changes as a new dashboard, select **Save As**, and then choose a subscription, resource group, and name.
- To create a dashboard from scratch, return to the gallery, select **New**, and add panels.

Each saved dashboard is an Azure resource. You can manage it with Azure RBAC, export it as an ARM template, and include it in deployment automation.

> [!NOTE]
> Dashboards created within a registry are automatically tagged so that they appear in the Azure Container Registry Grafana gallery.

## Explore registry data

Grafana **Explore** is available from the **Dashboards with Grafana** page in your container registry. It lets you run ad hoc queries without starting from a dashboard and add the results to a new or existing dashboard.

1. In the Azure portal, open your container registry.
2. On the resource menu, under **Monitoring**, select **Dashboards with Grafana** then **Dashboards**.
3. From the top menu, select **Explore**.
4. Choose a data source, and then build a query for the desired time range.
5. Select **Add to dashboard** to turn the visualization into a panel.

## Import dashboards from the Grafana community

You can import compatible dashboards from the Grafana public gallery that use Azure Monitor or Azure Monitor managed service for Prometheus data sources.

1. In **Dashboards with Grafana**, select **Dashboards**
2. Select **Browse Grafana dashboards Gallery**.
3. Choose a compatible dashboard, and then copy its dashboard ID.
4. Return to **Dashboards with Grafana**, and then select **New**.
5. Select **Import**, and then follow the prompts.

The imported dashboard is saved as an Azure resource.

> [!IMPORTANT]
> The integrated portal experience supports Azure data sources only. For information about supported data sources and how this experience compares with a dedicated Azure Managed Grafana workspace, see [Visualize Azure Monitor data with Grafana](/azure/azure-monitor/visualize/visualize-grafana-overview).

## Make dashboards visible in Azure Container Registry

Dashboards visible from the **Dashboards with Grafana** page in a registry use the following Azure resource tag:

- Name: `GrafanaDashboardResourceType`
- Value: `Microsoft.ContainerRegistry/registries`

Dashboards created from within a registry receive this tag automatically. If you create or import a dashboard outside the registry and want it to appear in the registry's gallery, add the tag manually:

1. Open the dashboard resource.
2. Select **Tags**, and then add the tag name and value.
3. Save the changes.

After you add the tag, refresh the gallery in the registry. The dashboard appears under **Saved dashboards**.

## Manage access and automate deployment

- To control access, assign Azure roles at the dashboard resource, resource group, or subscription scope.
- To automate deployment, export an ARM template from a dashboard and use it to deploy the dashboard across environments.
- The Grafana user interface uses the language configured for the Azure portal.

## Limitations

The integrated Grafana experience in the Azure portal supports Azure data sources only. It doesn't support the following Grafana features:

- Alerts
- Reports
- Library panels
- Snapshots
- Playlists
- App plugins

For current data-source support, feature limitations, and a comparison with Azure Managed Grafana, see [Visualize Azure Monitor data with Grafana](/azure/azure-monitor/visualize/visualize-grafana-overview).

## Troubleshooting

### A dashboard doesn't appear in the gallery

Confirm that the dashboard resource has the `GrafanaDashboardResourceType` tag with the value `Microsoft.ContainerRegistry/registries`. Refresh the gallery after you add or update the tag.

### You can't save a dashboard

Verify that you have permission to create resources in the target subscription and resource group.

### Data doesn't load

Confirm that the registry and selected data source contain data for the selected time range. Also verify that you have permission to read the data source.

## Related content

- [Monitor Azure Container Registry](monitor-container-registry.md)
- [Azure Container Registry monitoring data reference](monitor-container-registry-reference.md)
- [Use Azure Monitor dashboards with Grafana](/azure/azure-monitor/visualize/visualize-use-grafana-dashboards)
- [Visualize Azure Monitor data with Grafana](/azure/azure-monitor/visualize/visualize-grafana-overview)
