---
title: User Guide for the Workload Orchestration Portal
description: Learn how to navigate the workload orchestration portal to monitor, configure, and deploy solutions without writing code.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/12/2025
---

# User guide for the workload orchestration portal

This guide is designed for non-developer users who want to manage solutions in the workload orchestration portal without writing code. It walks you through three key areas:

- Monitor: Track the health and status of your deployments.
- Configure: Set up your solutions and environments.
- Deploy: Launch your solutions to the right targets.

## Monitor Your Solutions
What You Can Do
- View deployment status across all solutions and targets.
- Investigate failures and view configuration details.
- Filter and group data for easier analysis.

### User Scenario: Checking Deployment Failures
Goal: Investigate why a deployment failed for “ap-pso1-solution2”.
Steps:
Go to the Monitor tab.
Select the Solutions sub-tab.
Click on the solution with status “Deployment failure”.
Review the failure reason, affected targets, and timestamps.

### Explore Capabilities
Click on capability tags to understand what each solution supports.
🔗 For more on viewing capabilities, see #view-the-capabilities-of-a-solution.


## Configure Your Solutions
What You Can Do
- Set up configuration values for factories and lines.
- Link solutions to targets.
- Publish configurations for deployment.

### User Scenario: Setting Up a New Factory
Goal: Configure a new factory called “Contoso North” with the Hotmelt solution.
Steps:
1. Go to the Configure tab.
1. Select the Factory sub-tab.
1. Click on “Contoso North” and then Configure.
1. Enter required values (e.g., temperature range, endpoints).
1. Link the Hotmelt solution and define shared parameters.
1. Click Configure Factory to save.

For configuring lines instead of factories, see #configure-line-parameters.

### Advanced: Configure with Dependencies
If your solution depends on another (e.g., FSAD depends on SSA), the portal guides you through linking them during configuration.
🔗 See #configure-solution-parameters-for-a-solution-with-dependencies.

## Deploy Your Solutions
What You Can Do
- Deploy solutions to specific targets.
- Roll back, retry, stop, or delete deployments.

### User Scenario: Deploying to Chicago Factory
Goal: Deploy the Hotmelt solution to the Chicago Factory.
Steps:
Go to the Deploy tab.
Select “Chicago Factory” from the target list.
Choose the Hotmelt solution in “Ready to deploy” state.
Click Deploy Solution and confirm.
### If Something Goes Wrong
- Rollback: Choose a previous version and click Rollback.
- Retry: Click Retry Deployment on the failed version.
- Stop: Use Stop to pause a deployed solution.
- Delete: Remove solutions that are not yet deployed.
🔗 For rollback instructions, see #roll-back-a-solution.



## Quick Links to Key Actions

|Action	|Where to go |Link|
|Configure a factory|	Configure tab → Factory	|#configure-your-solutions|
|Deploy a solution	|Deploy tab	|#deploy-your-solutions|
|Monitor failures	|Monitor tab → Solutions|	#monitor-your-solutions|
|Roll back a deployment|	Deploy tab	|#roll-back-a-solution|
|View previous revisions|	Configure tab → Solutions	|#show-previous-versions-of-a-solution|