---
title: Configure your Solutions with Workload Orchestration Portal
description: Learn how to use the workload orchestration portal to configure your solutions and publish values for deployment.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 05/01/2025
ms.custom:
  - build-2025
# Customer intent: "As a solution administrator, I want to configure and publish solution parameters via the workload orchestration portal, so that I can effectively manage deployment settings across multiple targets."
---

# Configure your solutions with the workload orchestration portal

The Configure tab in the workload orchestration portal provides a comprehensive view of your solutions at the factory and line hierarchy levels. 

This article describes how to use the workload orchestration portal to add configuration values and publish values for deployment for your solutions. If you want information about other tabs in the workload orchestration portal, see [Monitor your solutions](monitor.md) and [Deploy your solutions](deploy.md).

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- All users with access to configure platform or solution level parameters can access the Configure tab. If you don't see a tab or a feature, it might be due to insufficient permissions. Contact your IT administrator to ensure you have the necessary access.

## Navigate the Configure tab

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview).
1. Once you sign in, you see the **Configure** tab on the left side of the page.
1. The Configure tab provides more sub-tabs: **Factory**, **Line**, **Solutions**, and **Published Solutions**. The Factory tab shows the status of all solutions deployed in your environment at the factory level, while the Line tab shows the status of all solutions deployed in your environment at the line level. The Solutions tab shows the status of all solutions deployed in your environment, while the Published Solutions tab shows the status of all published solutions.

    > [!IMPORTANT]
    > Hierarchical levels are custom-defined by the IT admin. You can have up to four hierarchical levels and name them per your requirements, for example, country, region, factory, and line levels. In that case, you see the name of the custom-defined levels in the Configure tab, together with the Solutions and Published Solutions tabs.

1. In any of the sub-tabs, you can filter the list by keyword. To do this, enter the keyword in the **search box** on the top right of the list next to "Group by:". The list of solutions will be filtered to show only those that match the keyword.

    :::image type="content" source="./media/configure-keyword.png" alt-text="Screenshot of the Configure tab showing how to search by keyword." lightbox="./media/configure-keyword.png":::

    You can also apply filters to the list of solutions. To do this, click on **Filter** on the top right of the list and select the **Column** you want to filter by. Then select an **Operator** and enter the **Value** you want to filter by. The list of solutions will be filtered to show only those that match the selected column and value.

    :::image type="content" source="./media/configure-filter-by.png" alt-text="Screenshot of the Configure tab showing how to apply filters." lightbox="./media/configure-filter-by.png":::


## Configure factory parameters

The **Factory** sub-tab shows the list of factories, including their configuration status, linked solutions, and parent site.

:::image type="content" source="./media/configure-factory.png" alt-text="Screenshot of the Factory tab in workload orchestration portal showing the default view." lightbox="./media/configure-factory.png":::

To configure a factory, follow these steps:

1. Select the name of the factory you want to configure and click on **Configure**. 

    :::image type="content" source="./media/configure-factory-1.png" alt-text="Screenshot of the Factory tab in workload orchestration portal showing how to select a factory." lightbox="./media/configure-factory-1.png":::

1. The new details pane shows the configuration values for the selected factory.
1. In the **Configure factory** step, enter the value to configure the factory and click on **Next**.

    :::image type="content" source="./media/configure-factory-2.png" alt-text="Screenshot of the Factory tab in workload orchestration portal showing how to configure a factory in the step 1." lightbox="./media/configure-factory-2.png":::

1. In the **Linked solutions** step, select the solutions and define the common solution parameters. Click on **Next**.

    :::image type="content" source="./media/configure-factory-3.png" alt-text="Screenshot of the Factory tab in workload orchestration portal showing how to add the configuration of the solutions for a factory." lightbox="./media/configure-factory-3.png":::

1. Review the details and click on **Configure factory** to apply the changes.

    :::image type="content" source="./media/configure-factory-4.png" alt-text="Screenshot of the Factory tab in workload orchestration portal showing how to review and apply the changes of a configuration." lightbox="./media/configure-factory-4.png":::


## Configure line parameters

The **Line** sub-tab shows the list of factories, including their configuration status, linked solutions, and parent site.

:::image type="content" source="./media/configure-line.png" alt-text="Screenshot of the Line tab in workload orchestration portal showing the default view." lightbox="./media/configure-line.png":::

To configure a line, follow these steps:

1. Select the name of the line you want to configure and click on **Configure Target**. 

    :::image type="content" source="./media/configure-line-1.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to select a line." lightbox="./media/configure-line-1.png":::

    > [!NOTE]
    > You can update the configuration of a line if its status is not *Configuration up to date*. 

1. The new details pane shows the configuration values for the selected line.
1. In the **Configure Target** step, enter the parameters to ensure the deployment of each target and click on **Next**.

    :::image type="content" source="./media/configure-line-2.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to enter the parameters to configure a target." lightbox="./media/configure-line-2.png":::

1. Review the details and click on **Configure Target** to apply the changes.

    :::image type="content" source="./media/configure-line-3.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to review and apply the changes of the configuration." lightbox="./media/configure-line-3.png":::


## Configure solution parameters

The **Solutions** sub-tab shows the list of solutions, including their version, status, capabilities, and other details.

:::image type="content" source="./media/configure-solutions.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing the default view." lightbox="./media/configure-solutions.png":::

To configure a solution, follow these steps:

1. Select the name of a solution with configuration status *Configuration pending* and click on **Configure and publish**. 

    :::image type="content" source="./media/configure-solution-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to select a solution to configure it." lightbox="./media/configure-solution-1.png":::

1. The new details pane shows the configuration values for the selected solution.
1. In the **Select targets to publish solution** step, auto-publish option is enabled by default which means the values will be applied for all targeted lines. You can **disable auto-publish** and choose certain lines from the dropdown. Click on **Next**.

    :::image type="content" source="./media/configure-solution-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure a solution and disable auto-publish." lightbox="./media/configure-solution-2.png":::

1. In the **Configure target** step, enter the instance name and the parameters to publish the solution and click on **Next**.

    :::image type="content" source="./media/configure-solution-3.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the parameters to configure the targets." lightbox="./media/configure-solution-3.png":::

    > [!TIP]
    > In the authoring process, the default value of a parameter is displayed below the field.

1. The **Review** step lists the final details of the configuration values for the selected targets. You can view the target name, configuration status, which shows if the status is resolved or not, and the publish status, which shows if the solution is published or not.

    :::image type="content" source="./media/configure-solution-4.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review the targets." lightbox="./media/configure-solution-4.png":::

1. Click on the **name** of the target to open the pane showing the list of the resolved configuration values.

    :::image type="content" source="./media/configure-solution-5.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to click on the targets." lightbox="./media/configure-solution-5.png":::

1. Review the resolved configuration values.

    :::image type="content" source="./media/configure-solution-6.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review the resolved configuration values." lightbox="./media/configure-solution-6.png":::

1. You can click on the **download symbol** next to the status to download the final configurations.

    :::image type="content" source="./media/configure-solution-download.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to download the final configurations." lightbox="./media/configure-solution-download.png":::

1. Finally, click on **Publish** to create a new revision of configuration values for the selected targets.

    :::image type="content" source="./media/configure-solution-7.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to publish the configuration of a solution target." lightbox="./media/configure-solution-7.png":::

1. Once the configuration is successful, the publish status is updated to **Published**.

    :::image type="content" source="./media/configure-solution-8.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to publish the configuration of the targets." lightbox="./media/configure-solution-8.png":::

### Configure solution parameters for a solution with dependencies

To configure a solution with dependencies, you need to ensure that the dependent solution is also configured. For example, let's consider a Factory Sensor Anomaly Detector (FSAD) solution, which depends on a Smart Sensor Anomaly (SSA) solution. The FSAD solution is deployed on a child target, while the SSA solution is deployed on a parent target. The FSAD solution uses the SSA solution to synchronize data between devices and servers.

The configuration process is similar to the one described in the previous section, but with some additional steps.

1. Select the **FSAD solution** you want to configure and click on **Configure and publish**.
1. The new details pane shows the configuration values for the selected solution.
1. In the **Select targets to publish solution** step, auto-publish option is enabled by default which means the values will be applied for all targeted lines. You can proceed with auto-publish or choose the targets where the FSAD solution needs to be deployed. Click on **Next**.

    :::image type="content" source="./media/configure-fsad-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure the targets for a solution FSAD." lightbox="./media/configure-fsad-1.png":::

1. In the **Configure target** step, enter the instance name for the solution and the values for FSAD configurations.

    :::image type="content" source="./media/configure-fsad-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the values to configure the targets for a solution FSAD." lightbox="./media/configure-fsad-2.png":::

1. In the **Shared dependencies** step, you can see the details of the dependant SSA instance. Under the **Instance** field, either choose existing or create a new SSA instance. 

    :::image type="content" source="./media/configure-fsad-3.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure the shared dependencies for a solution FSAD." lightbox="./media/configure-fsad-3.png":::

    1. To create a new instance of SSA, **enter** the new instance name and the configuration values for SSA. Click on **Next**.
    
        :::image type="content" source="./media/configure-fsad-4.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to create a new SSA instance." lightbox="./media/configure-fsad-4.png":::
    
    1. Review the SSA details and click on **Configure + publish**.
    
        :::image type="content" source="./media/configure-fsad-5.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review a new SSA instance and publish it." lightbox="./media/configure-fsad-5.png":::

1. Once the dependant solution details are filled, click on **Next**.

    :::image type="content" source="./media/configure-fsad-6.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review the details of the shared dependencies for a solution FSAD." lightbox="./media/configure-fsad-6.png":::

1. In the **Review** step, review the FSAD configuration details, status and click on **Publish** to create a new revision of configuration values for the selected targets.

    :::image type="content" source="./media/configure-fsad-7.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review and publish a FSAD solution." lightbox="./media/configure-fsad-7.png":::

### Show previous revisions of a solution during authoring

You can view the previous revisions of a solution while authoring it. This feature allows you to compare the current configuration with previous revisions and make necessary changes.

1. While configuring a solution, in the **Configure target** step, click on **Show previous versions**.

    :::image type="content" source="./media/configure-previous.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to see the previous versions of a solution." lightbox="./media/configure-previous.png":::

1. The new pane shows the latest five revisions for the solution version. You can see all the older versions by applying filters.

    :::image type="content" source="./media/configure-previous-2.png" alt-text="Screenshot showing the five previous versions of a solution." lightbox="./media/configure-previous-2.png":::

### Resolve a solution failure during authoring

During the configuration process, if a solution fails, you can resolve the failure by following these steps:

1. The **Review** step shows if a configuration fails. Click on the status **Resolve failed** to see the details of the failure.

    :::image type="content" source="./media/configure-failed-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to resolve a failure." lightbox="./media/configure-failed-1.png":::

1. The side pane displays the details. Click on **Retry**.

    :::image type="content" source="./media/configure-failed-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to retry a failed configuration." lightbox="./media/configure-failed-2.png":::

### View details of a solution version

1. Click on the **solution name** to open the details pane.

    :::image type="content" source="./media/configure-solution-details-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how open the details pane of a solution." lightbox="./media/configure-solution-details-1.png":::

1. In the new pane, click on **Show all revisions of this version**.

    :::image type="content" source="./media/configure-solution-details-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to show all revisions of a solution." lightbox="./media/configure-solution-details-2.png":::

1. This action opens the **Published Solutions** sub-tab with filters enabled for solution name and version.

    :::image type="content" source="./media/configure-solution-details-3.png" alt-text="Screenshot of workload orchestration portal showing the published solution tab after clicking on show all revisions of a solution." lightbox="./media/configure-solution-details-3.png":::

## View the published solutions 

The **Published Solutions** sub-tab shows the list of solutions which were authored and published to targets. This implies that the solutions are ready to be deployed.

1. Click on the **solution revision** to view the details of the published solution.

    :::image type="content" source="./media/configure-published-1.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing how to view the details of a solution." lightbox="./media/configure-published-1.png":::

1. The view displays the final configurations, its' values and other details. You can click on **Download YAML** to export final configurations of the solution.

    :::image type="content" source="./media/configure-published-2.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing how to download the YAML file of a solution." lightbox="./media/configure-published-2.png":::

1. You can delete a solution revision if this isn't yet deployed. **Select** the solution revision and click on **Delete**.

    :::image type="content" source="./media/configure-delete.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing how to delete a solution before deploy it." lightbox="./media/configure-delete.png":::

1. If staging is enabled for a solution, you can see the **Staging status**. For more information, see [View staged resources](how-to-stage.md#view-staged-resources).

### Compare revisions of a solution

You can compare the current configuration with previous revisions of a solution. This feature allows you to see the differences between the current and previous configurations.

1. Select the revisions to be compared and click on **Compare revisions**.

    :::image type="content" source="./media/configure-compare-1.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing how to compare revisions." lightbox="./media/configure-compare-1.png":::

1. The new pane displays the configurations and its values for multiple revisions. The differences are highlighted. 

    :::image type="content" source="./media/configure-compare-2.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing the comparison of different revisions." lightbox="./media/configure-compare-2.png":::

### Publish a solution to more targets

You can publish a solution to more targets after the solution is published. This feature allows you to add more targets to the existing solution without creating a new revision.

1. Select the solution name and click on **Publish Solution to Targets**.

    :::image type="content" source="./media/configure-publish-more-1.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing how to publish a solution to more targets." lightbox="./media/configure-publish-more-1.png":::

1. Select the targets where you want the same configurations to be published. Then, click on **Configure + publish**.

    :::image type="content" source="./media/configure-publish-more-2.png" alt-text="Screenshot of the published solutions tab in workload orchestration portal showing how to select the targets to publish a solution to more targets." lightbox="./media/configure-publish-more-2.png":::

## Configure a solution with external validation enabled

If you enable external validation for workload orchestration, during the configuration of the target you see if external validation is enabled for a particular solution.

:::image type="content" source="./media/external-validation-configure.png" alt-text="Screenshot of configure tab showing that external validation is mandatory when configuring a target." lightbox="./media/external-validation-configure.png":::

Under the "Published Solutions" tab, you can see that the solutions with *Publish in progress* and *Publish failed* status have an alert.

:::image type="content" source="./media/external-validation-configure-2.png" alt-text="Screenshot of configure tab showing the alerts when a validation fails." lightbox="./media/external-validation-configure-2.png":::

Click on the **alert** to view the details of external validation.

:::image type="content" source="./media/external-validation-configure-3.png" alt-text="Screenshot of configure tab showing the alerts details when a validation fails." lightbox="./media/external-validation-configure-3.png":::

For more information, see [External validation for workload orchestration](external-validation.md).
