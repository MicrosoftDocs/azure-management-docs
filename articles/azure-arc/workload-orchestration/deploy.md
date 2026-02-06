---
title: Deploy your Solutions with Workload Orchestration Portal
description: Learn to use workload orchestration portal to deploy your applications, and also to delete, stop, roll back, and retry solutions.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 10/27/2025
ms.custom:
  - build-2025
# Customer intent: As a deployment manager, I want to utilize the workload orchestration portal to deploy, manage, and troubleshoot application solutions, so that I can ensure seamless application operations within my environment.
---

# Deploy your solutions with workload orchestration portal

The Deploy tab in the workload orchestration portal displays the targets and the solutions applicable to the targets. It shows targets created at multiple hierarchical levels such as factory and line. 

This article describes how to use the workload orchestration portal to deploy, delete, stop, roll back, and retry solutions. If you want information about other tabs in the workload orchestration portal, see [Monitor your solutions](monitor.md) and [Configure your solutions](configure.md).


## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- Only users with access to deploy solutions can see the Deploy tab. If you don't see a tab or a feature, it might be due to insufficient permissions. Contact your IT administrator to ensure you have the necessary access.

## Navigate the Deploy tab

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview).
1. Once you sign in, you see the **Deploy** tab on the left side of the page.
1. Select the **Deploy** tab, which lists all targets created in your environment. The Deploy tab shows the target name, type, and other details.
1. You can filter the list by keyword. To do this, enter the keyword in the **search box** on the top right of the list next to "Group by:". The list of solutions will be filtered to show only those that match the keyword.

    :::image type="content" source="./media/deploy-keyword.png" alt-text="Screenshot of the Deploy tab showing how to search by keyword." lightbox="./media/deploy-keyword.png":::

1. You can also apply filters to the list of solutions. To do this, click on **Filter** on the top right of the list and select the **Column** you want to filter by. Then select an **Operator** and enter the **Value** you want to filter by. The list of solutions will be filtered to show only those that match the selected column and value.

    :::image type="content" source="./media/deploy-filter-by.png" alt-text="Screenshot of the Deploy tab showing how to apply filters." lightbox="./media/deploy-filter-by.png":::

## Deploy a solution

You can deploy a solution to a target by following these steps:

1. Click on the **Deploy** tab and click on the target you want to deploy the solution to.
:::image type="content" source="./media/single-deploy-1.png" alt-text="Screenshot of the Deploy tab showing how to click on a target." lightbox="./media/single-deploy-1.png":::

1. From the list of solutions applicable on that target, select any solution in **Publish completed** state and click on **Deploy Solution**.
:::image type="content" source="./media/single-deploy-2.png" alt-text="Screenshot of the Deploy tab showing how to click on a target1." lightbox="./media/single-deploy-2.png":::

1. In the confirmation window, click on **Confirm** to proceed.

    :::image type="content" source="./media/single-deploy-3.png" alt-text="Screenshot of the Deploy tab showing how to confirm the deployment of a solution." lightbox="./media/single-deploy-3.png":::

1. You can see a notification of deployment in progress at the top right corner of the page. 

    :::image type="content" source="./media/single-deploy-4.png" alt-text="Screenshot of the Deploy tab showing the notification of a deployment in progress1." lightbox="./media/single-deploy-4.png":::

1. You can click on the status of the solution you deployed, to open the **Status details** pane showing all the intermediate steps of the operation, along with date and time of completion and the user who initiated it. Details of shared app dependencies associated with the current deployment, if any, also show up on this side-pane.

    :::image type="content" source="./media/single-deploy-5.png" alt-text="Screenshot of the Deploy tab showing the deployment status details1." lightbox="./media/single-deploy-5.png":::

## Deploy a solution to multiple targets

You can deploy a solution to multiple targets at once by following these steps:

1. Click on the **Deploy** tab and switch to solution view to view the applicable solutions and their statuses.

    :::image type="content" source="./media/deploy-1.png" alt-text="Screenshot of the Deploy tab showing how to click on a target2." lightbox="./media/deploy-1.png":::

1. You can filter or group solutions by any of the column values. Click on a solution name which has 1 or more targets available to deploy to.

    :::image type="content" source="./media/deploy-2.png" alt-text="Screenshot of the Deploy tab showing how to click on a target3." lightbox="./media/deploy-2.png":::

1. Choose the targets in **Publish Completed** state and click on **Deploy Solution**.

    :::image type="content" source="./media/deploy-3.png" alt-text="Screenshot of the Deploy tab showing how to deploy a solution." lightbox="./media/deploy-3.png":::

1. In the confirmation window, click on **Confirm** to proceed.

    :::image type="content" source="./media/deploy-4.png" alt-text="Screenshot of the Deploy tab showing how to confirm the deployment of a solution1." lightbox="./media/deploy-4.png":::

1. You can see a notification of deployment in progress at the top right corner of the page. 

    :::image type="content" source="./media/deploy-5.png" alt-text="Screenshot of the Deploy tab showing the notification of a deployment in progress." lightbox="./media/deploy-5.png":::

1. To view the detailed status of your deployment, click on the notification icon at the top right and click on **Show in event logs**.

    :::image type="content" source="./media/deploy-6.png" alt-text="Screenshot of the Deploy tab showing the deployment status." lightbox="./media/deploy-6.png":::

1. Click on the respective **Event name** to open the **Status details** pane showing all the intermediate steps of the operation, along with date and time of completion and the user who initiated it. Details of shared app dependencies associated with the current deployment, if any, also show up on this side-pane.

    :::image type="content" source="./media/deploy-7.png" alt-text="Screenshot of the Deploy tab showing the deployment status details." lightbox="./media/deploy-7.png":::


## Roll back a solution

You can undo a deployment and roll back a solution to a previous version. To do this, follow these steps:

1. Select a solution and click on **Rollback**.

    :::image type="content" source="./media/deploy-rollback-1.png" alt-text="Screenshot of the Deploy tab showing how to roll back a solution." lightbox="./media/deploy-rollback-1.png":::

1. This opens the confirmation window displaying the details of the failed version and the list of old versions. Choose the version to roll back and click **Apply**.

    :::image type="content" source="./media/deploy-rollback-2.png" alt-text="Screenshot of the Deploy tab showing how to roll back to a specific version." lightbox="./media/deploy-rollback-2.png":::

1. Once the deployment finishes, the status of the solution is updated to **Deployment Completed**.

## Retry a failed deployment

If a deployment fails, you can retry the deployment by following these steps:

1. Choose the solution version with status **Deployment failure** and click on **Retry deployment**.

    :::image type="content" source="./media/deploy-retry.png" alt-text="Screenshot of the Deploy tab showing how to retry a failed deployment." lightbox="./media/deploy-retry.png":::

1. In the confirmation window, click on **Confirm** to proceed.

## Stop a deployed solution

You can stop a deployed solution. This action is useful if you want to stop the solution without deleting it. To stop a solution, follow these steps:

1. Choose the solution version to be un-deployed and click **Stop**.

    :::image type="content" source="./media/deploy-stop.png" alt-text="Screenshot of the Deploy tab showing how to stop a deployed solution." lightbox="./media/deploy-stop.png":::

1. In the confirmation window, click on **Confirm** to proceed.

## Delete a solution

You can delete a solution before it's deployed, that is, if its status is *Stopped*, *Publish Completed*, *Uninstallation Completed*, or *Deployment failure*. To delete a solution, follow these steps:

1. Choose the solution version with status **Stopped/Publish Completed/Uninstallation Completed/Deployment failure** and click on **Delete**.

    :::image type="content" source="./media/deploy-delete.png" alt-text="Screenshot of the Deploy tab showing how to delete a solution before it's deployed." lightbox="./media/deploy-delete.png":::

1. In the confirmation window, click on **Confirm** to proceed.
