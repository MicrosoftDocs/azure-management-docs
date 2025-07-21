---
title: Recover a deleted dashboard in the Azure portal
description: If you delete a published dashboard in the Azure portal, you can recover the dashboard.
ms.date: 10/10/2024
ms.topic: how-to
# Customer intent: "As a cloud administrator, I want to recover a deleted published dashboard within seven days, so that I can restore important visualizations and data tracking without losing shared insights."
---

# Recover a deleted dashboard in the Azure portal

If you're in the global Azure cloud, and you delete a _published_ (shared) dashboard in the Azure portal, you can recover that dashboard within seven days of the delete.

> [!IMPORTANT]
> If you're in an Azure Government cloud, or if the dashboard isn't published, you can't recover a deleted dashboard.

Follow these steps to recover a published dashboard:

1. From the Azure portal menu, select **Resource groups**, then select the resource group where you published the dashboard. (The default resource group is named **dashboards**.)

1. Under **Activity log**, expand the **Delete Dashboard** operation, then select the **Delete Dashboard** item underneath it. If you don't see this operation, try changing the **Timespan** filter to a longer duration.

1. Select the [**Change history** tab](/azure/azure-monitor/change/change-analysis-visualizations##view-the-activity-log-change-history), then select **\<deleted resource\>**.

   :::image type="content" source="media/recover-shared-deleted-dashboard/change-history-tab.png" alt-text="Screenshot showing the Change history tab for a deleted dashboard in the Azure portal.":::

1. In the **Old value** pane, select and copy the full contents. Paste this content into a text file saved with a _.json_ file extension. The portal can use this JSON file to recreate the dashboard.

    :::image type="content" source="media/recover-shared-deleted-dashboard/change-history-diff.png" alt-text="Screenshot of the JSON representation of a deleted dashboard in the Azure portal.":::

1. From the Azure portal menu, select **Dashboard**, then select **Upload**.

    :::image type="content" source="media/recover-shared-deleted-dashboard/dashboard-upload.png" alt-text="Screenshot of the Upload dashboard option in the Azure portal.":::

1. Select the JSON file you saved. The portal recreates the dashboard with the same name and elements as the deleted dashboard.

1. Select **Share** to [publish the dashboard and reestablish the appropriate access control](azure-portal-dashboard-share-access.md).

    :::image type="content" source="media/recover-shared-deleted-dashboard/dashboard-share.png" alt-text="Screenshot of the Share option for dashboards in the Azure portal.":::
