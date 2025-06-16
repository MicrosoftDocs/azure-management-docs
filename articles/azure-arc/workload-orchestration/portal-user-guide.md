---
title: User Guide for the Workload Orchestration Portal
description: Learn how to navigate the workload orchestration portal to monitor, configure, and deploy solutions without writing code.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/12/2025
---

# User guide for the workload orchestration portal

This guide is designed for nondeveloper users who want to manage solutions in the workload orchestration portal without writing code. 

|Tab|Actions|
|----|-------|
|[Monitor](monitor.md)|Track the health and status of your solutions.|
|[Configure](configure.md)|Set up the parameters of your solutions.|
|[Deploy](deploy.md)|Launch your solutions to the right targets and roll back to earlier versions if needed.|


## Monitor your solutions

In the [Monitor tab](monitor.md), you can:

- View deployment status across all solutions and targets.
- Investigate failures and view configuration details.
- Filter and group data for easier analysis.

### User scenario: Checking the failure of a solution deployment

Goal: Identify why a solution deployment failed.
Steps:
1. Go to the **Monitor** tab.
1. Select the **Solutions** subtab.

    :::image type="content" source="./media/monitor-solutions.png" alt-text="Screenshot of the Solutions tab in workload orchestration portal showing the default view." lightbox="./media/monitor-solutions.png":::

1. Search for the solution you're interested in.
    - Enter the keyword in the **search box** on the top right of the list next to "Group by:", or
    - Select the **Deployment failure** status tile at the top of the page to filter the list of solutions by that status.
1. Identify the solution with status "Deployment failure".
1. Click on the **status** to view the details of the failure. 

    :::image type="content" source="./media/monitor-failure.png" alt-text="Screenshot of the Monitor tab showing how to click on a failed deployment." lightbox="./media/monitor-failure.png":::

1. In the new pane, review the failure reason, fired time, and affected target.

    :::image type="content" source="./media/monitor-failure-2.png" alt-text="Screenshot of the Monitor tab showing how to view the details of a failed deployment." lightbox="./media/monitor-failure-2.png":::


### Other things you can do in the Monitor tab

In the Monitor tab, you can also:

- [View the capabilities of a solution](monitor.md#view-the-capabilities-of-a-solution)
- [View the details of a solution](monitor.md#view-the-details-of-a-solution)
- [View the the details of a target](monitor.md#view-the-details-of-a-target)

## Configure your solutions

In the [Configure tab](configure.md), you can:

- Set up configuration values for factories and lines.
- Link solutions to targets.
- Publish configurations for deployment.

### User Scenario: Setting up a new factory 

Goal: Configure a new factory called "Contoso North" with the Hotmelt solution.
Steps:
1. Go to the Configure tab.
1. Select the Factory subtab.
1. Click on “Contoso North” and then Configure.
1. Enter required values, for example, temperature range, endpoints, etc.
1. Link the Hotmelt solution and define shared parameters.
1. Click Configure Factory to save.

### Other things you can do in the Configure tab

In the Configure tab, you can also:


## Deploy your solutions

In the [Deploy tab](deploy.md), you can:

- Deploy solutions to specific targets.
- Roll back, retry, stop, or delete deployments.

### User Scenario: Deploying a solution to a target

Goal: Deploy a solution to a target.
Steps:
1. Go to the **Deploy** tab.
1. Select the target from the target list.
1. Choose the solution in “Ready to deploy” state.
1. Click **Deploy Solution** and confirm.

### Other things you can do in the Deploy tab

In case something goes wrong during deployment, you can:

- [Rollback](deploy.md#roll-back-a-solution) the solution to a previous version.
- [Retry](deploy.md#retry-a-failed-deployment) the deployment in case of a failure.
- [Stop](deploy.md#stop-a-deployed-solution) a deployed solution to pause it.
- [Delete](deploy.md#delete-a-solution) solutions that aren't yet deployed. 


## Quick links to key actions

|Action	|Where to go |Link|
|------------------|---------------------|----------------------------|
|Configure a factory|	Configure tab → Factory	|#configure-your-solutions|
|Deploy a solution	|Deploy tab	|#deploy-your-solutions|
|Monitor failures	|Monitor tab → Solutions|	#monitor-your-solutions|
|Rollback a deployment|	Deploy tab	|#roll-back-a-solution|
|View previous revisions|	Configure tab → Solutions	|#show-previous-versions-of-a-solution|