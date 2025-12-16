---
title: Create a dashboard in the Azure portal
description: This article describes how to create and customize a dashboard in the Azure portal.
ms.topic: how-to
ms.date: 10/04/2024
# Customer intent: As an Azure user, I want to create and customize dashboards in the Azure portal so that I can efficiently monitor resources and facilitate quick access to tasks related to my projects and organizational roles.
---

# Create a dashboard in the Azure portal

Dashboards provide a focused and organized view of your cloud resources in the Azure portal. Use dashboards as a workspace where you can monitor resources and quickly launch tasks for day-to-day operations. For example, you can build custom dashboards based on projects, tasks, or user roles in your organization.

> [!IMPORTANT]
> This article describes the older dashboard experience, which you can still access. If you're using the improved dashboard editing experience, see [Create and manage dashboards in Dashboard hub (preview)](./dashboard-hub.md).

The Azure portal provides a default dashboard as a starting point. You can edit this default dashboard, and you can create and customize additional dashboards.

All dashboards are private when created, and each user can create up to 100 private dashboards. If you [publish and share a dashboard with other users in your organization](azure-portal-dashboard-share-access.md), the shared dashboard is implemented as an Azure resource in your subscription and doesn't count towards the private dashboard limit. 

## Create a new dashboard

This example shows how to create a new private dashboard with an assigned name.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. From the Azure portal menu, select **Dashboard**. Your default view might already be set to dashboard.

    :::image type="content" source="media/azure-portal-dashboards/portal-menu-dashboard.png" alt-text="Screenshot of the Azure portal with Dashboard selected.":::

1. Select **Create**, and then select **Custom**.

    This action opens the **Tile Gallery**, from which you can select tiles that display different types of information. You also see an empty grid representing the dashboard layout, where you can arrange the tiles.

1. Select the text in the dashboard label and enter a name that helps you easily identify the custom dashboard.

    :::image type="content" source="media/azure-portal-dashboards/dashboard-name.png" alt-text="Screenshot of an empty grid with the Tile Gallery.":::

1. To save the dashboard, select **Save** in the page header.

The dashboard view now shows your new dashboard. Select the arrow next to the dashboard name to see other available dashboards. The list might include dashboards that other users created and shared.

> [!TIP]
> If you have an existing dashboard and want to create a new one that's similar, you can [clone your dashboard](#clone-a-dashboard) and use the duplicate copy as a starting point.

## Edit a dashboard

Now, you can edit the example dashboard you created to add, resize, and arrange tiles that show your Azure resources or display other helpful information. Start by working with the Tile Gallery, and then explore other ways to customize dashboards.

### Add tiles from the Tile Gallery

To add tiles to a dashboard by using the Tile Gallery, follow these steps.

1. Select **Edit** from the dashboard's page header.

    :::image type="content" source="media/azure-portal-dashboards/dashboard-edit.png" alt-text="Screenshot of dashboard highlighting the Edit option.":::

1. Browse the **Tile Gallery** or use the search field to find a certain tile. Select the tile you want to add to your dashboard.

   :::image type="content" source="media/azure-portal-dashboards/dashboard-tile-gallery.png" alt-text="Screenshot of the Tile Gallery.":::

1. Select **Add** to add the tile to the dashboard with a default size and location. Or, drag the tile to the grid and place it where you want. 

1. Select **Save** to save your changes. You can also preview the changes without saving by selecting **Preview**. This preview mode also allows you to see how [filters](#apply-dashboard-filters) affect your tiles. From the preview screen, you can select **Save** to keep the changes, **Cancel** to remove them, or **Edit** to go back to the editing options and make further changes.

### Resize or rearrange tiles

To change the size of a tile, or to rearrange the tiles on a dashboard, follow these steps:

1. Select **Edit** from the page header.

1. Select the context menu in the upper right corner of a tile. Then, choose a tile size. Tiles that support any size also include a "handle" in the lower right corner that you can use to drag the tile to the size you want.

    :::image type="content" source="media/azure-portal-dashboards/dashboard-tile-resize.png" alt-text="Screenshot of dashboard with tile size menu open.":::

1. Select a tile and drag it to a new location on the grid to arrange your dashboard.

1. When you're finished, select **Save**.

### Pin content from a resource page

You can add some tiles to your dashboard directly from a resource page.

Many resources and services include a pin icon in the page header. By using this icon, you can easily pin a tile that represents the resource or service to a dashboard.

:::image type="content" source="media/azure-portal-dashboards/dashboard-pin-icon.png" alt-text="Screenshot of page command bar with pin icon.":::

Select this icon to pin the tile to an existing private or shared dashboard. You can also create a new dashboard that includes this tile by selecting **Create new**.

:::image type="content" source="media/azure-portal-dashboards/dashboard-pin-pane.png" alt-text="Screenshot of Pin to dashboard options.":::

### Copy a tile to a new dashboard

If you want to reuse a tile on a different dashboard, you can copy it from one dashboard to another. To do so, select the context menu in the upper right corner and then select **Copy**.

You can then select whether to copy the tile to a different private or shared dashboard, or create a copy of the tile within the dashboard you're already working in. You can also create a new dashboard that includes a copy of the tile by selecting **Create new**.

## Modify tile settings

> [!IMPORTANT]
> This article describes the older dashboard experience, which you can still access. If you're using the improved dashboard editing experience, see [Create and manage dashboards in Dashboard hub (preview)](./dashboard-hub.md).

Some tiles require more configuration to show the information you want. For example, the **Metrics chart** tile needs to be set up to display a metric from Azure Monitor. You can also customize tile data to override the dashboard's default time settings and filters, or to change the title and subtitle of a tile.

> [!NOTE]
> The **Markdown** tile lets you display custom, static content on your dashboard. This content can be any information you provide, such as basic instructions, an image, a set of hyperlinks, or even contact information. For more information about using markdown tiles, see [Use a markdown tile on Azure dashboards to show custom content](azure-portal-markdown-tile.md).

### Change the title and subtitle of a tile

Some tiles let you edit their title and subtitle. To update the title or subtitle, select **Configure tile settings** from the context menu.

:::image type="content" source="media/azure-portal-dashboards/dashboard-tile-rename.png" alt-text="Screenshot showing the Configure tile settings option.":::

Make your changes, and then select **Apply**.

### Complete tile configuration

Any tile that requires configuration displays a banner until you customize the tile. For example, in the **Metrics chart**, the banner reads **Edit in Metrics**. Other banners might use different text, such as **Configure tile**.

To customize the tile:

1. If needed, select **Save** or **Cancel** near the top of the page to exit edit mode.

1. Select the banner, and then make selections to customize the tile.

    :::image type="content" source="media/azure-portal-dashboards/dashboard-configure-tile.png" alt-text="Screenshot of a tile that requires configuration.":::

### Apply dashboard filters

Near the top of your dashboard, you'll see options to set the **Auto refresh** and **Time settings** for data displayed in the dashboard, along with an option to add additional filters.

:::image type="content" source="media/azure-portal-dashboards/dashboard-global-filters.png" alt-text="Screenshot showing a dashboard's global filters.":::

To change how often data is refreshed, select **Auto refresh**, and then choose a new refresh interval. After you make your selection, select **Apply**.

The default time settings are **UTC Time**, showing data for the **Past 24 hours**. To change this setting, select the button and choose a new time range, time granularity, and/or time zone. Then select **Apply**.

To apply more filters, select **Add filter** . The options you see vary depending on the tiles in your dashboard. For example, you might see options to filter data for a specific subscription or location. In some cases, you might not see any additional filters.

If you want to use a new filter option, select it, and then make your selections. The filter is then applied to your data.

To remove a filter, select the **X** in its button.

### Override dashboard filters for specific tiles

Tiles that support filtering have a ![filter icon](./media/azure-portal-dashboards/dashboard-filter.png) filter icon in the top-left corner of the tile. These tiles allow you to override the global filters with filters specific to that tile.

To override global filters, select **Configure tile settings** from the tile's context menu, or select the filter icon. Then you can change the desired filters for that tile. For example, some tiles provide an option to override the dashboard time settings at the tile level,  allowing you to select a different time span to refresh data.

When you apply filters for a particular tile, the left corner of that tile changes to show a double filter icon, indicating that the data in that tile reflects its own filters.

:::image type="content" source="media/azure-portal-dashboards/dashboard-filter-override.png" alt-text="Screenshot showing the icon for a tile with a filter override.":::

## Delete a tile

To remove a tile from a dashboard, use one of the following methods:

- Select the context menu in the upper right corner of the tile, and then select **Remove from dashboard**.

- Select **Edit** to enter customization mode. Hover in the upper right corner of the tile, and then select the ![delete icon](./media/azure-portal-dashboards/dashboard-delete-icon.png) delete icon to remove the tile from the dashboard.

## Clone a dashboard

To use an existing dashboard as a template for a new dashboard, follow these steps:

1. Make sure that the dashboard view is showing the dashboard that you want to copy.

1. In the page header, select ![clone icon](./media/azure-portal-dashboards/dashboard-clone.png) **Clone**.

1. A duplicate copy of the dashboard, named **Clone of (your dashboard name)**, opens in edit mode. You can then rename and customize the new dashboard.

## Use shared dashboards

When you create a dashboard, it's private by default, which means you're the only one who can see it. To make dashboards available to others, you can publish and share them. For more information, see [Share Azure dashboards by using Azure role-based access control](azure-portal-dashboard-share-access.md).

### Open a shared dashboard

To find and open a shared dashboard, follow these steps.

1. Select the arrow next to the dashboard name.

1. Select from the displayed list of dashboards. If the dashboard you want to open isn't listed:

    1. Select **Browse all dashboards**.

        :::image type="content" source="media/azure-portal-dashboards/dashboard-browse.png" alt-text="Screenshot of dashboard selection menu.":::

    1. Select the **Type equals** filter, and then select **Shared dashboard**.

        :::image type="content" source="media/azure-portal-dashboards/dashboard-browse-all.png" alt-text="Screenshot of all dashboards selection menu.":::

    1. Select a dashboard from the list of shared dashboards. If you don't see the one you want, use the filters to limit the results shown, such as selecting a specific subscription or filtering by name.

## Delete a dashboard

You can delete your private dashboards, or a shared dashboard that you created or have permissions to modify.

To permanently delete a private or shared dashboard, follow these steps.

1. Select the dashboard you want to delete from the list next to the dashboard name.

1. Select ![delete icon](./media/azure-portal-dashboards/dashboard-delete-icon.png) **Delete** from the page header.

1. For a private dashboard, select **OK** on the confirmation dialog to remove the dashboard. For a shared dashboard, on the confirmation dialog, select the checkbox to confirm that the published dashboard will no longer be viewable by others. Then, select **OK**.

## Next steps

- [Share Azure dashboards by using Azure role-based access control](azure-portal-dashboard-share-access.md)
- [Programmatically create Azure dashboards](azure-portal-dashboards-create-programmatically.md)
