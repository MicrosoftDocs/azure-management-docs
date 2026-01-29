---
title: Create and Manage an Azure Arc Site
description: Describes how to create, view, delete, or modify an Azure Arc site in the Azure portal by using the site manager.
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to
ms.date: 04/18/2024
ms.custom: sfi-image-nochange

#customer intent: "As a site admin, I want to know how to create, modify, and delete sites by using a site manager so that I can efficiently manage and organize geographically related resources within my Azure environment."
---

# Create and manage sites

This article guides you through how to create, modify, and delete a site by using the Azure Arc site manager (preview).

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure portal access.
* Internet connectivity.
* A resource group or subscription in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).

## Open the Azure Arc site manager

In the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**. On the Azure Arc service menu, select **Site manager (preview)**.

:::image type="content" source="media/how-to-crud-site/screenshot-azure-arc-portal-view.jpg" alt-text="Screenshot that shows the Azure Arc portal view.":::

Alternatively, you can search for the Azure Arc site manager directly in the Azure portal. Use terms like **site**, **Arc Site**, and **site manager**.

## Create a site

Create a site to manage geographically related resources.

1. In Azure Arc, on the **Site manager** page, select **Create a site**.

   :::image type="content" source="media/how-to-crud-site/screenshot-site-manager-get-started-tab.jpg" alt-text="Screenshot that shows the site manager Get started tab.":::

1. Provide the following information about your site:

   | Parameter | Description |
   |--|--|
   | Site name | Custom name for a site. |
   | Display name | Custom display name for a site. |
   | Site address| Physical address for a site. Providing the country/region name is mandatory. The street address, city, state/province, and postal code are optional.|
   | Site scope | Select either **Subscription** or **Resource group**. You can define the scope only when you create a site. You can't modify it later. Use the site manager to view and manage all the resources in the scope. |
   | Subscription/Resource group | Select the subscription/resource group according to the site scope. |
   | Parent site| Subscription scope sites don't have a parent site. Resource group scope sites can only be a child of the parent subscription scope site.|

1. After you enter all these details, select **Review + Create**.

   :::image type="content" source="media/how-to-crud-site/screenshot-create-site-basics-page.jpg" alt-text="Screenshot that shows the Basics tab on the Create a Site page.":::

   :::image type="content" source="media/how-to-crud-site/screenshot-create-site-scope-page.jpg" alt-text="Screenshot that shows the Site scope tab on the Create a Site page.":::

1. On the summary page, review and confirm the site details, and then select **Create** to create your site.

   :::image type="content" source="media/how-to-crud-site/screenshot-create-site-review-page.jpg" alt-text="Screenshot that shows the Create a Site review page.":::
  
If a site is created from a resource group or subscription that contains resources that the site supports, these resources are automatically visible within the created site.

## View and modify a site

After you create a site, you can access it and its managed resources through the site manager.

1. In Azure Arc, on the **Site manager** page, select **Sites** to view all existing sites.

   :::image type="content" source="media/how-to-crud-site/screenshot-site-manager-sites-tab.jpg" alt-text="Screenshot that shows the Sites tab on the Site manager page.":::

1. On the **Sites** page, you can view all existing sites. Select the name of the site that you want to delete.

   :::image type="content" source="media/how-to-crud-site/screenshot-site-manager-sites-view.jpg" alt-text="Screenshot that shows the Sites view on the Site manager page.":::

1. On a specific site's resource page, you can:

   * View resources.
   * Modify resources (modifications also affect the resources elsewhere).
   * View connectivity status.
   * View update status.
   * View alerts.
   * Add new resources.

Currently, you can modify only some aspects of a site:

| Site attribute | Available modifications |
|--|--|
| Display name | Update the display name of a site to a new unique name. |
| Site address | Update the address of a site. |

## Delete a site

Deleting a site doesn't affect the resources, resource group, or subscription in its scope. After a site is deleted, the resources of that site still exist, but you can't view or manage them from the site manager. You can create a new site for the resource group or the subscription after the original site is deleted.

1. In Azure Arc, on the **Site manager** page, select **Sites** to view all existing sites.

1. On the **Sites** page, you can view all existing sites. Select the name of the site that you want to delete.

1. On the site's resource page, select **Delete**.

   :::image type="content" source="media/how-to-crud-site/screenshot-site-manager-site-details.jpg" alt-text="Screenshot that shows the site details view on the Site manager page.":::