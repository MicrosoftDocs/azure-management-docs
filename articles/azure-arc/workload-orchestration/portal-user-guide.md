---
title: User Guide for the Workload Orchestration Portal
description: Learn how to navigate the workload orchestration portal to monitor, configure, and deploy solutions without writing code.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 10/27/2025
---

# User guide for the workload orchestration portal

This guide is designed for low code/non-code users who want to manage solutions in the workload orchestration portal. 

The [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview) provides a user-friendly interface with three main tabs on the left side: **Monitor**, **Configure**, and **Deploy**. Each tab has its own set of features and functionalities that help you manage your solutions effectively.

|Tab|Actions|
|----|-------|
|[Configure](#configure-your-solutions)|Set up the parameters of your solutions.|
|[Deploy](#deploy-your-solutions)|Deploy your solutions to the right targets and rollback to earlier versions if needed.|
|[Monitor](#monitor-your-solutions)|Track the status of your solutions.|

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- Each tab provides different functionalities and access levels based on the Role Based Access Control (RBAC) assigned to the user. If you don't see a tab or a feature, it might be due to insufficient permissions. Contact your IT administrator to ensure you have the necessary access.

## Configure your solutions

In the [Configure tab](configure.md), you can:

- Set up configuration values for factories and lines.
- Link solutions to targets.
- Publish configurations for deployment.

> [!NOTE]
> Line and factory levels are custom-defined by the IT admin. You may see different names or levels based on your organization's setup.

### User Scenario: Configuring a factory 

Goal: Configure the parameters of a factory level to prepare it for deployment.

Steps:

1. Go to the **Configure Hierarchy** tab.
1. Select the **Factory** subtab.

    :::image type="content" source="./media/configure-line.png" alt-text="Screenshot of the Factory tab in workload orchestration portal showing the default view." lightbox="./media/configure-line.png":::

1. Select the name of the factory you want to configure and click on **Configure**. 

    :::image type="content" source="./media/configure-line-1.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to select a line." lightbox="./media/configure-line-1.png":::

    > [!NOTE]
    > You can update the configuration of a target if its status is not *Configuration up to date*. 

1. In case there is a single common configuration template linked to the hierarchy level, you can directly configure the values. In case of multiple templates, select the templates to be configured from the **Select Version** screen and click on **Next**. You can also filter templates by Name, Version and Status.

:::image type="content" source="./media/configure-line-2.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to select a line-1." lightbox="./media/configure-line-2.png":::

1. In the **Configure** step, enter the parameters for each template and click on **Next**. You can choose to autofill values from previous version only if configuration template was previously configured.

    :::image type="content" source="./media/configure-line-3.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to enter the parameters to configure a target." lightbox="./media/configure-line-3.png":::

1. Review the details and click on **Confirm** to apply the changes.

    :::image type="content" source="./media/configure-line-4.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to review and apply the changes of the configuration." lightbox="./media/configure-line-4.png":::

### Other things you can do in the Configure tab

In the Configure tab, you can also:

|Link|Action	|
|---------------------|------------------|
|[Configure solution parameters](configure.md#configure-solution-parameters)|Configure the parameters of a solution and choose the target to publish it.|
|[Configure solution parameters for a solution with dependencies](configure.md#configure-solution-parameters-for-a-solution-with-dependencies)|Configure the parameters of a solution which is dependent on another solution.|
|[Resolve a solution failure during authoring](configure.md#resolve-a-solution-failure-during-authoring)|Resolve the failure of a solution that fails during the configuration process.|
|[Publish a solution to more targets](configure.md#publish-a-solution-to-more-targets)|Add more targets to an existing solution without creating a new revision|

## Deploy your solutions

In the [Deploy tab](deploy.md), you can:

- Deploy solutions to specific targets.
- Rollback, retry, stop, and delete deployments.

### User Scenario: Deploying a solution to a target

Goal: Deploy a solution to a target.

Steps:

1. Click on the **Deploy** tab and switch to solution view to view the applicable solutions and their statuses.

    :::image type="content" source="./media/deploy-1.png" alt-text="Screenshot of the Deploy tab showing how to click on a target." lightbox="./media/deploy-1.png":::

1. You can filter or group solutions by any of the column values. Click on a solution name which has 1 or more targets available to deploy to.

    :::image type="content" source="./media/deploy-2.png" alt-text="Screenshot of the Deploy tab showing how to click on a target-1." lightbox="./media/deploy-2.png":::

1. Choose the targets in **Publish Completed** state and click on **Deploy Solution**.

    :::image type="content" source="./media/deploy-3.png" alt-text="Screenshot of the Deploy tab showing how to deploy a solution." lightbox="./media/deploy-3.png":::

1. In the confirmation window, click on **Confirm** to proceed.

    :::image type="content" source="./media/deploy-4.png" alt-text="Screenshot of the Deploy tab showing how to confirm the deployment of a solution." lightbox="./media/deploy-4.png":::

1. You can see a notification of deployment in progress at the top right corner of the page. 

    :::image type="content" source="./media/deploy-5.png" alt-text="Screenshot of the Deploy tab showing the notification of a deployment in progress." lightbox="./media/deploy-5.png":::

1. To view the detailed status of your deployment, click on the notification icon at the top right and click on **Show in event logs**.

    :::image type="content" source="./media/deploy-6.png" alt-text="Screenshot of the Deploy tab showing the deployment status." lightbox="./media/deploy-6.png":::

1. Click on the respective **Event name** to open the **Status details** pane showing all the intermediate steps of the operation, along with date and time of completion and the user who initiated it. Details of shared app dependencies associated with the current deployment, if any, also show up on this side-pane.

    :::image type="content" source="./media/deploy-7.png" alt-text="Screenshot of the Deploy tab showing the deployment status details." lightbox="./media/deploy-7.png":::

### Other things you can do in the Deploy tab

In case something goes wrong during deployment, you can:

|Link|Action	|
|---------------------|------------------|
|[Roll back a solution](deploy.md#roll-back-a-solution)|Roll back the solution to a previous version if a deployment fails.|
|[Retry a deployment](deploy.md#retry-a-failed-deployment)|Retry the deployment in case of a failure.|
|[Stop a deployed solution](deploy.md#stop-a-deployed-solution)|Stop a deployed solution but don't want to delete it.|
|[Delete a solution](deploy.md#delete-a-solution)|Delete solutions that aren't yet deployed.|


## Monitor your solutions

In the [Monitor tab](monitor.md), you can:

- View deployment status across all solutions and targets.
- Investigate failures and view configuration details.
- Filter and group data for easier analysis.

### User scenario: Checking the failure of a solution deployment

Goal: Identify why a solution deployment failed.

Steps:

1. Go to the [**Monitor tab**](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview).
1. Select the **Solutions** subtab.

    :::image type="content" source="./media/monitor-solutions.png" alt-text="Screenshot of the Solutions tab in workload orchestration portal showing the default view." lightbox="./media/monitor-solutions.png":::

1. Search for the solution you're interested in.
    - Enter the keyword in the **search box** on the top right of the list next to "Group by:", or
    - Select the **Deployment failure** status tile at the top of the page to filter the list of solutions by that status.
    
    :::image type="content" source="./media/user-guide-monitor.png" alt-text="Screenshot of the Monitor tab showing how to filter by status or keyword." lightbox="./media/user-guide-monitor.png":::

1. Identify the solution you're looking for, it should display "Deployment failure" status.
1. Click on the **status** to view the details of the failure. 

    :::image type="content" source="./media/monitor-failure.png" alt-text="Screenshot of the Monitor tab showing how to click on a failed deployment." lightbox="./media/monitor-failure.png":::

1. In the new pane, review the failure reason, fired time, and affected target.

    :::image type="content" source="./media/monitor-failure-2.png" alt-text="Screenshot of the Monitor tab showing how to view the details of a failed deployment." lightbox="./media/monitor-failure-2.png":::

### Other things you can do in the Monitor tab

In the Monitor tab, you can also:

|Link|Action	|
|---------------------|------------------|
|[View the capabilities of a solution](monitor.md#view-the-capabilities-of-a-solution)|View the features and functionalities of a solution.|
|[View the details of a target](monitor.md#view-the-details-of-a-target)|View the detailed revision of a target if it has a solution available.|