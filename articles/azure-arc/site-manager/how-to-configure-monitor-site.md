---
title: Configure Azure Monitor Alerts for a Site
description: Learn how to create and configure alerts by using Azure Monitor to manage resources in an Azure Arc site.
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to
ms.date: 04/18/2024

#customer intent: "As a site administrator, I want to configure alerts for resources by using Azure Monitor so that I can effectively manage and monitor the status of resources within my Azure Arc site."
---

# Monitor sites in Azure Arc

Azure Arc sites provide a centralized view to monitor groups of resources, but they don't provide monitoring capabilities for sites overall. Instead, you can set up alerts and monitoring for supported resources within a site. After alerts are set up and triggered depending on the alert criteria, the Azure Arc site manager (preview) makes the resource alert status visible within the site pages.

If you aren't familiar with Azure Monitor, learn more about how to [monitor Azure resources with Azure Monitor](/azure/azure-monitor/essentials/monitor-azure-resource).

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure portal access.
* Internet connectivity.
* A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).

## Configure alerts for sites in Azure Arc

This section provides basic steps for configuring alerts for sites in Azure Arc. For more information about Azure Monitor, see [Create or edit an alert rule](/azure/azure-monitor/alerts/alerts-create-metric-alert-rule).

To configure alerts for sites in Azure Arc, follow these steps:

1. Go to Azure Monitor by searching for **monitor** in the Azure portal. Select **Monitor**.

   :::image type="content" source="./media/how-to-configure-monitor-site/search-monitor.png" alt-text="Screenshot that shows searching for monitor in the Azure portal.":::

1. On the **Overview** page, select **Alerts** on either the service menu or in the boxes shown on the primary pane.

   :::image type="content" source="./media/how-to-configure-monitor-site/select-alerts-monitor.png" alt-text="Screenshot that shows selecting the Alerts option on the Overview page.":::

1. On the **Alerts** page, you can manage existing alerts or create new ones:

   - Select **Alert rules** to see all of the alerts currently in effect in your subscription.
   - Select **Create** to create an alert rule for a specific resource. If a resource is managed as part of a site, any alerts triggered via its rule appear in the site manager overview.

   :::image type="content" source="./media/how-to-configure-monitor-site/create-alert-monitor.png" alt-text="Screenshot that shows the Create and Alert rules actions on the Alerts page.":::

By having either existing alert rules or creating a new alert rule, after the rule is in place for resources supported by the Azure Arc site monitor, any alerts that are triggered on that resource are visible on the **Overview** tab for the site.

## Related content

- To learn how to view alerts triggered from Azure Monitor for supported resources in the site manager, see [View alerts in the site manager](./how-to-view-alerts.md).