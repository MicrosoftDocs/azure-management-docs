---
title: Create and Manage an Azure Arc Site
description: The article describes how to create, view, delete, or modify an Azure Arc site in the Azure portal by using the site manager.
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to
ms.date: 04/18/2024
ms.custom: sfi-image-nochange

#customer intent: "As a site admin, I want to know how to create, modify, and delete sites by using the site manager so that I can efficiently manage and organize geographically related resources within my Azure environment."
---

# Create and manage sites

This article shows you how to create, modify, and delete a site by using the Azure Arc site manager (preview).

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* Azure portal access.
* Internet connectivity.
* A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).

## Open the Azure Arc site manager

In the [Azure portal](https://portal.azure.com/), search for and select **Azure Arc**. On the Azure Arc service menu, select **Site manager (preview)**.

Alternatively, you can search for the Azure Arc site manager directly in the Azure portal. Use terms like **site**, **Arc Site**, and **site manager**.

## Create a site

Create a site to manage geographically related resources.

1. In Azure Arc, on the **Site manager** page, select **Create a site** to open the **Create a Site** page.

   :::image type="content" source="media/how-to-crud-site/create-a-site.png" alt-text="Screenshot that shows how to create a site.":::

1. Provide the following information about your site:

   | Parameter | Description |
   |--|--|
   | Site name | Custom name for a site. |
   | Display name | Custom display name for a site. |
   | Site address| Physical address for a site. Providing the country/region name is mandatory. The street address, city, state/province, and postal code are optional.|
   | Site scope | Select **Subscription**, **Resource group**, or **Service group**. You can define the scope only when you create a site. You can't modify it later. Use the site manager to view and manage all the resources in the scope. |
   | Subscription, resource group, or service group| Select the subscription, resource group, or service group according to the site scope. |

    You can create a hierarchical representation of sites in two ways:
    
    - You can create a two-level hierarchy by mapping the scopes to a subscription as the parent and the resource groups within as child sites.
    - You can create a multilevel hierarchy by using a service group. A site based on a service group scope provides flexibility in resource selection across subscriptions and resource groups. You can define the hierarchy by associating a service group-scoped site with the required service group-scoped parent site.
    
1. Create a service group and associate it with a parent.

   :::image type="content" source="media/how-to-crud-site/create-a-service-group.png" alt-text="Screenshot that shows how to create a service group.":::

1. To select the resources within a service group, select **Add members** and then select the required resources.

   :::image type="content" source="media/how-to-crud-site/select-site-resources.png" alt-text="Screenshot that shows how to select site resources.":::

1. You can associate a site scope with a created (or preexisting) service group to define the site to reflect the service group.

   In the following example, service group ca001 was created as a parent service group. The site named California was associated with the service group ca001 and became the root of the hierarchy. Service groups la001, sc001, sd001, and sf001 were created with the parent service group as ca001. The sites named Los Angeles, Sacramento, San Diego, and San Francisco were associated with the scopes la001, sc001, sd001, and sf001, respectively. They became child sites under the parent site named California.

   For the next hierarchy level, service groups cf001 and nw001 were created under the parent service group la001. The service group ff001 was created under the parent service group sf001. The sites named Contoso Factory and NorthWind Factory were associated with service groups cf001 and nw001, respectively. They became child sites under the parent site named Los Angeles. The site named Fabrikam Factory was associated with service group ff001. It became a child site under the parent site named San Francisco.

   :::image type="content" source="media/how-to-crud-site/site-list.png" alt-text="Screenshot that shows a site list.":::

   Currently, the site manager supports a hierarchy of up to 10 levels.

1. After all these details are provided, select **Review + create**.

1. On the summary page, review and confirm the site details, and then select **Create** to create your site.

If a site is created from a resource group or subscription that contains resources that the site supports, the resources are automatically visible in the created site.

## View and modify a site

After you create a site, you can access it and its managed resources through the site manager.

1. In Azure Arc, on the **Site manager** page, select **Sites** to view all existing sites.

1. On the **Sites** page, you can view all existing sites. Select the name of the site that you want to modify.

1. On a specific site's resource page, you can:

   * View resources.
   * Modify the display name and address.
   * Modify resources (modifications also affect the resources elsewhere).
   * View connectivity status.
   * View update status.
   * View alerts.
   * Add new resources.

Currently, you can modify only some aspects of a site.

| Site attribute | Available modifications |
|--|--|
| Display name | Update the display name of a site to a new unique name. |
| Site address | Update the address of a site. |

## Delete a site

Deleting a site doesn't affect the resources, resource group, subscription, or service group in its scope. After a site is deleted, the resources of that site still exist, but you can't view or manage them from the site manager. You can create a new site for the resource group or the subscription after the original site is deleted.

1. In Azure Arc, on the **Site manager** page, select **Sites** to view all existing sites.

1. On the **Sites** page, you can view all existing sites. Select the name of the site that you want to delete.

1. On the site's resource page, select **Delete**.

   :::image type="content" source="media/how-to-crud-site/delete-a-site-confirmation.png" alt-text="Screenshot that shows how to delete a site confirmation.":::
