---
title: View Connectivity Status
description: Describes how to view the connectivity status of an Azure Arc site and all of its managed resources through the Azure portal.
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to
ms.date: 04/18/2024

#customer intent: "As a site admin, I want to know how to view the connectivity status of my Azure Arc site and its resources so that I can ensure that everything is functioning properly and address any connectivity issues promptly."
---

# View connectivity status for an Azure Arc site

This article shows how to view the connectivity status for an Azure Arc site. A site's connectivity status reflects the status of the underlying resources. From the site status view, you can also find detailed status information for the supported resources.

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure portal access.
* Internet connectivity.
* A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).
* A site created for the associated resource group or subscription or service group. If you don't have one, see [Create and manage sites](./how-to-crud-site.md).

## Connectivity status colors and meanings

In the Azure portal, the color indicates the status:

* **Green**: Means **Connected**.
* **Yellow**: Means **Not connected recently**.
* **Red**: Means **Needs attention**.

## View connectivity status

You can view connectivity status for an Azure Arc site as a whole from the main page of the Azure Arc site manager (preview).

1. In the [Azure portal](https://portal.azure.com), go to **Azure Arc** and select **Site manager (preview)** to open the site manager.

1. From the Azure Arc site manager, go to the **Overview** page.

   :::image type="content" source="./media/how-to-view-connectivity-status/overview-sites-page.png" alt-text="Screenshot that shows selecting the Overview page in the site manager.":::

1. On the **Overview** page, you can see a summary of the connectivity status for all your sites. The connectivity status of a specific site is an aggregation of the connectivity status of its resources. In the following example, sites are shown with different statuses.

   :::image type="content" source="./media/how-to-view-connectivity-status/site-connection-overview.png" alt-text="Screenshot that shows the connectivity view on the Overview tab for the sites." lightbox="./media/how-to-view-connectivity-status/site-connection-overview.png":::

1. To understand which site has which status, select either the **Sites** tab or the status text to go to the **Sites** page.

   :::image type="content" source="./media/how-to-view-connectivity-status/click-connectivity-status-site-details.png" alt-text="Screenshot that shows selecting the status text to get more detail about connectivity status." lightbox="./media/how-to-view-connectivity-status/click-connectivity-status-site-details.png":::

1. On the **Sites** page, you can view the top-level status for each site. This site-level status reflects the most significant resource-level status for the site.

1. Select the **Needs attention** link to view the resource details.

   :::image type="content" source="./media/how-to-view-connectivity-status/site-connectivity-status-from-sites-page.png" alt-text="Screenshot that shows selecting the connectivity status for a site to see the resource details." lightbox="./media/how-to-view-connectivity-status/site-connectivity-status-from-sites-page.png":::

1. On the resource page for the site, you can view the connectivity status for each resource within the site, including the resource responsible for the top-level most significant status.

   :::image type="content" source="./media/how-to-view-connectivity-status/los-angeles-resource-status-connectivity.png" alt-text="Screenshot that shows using the site details page to identify resources with connectivity issues." lightbox="./media/how-to-view-connectivity-status/los-angeles-resource-status-connectivity.png":::
