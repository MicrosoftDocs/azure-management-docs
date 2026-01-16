---
title: Monitor delegated resources at scale
description: Azure Lighthouse helps you use Azure Monitor Logs in a scalable way across customer tenants.
ms.date: 01/16/2026
ms.topic: how-to
ms.custom:
# Customer intent: As a service provider managing multiple customer tenants, I want to monitor and analyze data across these tenants using Log Analytics workspaces, so that I can improve operational efficiency and ensure compliance with diagnostic data policies.
---

# Monitor delegated resources at scale

As a service provider, you can onboard customer tenants to [Azure Lighthouse](../overview.md). By using Azure Lighthouse, service providers can perform operations at scale across several tenants at once, making management tasks more efficient.

This article shows you how to use [Azure Monitor Logs](/azure/azure-monitor/logs/data-platform-logs) in a scalable way across the customer tenants you're managing. While this article refers to service providers and customers, this guidance also applies to [enterprises using Azure Lighthouse to manage multiple tenants](../concepts/enterprise.md).

> [!NOTE]
> Make sure users in your managing tenant have the [necessary roles for managing Log Analytics workspaces](/azure/azure-monitor/logs/manage-access#azure-rbac) on your delegated customer subscriptions.

## Create Log Analytics workspaces

To collect data, you need to create Log Analytics workspaces. These workspaces are unique environments for data collected by Azure Monitor. Each workspace has its own data repository and configuration. You configure data sources and solutions to store their data in a particular workspace.

We recommend creating these workspaces directly in the customer tenants, so that their data stays in their tenants instead of being exported to yours. When you create workspaces in customer tenants, you can centrally monitor any resources or services supported by Log Analytics. This approach gives you more flexibility on what types of data you monitor. To collect information from [diagnostic settings](/azure/azure-monitor/essentials/diagnostic-settings), workspaces must be created in the customer tenants.

> [!TIP]
> Any automation account that accesses data from a Log Analytics workspace must be created in the same tenant as the workspace.

You can [create a Log Analytics workspace](/azure/azure-monitor/logs/quick-create-workspace) by using the Azure portal, PowerShell, Azure CLI, Bicep, or ARM templates.

> [!IMPORTANT]
> If you only create workspaces in customer tenants, you must also [register](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider) the Microsoft.Insights resource providers on a subscription in the managing tenant. If your managing tenant doesn't have an existing Azure subscription, you can register the resource provider manually by using the following PowerShell commands:
>
> ```powershell
> $ManagingTenantId = "your-managing-Azure-AD-tenant-id"
>
> # Authenticate as a user with admin rights on the managing tenant
> Connect-AzAccount -Tenant $ManagingTenantId
>
> # Register the Microsoft.Insights resource providers Application Ids
> New-AzADServicePrincipal -ApplicationId 1215fb39-1d15-4c05-b2e3-d519ac3feab4 -Role Contributor
> New-AzADServicePrincipal -ApplicationId 6da94f3c-0d67-4092-a408-bb5d1cb08d2d -Role Contributor
> New-AzADServicePrincipal -ApplicationId ca7f3f0b-7d91-482c-8e09-c5d840d0eac5 -Role Contributor
> ```

## Deploy policies that log data

After you create your Log Analytics workspaces, use [Azure Policy](/azure/governance/policy/overview) across your customer hierarchies to send diagnostic data to the appropriate workspace in each tenant. The exact policies you deploy might vary, depending on the resource types that you want to monitor.

For more information about creating policies, see [Tutorial: Create and manage policies to enforce compliance](/azure/governance/policy/tutorials/create-and-manage). For a script to help you create policies to monitor the specific resource types that you choose, try this [community tool](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/azure-diagnostics-policy-generator).

Once you determine which policies to deploy, [deploy them to your delegated subscriptions at scale](policy-at-scale.md).

## Analyze the gathered data

After you deploy your policies, the Log Analytics workspaces you created in each customer tenant start logging data. For insights across all managed customers, use tools such as [Azure Workbooks](/azure/azure-monitor/visualize/workbooks-overview) that gather and analyze information from multiple data sources.

## Query data across customer workspaces

Use [log queries](/azure/azure-monitor/logs/log-query-overview) to retrieve data across Log Analytics workspaces in different customer tenants by creating a union that includes multiple workspaces. By including the TenantID column, you can see which results belong to which tenants.

The following example query creates a union on the AzureDiagnostics table across workspaces in two separate customer tenants. The results show the Category, ResourceGroup, and TenantID columns.

``` Kusto
union AzureDiagnostics,
workspace("WS-customer-tenant-1").AzureDiagnostics,
workspace("WS-customer-tenant-2").AzureDiagnostics
| project Category, ResourceGroup, TenantId
```

For more examples of queries across multiple Log Analytics workspaces, see [Query data across Log Analytics workspaces, applications, and resources in Azure Monitor](/azure/azure-monitor/logs/cross-workspace-query).

> [!IMPORTANT]
> If you use an automation account to query data from a Log Analytics workspace, that automation account must be in the same tenant as the workspace.

## View alerts across customers

You can view [alerts](/azure/azure-monitor/alerts/alerts-overview) for delegated subscriptions in the customer tenants that you manage. From your managing tenant, you can [create, view, and manage activity log alerts](/azure/azure-monitor/alerts/alerts-activity-log) in the Azure portal or through APIs and management tools.

To refresh alerts automatically across multiple customers, use an [Azure Resource Graph](/azure/governance/resource-graph/overview) query to filter for alerts. You can pin the query to your dashboard and select all of the appropriate customers and subscriptions.

For example, the following query displays alerts with severity 0 and 1, refreshing every 60 minutes.

```kusto
alertsmanagementresources
| where type == "microsoft.alertsmanagement/alerts"
| where properties.essentials.severity =~ "Sev0" or properties.essentials.severity =~ "Sev1"
| where properties.essentials.monitorCondition == "Fired"
| where properties.essentials.startDateTime > ago(60m)
| project StartTime=properties.essentials.startDateTime,name,Description=properties.essentials.description, Severity=properties.essentials.severity, subscriptionId
| sort by tostring(StartTime)
```

## Next steps

- Try out the [Activity Logs by Domain](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain) workbook on GitHub.
- Explore this [MVP-built sample workbook](https://github.com/scautomation/Azure-Automation-Update-Management-Workbooks), which tracks patch compliance reporting by [querying Azure Update Manager data](/azure/automation/update-management/query-logs) across multiple Log Analytics workspaces.
- Learn about other [cross-tenant management experiences](../concepts/cross-tenant-management-experience.md) enabled through Azure Lighthouse.
