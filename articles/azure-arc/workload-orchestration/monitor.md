---
title: Monitor your Solutions with Workload Orchestration Portal
description: Learn how to navigate the workload orchestration portal to monitor your solutions, including their status, capabilities, and deployment details.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 09/05/2025
ms.custom:
  - build-2025
# Customer intent: As a solutions architect, I want to monitor the status and capabilities of my deployed solutions in a centralized portal, so that I can ensure optimal performance and quickly address any deployment issues.
---

# Monitor your solutions with the workload orchestration portal

Workload orchestration portal provides a user-friendly interface for monitoring your solutions. In the Monitor tab you can view your solutions, including their status, capabilities, and deployment details. 

This article describes how to use the workload orchestration portal to monitor your solutions. If you want information about other tabs in the workload orchestration portal, see [Configure your solutions](configure.md) and [Deploy your solutions](deploy.md).

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- All RBAC-enabled users can access the Monitor tab. If you don't see a tab or a feature, it might be due to insufficient permissions. Contact your IT administrator to ensure you have the necessary access.

## Navigate the Solutions tab 

The Monitor tab provides two sub-tabs: **Solutions** and **Targets**. The Solutions tab shows the status of all solutions deployed in your environment, while the Targets tab shows the status of all targets.

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview).
1. Once you sign in, you see the **Monitor** tab on the left side of the page. 
1. Select the **Solutions** sub-tab, which lists all solutions including their name, version, status, and other details.

    :::image type="content" source="./media/monitor-solutions.png" alt-text="Screenshot of the Solutions tab in workload orchestration portal showing the default view." lightbox="./media/monitor-solutions.png":::

1. In the Solutions tab, the default view lists all solution versions grouped by solution name. You can click on **Group by: Name** on the top right of the list to select a different view.

    :::image type="content" source="./media/monitor-group-by.png" alt-text="Screenshot of the Monitor tab showing how to group the solutions." lightbox="./media/monitor-group-by.png":::

1. The Solutions tab includes **status tiles** on the top of the page that show the total number of solutions and the number of solutions in each state, which are *Configuration pending*, *Ready to deploy*, *Deployed (Update available)*, *Deployed (Updating)* *Deploying*, *Deployed*, *Deployment failure*, and *Previously deployed*. You can select any of the status tiles to filter the list of solutions by that status.

     :::image type="content" source="./media/monitor-filter-by-status.png" alt-text="Screenshot of the Monitor tab showing how to filter the solutions by status." lightbox="./media/monitor-filter-by-status.png":::

1. You can filter the list of solutions by keyword. To do this, enter the keyword in the **search box** on the top right of the list next to "Group by:". The list of solutions will be filtered to show only those that match the keyword.

    :::image type="content" source="./media/monitor-keyword.png" alt-text="Screenshot of the Monitor tab showing how to search by keyword." lightbox="./media/monitor-keyword.png":::

1. You can apply filters to the list of solution. To do this, click on **Filter** on the top right of the list and select the **Column** you want to filter by, for example *Name*. Then select  an **Operator**, for example, *Equals*, and enter the **Value** you want to filter by. The list of solutions will be filtered to show only those that match the selected column and value.

    :::image type="content" source="./media/monitor-apply-filters.png" alt-text="Screenshot of the Monitor tab showing how to apply filters." lightbox="./media/monitor-apply-filters.png":::

### View the details of a deployment failure

If the deployment of a solution fails, the status column shows **Deployment failure**. 

1. You can click on the **status** to view the details of the failure. 

    :::image type="content" source="./media/monitor-failure.png" alt-text="Screenshot of the Monitor tab showing how to click on a failed deployment." lightbox="./media/monitor-failure.png":::

1. The new details pane shows the failure reason, fired time, and affected target.

    :::image type="content" source="./media/monitor-failure-2.png" alt-text="Screenshot of the Monitor tab showing how to view the details of a failed deployment." lightbox="./media/monitor-failure-2.png":::

1. Note that deployment failure shows **aggregated status** for each target, which means that if a solution has multiple targets, the status will be shown as failure even if only one of the targets has failed. 

    :::image type="content" source="./media/monitor-failure-targets.png" alt-text="Screenshot of the Monitor tab showing the number of published versus deployed targets when a solution fails." lightbox="./media/monitor-failure-targets.png":::

1. Click on the **solution name** to see which targets failed.

    :::image type="content" source="./media/monitor-failure-targets-2.png" alt-text="Screenshot of the Monitor tab showing to click on the solution name to view the details of the failure." lightbox="./media/monitor-failure-targets-2.png":::

### View the capabilities of a solution

1. You can view the capabilities of each solution by clicking on **tags** in the **Capabilities** column. 

    :::image type="content" source="./media/monitor-capabilities.png" alt-text="Screenshot of the Monitor tab showing how to view the capabilities of each solution." lightbox="./media/monitor-capabilities.png":::

1. This action opens a new pane that shows the capabilities of the solution.

    :::image type="content" source="./media/monitor-capabilities-2.png" alt-text="Screenshot of the new pane showing how to view the capabilities of each solution." lightbox="./media/monitor-capabilities-2.png":::

### View the details of a solution

You can view the detailed revision of a solution  if the solution state is either Deployed, ReadyToDeploy, or Stopped.

1. Click on **Solution name** from the list view. 

    :::image type="content" source="./media/monitor-revision.png" alt-text="Screenshot of the Monitor tab showing how to view the detailed revision of a solution." lightbox="./media/monitor-revision.png":::

1. In the details view of the solution, click on the **Solution name** for the particular target to see the revision.

    :::image type="content" source="./media/monitor-revision-2.png" alt-text="Screenshot of the Monitor tab showing how to choose a particular target to view the detailed solution." lightbox="./media/monitor-revision-2.png":::

1. The new pane displays details of revision and list of final configurations.

    :::image type="content" source="./media/monitor-revision-3.png" alt-text="Screenshot of the new pane showing how to view the detailed revision of a solution." lightbox="./media/monitor-revision-3.png":::

## Navigate the Targets tab

1. On the **Monitor** tab, select the **Targets** sub-tab, which lists all solutions including their name, version, status, and other details.

    :::image type="content" source="./media/monitor-targets.png" alt-text="Screenshot of the Targets tab in the Monitor tab portal." lightbox="./media/monitor-targets.png":::

1. The Targets tab includes **status tiles** on the top of the page that show the total number of targets and the number of targets in each state, which are *Ready to deploy*, *Deployed (Update available)*, *Deployed (Updating)* *Deploying*, *Deployed*, *Deployment failure*, *Previously deployed*, and *No solution available*. You can select any of the status tiles to filter the list of targets by that status.
1. All capabilities are selected by default. You can select or deselect the capabilities to filter the list of targets by that capability.

    :::image type="content" source="./media/monitor-target-capabilities.png" alt-text="Screenshot of the Targets tab showing how to filter the targets by capabilities." lightbox="./media/monitor-target-capabilities.png":::

1. In the Targets tab, the default view lists all targets ungrouped. You can click on **Group by: None** on the top right of the list to select a different view.

    :::image type="content" source="./media/monitor-target-group-by.png" alt-text="Screenshot of the Target tab showing how to group the solutions." lightbox="./media/monitor-target-group-by.png":::

1. You can filter the list of targets by keyword. To do this, enter the keyword in the **search box** on the top right of the list next to "Group by:". The list of targets will be filtered to show only those that match the keyword.

1. You can apply filters to the list of target. To do this, click on **Filter** on the top right of the list and select the **Column** you want to filter by, for example *Hierarchy Level*. Then select  an **Operator**, for example, *Equals*, and enter the **Value** you want to filter by. The list of solutions will be filtered to show only those that match the selected column and value.

    :::image type="content" source="./media/monitor-target-apply-filters.png" alt-text="Screenshot of the Target tab showing how to apply filters." lightbox="./media/monitor-target-apply-filters.png":::

### View the details of a target

You can view the detailed revision of a target if it has a solution available.

1. Click on **Target name** from the list view. 

    :::image type="content" source="./media/monitor-target-revision.png" alt-text="Screenshot of the Target tab showing how to view the detailed revision of a solution." lightbox="./media/monitor-target-revision.png":::

1. In the details view of the target, click on the **Target name** to see the revision.

    :::image type="content" source="./media/monitor-target-revision-2.png" alt-text="Screenshot of the Target tab showing how to choose a particular target to view the detailed solution." lightbox="./media/monitor-target-revision-2.png":::

1. The new pane displays details of revision and list of final configurations.

    :::image type="content" source="./media/monitor-target-revision-3.png" alt-text="Screenshot of the new pane showing how to view the detailed revision of a target." lightbox="./media/monitor-target-revision-3.png":::

## Navigate the Event log tab

1. The **Event log** feature offers a detailed, chronological view of all user activities. It allows users to track actions performed across solutions and targets, providing comprehensive visibility into deployment history and associated operations.

    :::image type="content" source="./media/monitor-event-log-1.png" alt-text="Screenshot of the event log tab showing all user activities." lightbox="./media/monitor-event-log-1.png":::

1. The list can be grouped by solution, revision or target name by clicking on **Group by** and selecting the desired option.

    :::image type="content" source="./media/monitor-event-log-2.png" alt-text="Screenshot of the event log tab grouped by solution." lightbox="./media/monitor-event-log-2.png":::

1. You can filter the logs by solution name by directly clicking on the **Solution** filter, and by target and revision by clicking on **Filter** on the top right of the list, selecting the **Column** you want to filter by, and the desired **Operator**.

    :::image type="content" source="./media/monitor-event-log-3.png" alt-text="Screenshot of the event log tab showing filter options." lightbox="./media/monitor-event-log-3.png":::

1. To view the detailed status of any operation, click on the corresponding link under **Event Name** column to open the side pane showing all the intermediate steps of that operation, along with timestamp and the user who initiated it.

    :::image type="content" source="./media/monitor-event-log-4.png" alt-text="Screenshot of the event log tab showing example event details." lightbox="./media/monitor-event-log-4.png":::

## Monitor solutions with external validation enabled

If you have enabled external validation for workload orchestration, you see two new statuses displayed in the Solutions and Targets tabs - *Publish in progress* and *Publish failed*. You can click on the status tiles to filter by the status to view the solutions. 

:::image type="content" source="./media/external-validation-monitor.png" alt-text="Screenshot of the monitor tab showing two new statuses when you enable external validation." lightbox="./media/external-validation-monitor.png":::

For more information, see [External validation for workload orchestration](external-validation.md).
