---
title: Tutorial - Monitor a hybrid machine with Azure Monitor VM insights
description: Learn how to collect and analyze data from a hybrid machine in Azure Monitor.
ms.topic: tutorial
ms.date: 04/24/2025
---

# Tutorial: Monitor a hybrid machine with VM insights

[Azure Monitor](/azure/azure-monitor/overview) can collect data directly from your hybrid machines into a Log Analytics workspace for detailed analysis and correlation. Typically, this would require installing the [Log Analytics agent](/azure/azure-monitor/agents/log-analytics-agent) on the machine using a script, manually, or an automated method following your configuration management standards. Now, Azure Arc-enabled servers can install the Log Analytics and Dependency agent [VM extension](../manage-vm-extensions.md) for Windows and Linux, enabling [VM insights](/azure/azure-monitor/vm/vminsights-overview) to collect data from your non-Azure VMs.

<!---This tutorial shows you how to configure and collect data from your Linux or Windows machines by enabling VM insights following a simplified set of steps, which streamlines the experience and takes a shorter amount of time.--->

In this tutorial, you'll learn how to:

> [!div class="checklist"]
> * Enable and configure VM insights for your Linux or Windows non-azure VMs
> * Collect and view data from these VMs

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

* Use our quickstart to [connect a hybrid machine](quick-enable-hybrid-vm.md) to Azure Arc. This tutorial assumes that you have already connected a machine to Azure Arc.

* VM extension functionality is available only in the list of [supported regions](../overview.md#supported-regions).

* See [Supported operating systems](/azure/azure-monitor/vm/vminsights-enable-overview#supported-operating-systems) to ensure that the servers operating system you're enabling is supported by VM insights.

* Review firewall requirements for the Log Analytics agent provided in the [Log Analytics agent overview](/azure/azure-monitor/agents/log-analytics-agent#network-requirements). The VM insights Map Dependency agent doesn't transmit any data itself, and it doesn't require any changes to firewalls or ports.

## Enable VM insights

1. Launch the Azure Arc service in the Azure portal by clicking **All services**, then searching for and selecting **Machines - Azure Arc**.

1. On the **Azure Arc - Machines** page, select the connected machine you created in the [quickstart](quick-enable-hybrid-vm.md) article.

1. In the service menu, under **Monitoring**, select **Insights** and then **Enable**.

    :::image type="content" source="./media/tutorial-enable-vm-insights/insights-option.png" lightbox="./media/tutorial-enable-vm-insights/insights-option.png" alt-text="Screenshot of the Insights pane with Enable button." border="false":::

1. In the **Monitoring configuration** pane, confirm that the right subscription appears. For **Data collection rule**, select **Create new**.

1. In the **Create new rule** pane, enter a name for your data collection rule. For this tutorial, leave the other options as is. Don't select an existing Log Analytics workspace if you already have one. Instead, select the default, which is a workspace with a unique name in the same region as your registered connected machine. This workspace is created and configured for you.

    Status messages display while the configuration is performed and extensions are installed on your connected machine. This process takes a few minutes.

    When the process is complete, a message displays that the machine has been onboarded and that Insights has been successfully deployed.

## View data collected

1. After deployment and configuration are complete, select **Insights**, and then select the **Performance** tab. The Performance tab shows a select group of performance counters collected from the guest operating system of your machine. Scroll down to view more counters, and move the mouse over a graph to view average and percentiles taken starting from the time when the Log Analytics VM extension was installed on the machine.

    :::image type="content" source="./media/tutorial-enable-vm-insights/insights-performance-charts.png" lightbox="./media/tutorial-enable-vm-insights/insights-performance-charts.png"alt-text="Screenshot of Insights Performance tab with charts for selected machine." border="false":::

1. Select **Map**. The maps feature shows the processes running on the machine and their dependencies. Select **Properties** to open the property pane (if it isn't already open).

    :::image type="content" source="./media/tutorial-enable-vm-insights/insights-map.png" lightbox="./media/tutorial-enable-vm-insights/insights-map.png" alt-text="Screenshot of Insights Map tab with map for selected machine." border="false":::

1. Expand the processes for your machine. Select one of the processes to view its details and to highlight its dependencies.

1. Select your machine again and then select **Log Events**. You see a list of tables that are stored in the Log Analytics workspace for the machine. This list differs between Windows or Linux machines.

1. Select the **Event** table. The **Event** table includes all events from the Windows event log. Log Analytics opens with a simple query to retrieve collected event log entries.

## Next steps

To learn more about Azure Monitor, see the following article:

> [!div class="nextstepaction"]
> [Azure Monitor overview](/azure/azure-monitor/overview)
