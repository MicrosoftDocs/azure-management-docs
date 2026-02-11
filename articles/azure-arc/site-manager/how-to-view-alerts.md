---
title: View Alerts for a Site
description: Learn how to view and create alerts for a site.
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to
ms.date: 04/18/2024
ms.custom: sfi-image-nochange

#customer intent: As a site manager, I want to view the alert status for my Azure Arc site so that I can monitor the health of the underlying resources and take appropriate actions as needed.
---

# View alert status for an Azure Arc site

This article shows you how to view the alert status for an Azure Arc site. The alert status for a site reflects the status of the underlying resources. From the site status view, you can also find detailed status information for the supported resources.

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure portal access.
* Internet connectivity.
* A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).
* A site created for the associated resource group or subscription or service group. If you don't have one, see [Create and manage sites](./how-to-crud-site.md).

## Alert status colors and meanings

In the Azure portal, the color indicates the status:

* **Green**: Means **Up to date**.
* **Blue**: Means **Info**.
* **Purple**: Means **Verbose**.
* **Yellow**: Means **Warning**.
* **Orange**: Means **Error**.
* **Red**: Means **Critical**.

## View alert status

View alert status for an Azure Arc site from the main page of the Azure Arc site manager (preview).

1. In the [Azure portal](https://portal.azure.com), go to **Azure Arc** and select **Site manager (preview)** to open the site manager.

1. From the Azure Arc site manager, go to the **Overview** page.

   :::image type="content" source="./media/how-to-view-alerts/overview-sites-page.png" alt-text="Screenshot that shows selecting the Overview page from the site manager.":::

1. On the **Overview** page, you can view the summarized alert statuses of all sites. This site-level alert status is an aggregation of all the alert statuses of the resources in that site. In the following example, sites are shown with different statuses.

   :::image type="content" source="./media/how-to-view-alerts/site-manager-overview-alerts.png" alt-text="Screenshot that shows viewing the alert status on the site manager Overview page." lightbox="./media/how-to-view-alerts/site-manager-overview-alerts.png":::

1. To understand which site has which status, select either the **Sites** tab or the blue status text.

   :::image type="content" source="./media/how-to-view-alerts/site-manager-overview-alerts-details.png" alt-text="Screenshot that shows the site manager Overview page directing to the Sites page to view more details." lightbox="./media/how-to-view-alerts/site-manager-overview-alerts-details.png":::

1. The **Sites** page shows the top-level status for each site, which reflects the most significant status for the site.

   :::image type="content" source="./media/how-to-view-alerts/site-manager-overview-alerts-details-status-site-page.png" alt-text="Screenshot that shows the top-level alert status for each site." lightbox="./media/how-to-view-alerts/site-manager-overview-alerts-details-status-site-page.png":::

1. If there's an alert, select the status text to open details for a specific site. You can also select the name of the site to open its details.

1. On the resource page for the site, you can view the alert status for each resource within the site. You can see the resource responsible for the most significant status of the top level.

   :::image type="content" source="./media/how-to-view-alerts/site-manager-overview-alerts-details-status-los-angeles.png" alt-text="Screenshot that shows the site detail page with alert status for each resource." lightbox="./media/how-to-view-alerts/site-manager-overview-alerts-details-status-los-angeles.png":::
