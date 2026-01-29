---
title: View Update Status for a Site
description: This article shows you how to view update status for a site.
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to
ms.date: 04/18/2024

#customer intent: "As a site administrator, I want to view the update status for my sites so that I can ensure that all underlying resources are properly maintained and operational."
---

# View update status for an Azure Arc site

This article shows you how to view update status for an Azure Arc site. A site's update status reflects the status of the underlying resources. From the site status view, you can also find detailed status information for the supported resources.

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure portal access.
* Internet connectivity.
* A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).
* A site created for the associated resource group or subscription or service group. If you don't have one, see [Create and manage sites](./how-to-crud-site.md).

## Update status colors and meanings

In the Azure portal, the color indicates the status:

* **Green**: Means **Up to date**.
* **Blue**: Means **Update available**.
* **Yellow**: Means **Update in progress**.
* **Red**: Means **Needs attention**.

This update status comes from the resources within each site and is provided by Azure Update Manager.

## View update status

You can view update status for an Azure Arc site as a whole from the main page of the Azure Arc site manager (preview).

1. In the [Azure portal](https://portal.azure.com), go to **Azure Arc** and select **Site manager (preview)** to open the site manager.

1. From the Azure Arc site manager, go to the **Overview** page.

   :::image type="content" source="./media/how-to-view-update-status/overview-sites-page.png" alt-text="Screenshot that shows selecting the Overview page in the site manager.":::

1. On the **Overview** page, you can view the summarized update statuses for your sites. This site-level status is aggregated from the statuses of its managed resources. In the following example, sites are shown with different statuses.

   :::image type="content" source="./media/how-to-view-update-status/site-manager-update-status-overview-page.png" alt-text="Screenshot that shows the update status summary on the Overview page." lightbox="./media/how-to-view-update-status/site-manager-update-status-overview-page.png":::

1. To understand which site has which status, select either the **Sites** tab or the status text to go to the **Sites** page.

   :::image type="content" source="./media/how-to-view-update-status/click-update-status-site-details.png" alt-text="Screenshot that shows selecting the Sites tab to get more detail about update status." lightbox="./media/how-to-view-update-status/click-update-status-site-details.png":::

1. On the **Sites** page, you can view the top-level status for each site. This site-level status reflects the most significant resource-level status for the site.

1. Select the **Needs attention** link to view the resource details.

   :::image type="content" source="./media/how-to-view-update-status/site-update-status-from-sites-page.png" alt-text="Screenshot that shows selecting the update status for a site to see the resource details." lightbox="./media/how-to-view-update-status/site-update-status-from-sites-page.png" :::

1. On the resource page for the site, you can view the update status for each resource within the site, including the resource responsible for the top-level most significant status.

   :::image type="content" source="./media/how-to-view-update-status/los-angeles-resource-status-updates.png" alt-text="Screenshot that shows using the site details page to identify resources with pending or in-progress updates." lightbox="./media/how-to-view-update-status/los-angeles-resource-status-updates.png" :::
