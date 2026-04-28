---
title: Monitor a hybrid machine with Azure Monitor VM insights
description: Learn how to collect and analyze data from a hybrid machine in Azure Monitor.
ms.topic: tutorial
ms.date: 04/28/2026
# Customer intent: As a system administrator, I want to enable VM insights on my hybrid machines, so that I can collect and analyze performance data for better monitoring and management of my infrastructure.
---

# Tutorial: Monitor a hybrid machine with VM insights

[Azure Monitor](/azure/azure-monitor/overview) can collect data directly from your hybrid machines into a Log Analytics workspace for detailed analysis and correlation. You can enable [VM insights](/azure/azure-monitor/vm/vminsights-overview) to collect data from your non-Azure virtual machines (VM). While there are [different options for deploying the Azure Monitor Agent](azure-monitor-agent-deployment.md) to your Arc-enabled servers, this tutorial shows how to do so by using the Azure portal to enable VM insights on a connected machine.

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

* Use our quickstart to [connect a hybrid machine](quick-enable-hybrid-vm.md) to Azure Arc. This tutorial assumes that you already connected a machine to Azure Arc.

* Verify that your hybrid machine uses an [operating system supported by the Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-supported-operating-systems).

* Have permissions to create data collection rules (DCRs) and associate them with machines. See [Data collection rule permissions](/azure/azure-monitor/data-collection/data-collection-rule-create-edit#permissions).

* Configure your network security group (NSG) or firewall to allow outbound TCP port 443 (HTTPS) traffic to the `AzureMonitor` and `AzureResourceManager` **service tag** endpoints. These service tags are for your NSG and *not* Azure resource tags.

  For full requirements, see [Azure Monitor Agent network configuration](/azure/azure-monitor/agents/azure-monitor-agent-network-configuration). You can run one of the following commands to configure traffic for your firewall:

  # [Azure PowerShell](#tab/azure-powershell)

  ```azurepowershell
  $nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "<resource-group>" -Name "<nsg-name>"

  # Add AzureMonitor outbound rule
  $nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "AllowAzureMonitorOutbound" `
  -Priority 150 `
  -Direction Outbound `
  -Access Allow `
  -Protocol Tcp `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix AzureMonitor `
  -DestinationPortRange 443

  # Add AzureResourceManager outbound rule
  $nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "AllowAzureResourceManagerOutbound" `
  -Priority 151 `
  -Direction Outbound `
  -Access Allow `
  -Protocol Tcp `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix AzureResourceManager `
  -DestinationPortRange 443

  $nsg | Set-AzNetworkSecurityGroup
  ```

  # [Azure CLI](#tab/azure-cli)

  ```azurecli-interactive
  # Add AzureMonitor outbound rule

  az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name AllowAzureMonitorOutbound \
  --priority 150 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 443 \
  --destination-address-prefixes AzureMonitor

  # Add AzureResourceManager outbound rule

  az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name AllowAzureResourceManagerOutbound \
  --priority 151 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 443 \
  --destination-address-prefixes AzureResourceManager
  ```

  ---

## Enable VM insights

1. Sign in to the [Azure portal](https://portal.azure.com). In the search bar, search for and select **Azure Arc**.

1. Under **Infrastructure**, select **Machines**, then select the connected machine you created in the [quickstart](quick-enable-hybrid-vm.md) article.

1. In the service menu, under **Monitoring**, select **Insights**. The portal opens the **Monitor** page for your machine. If enhanced monitoring wasn't enabled, several performance charts show no data and a message appears offering to enable it. Select **Configure** to open the **Configure monitor** page.

   :::image type="content" source="./media/tutorial-enable-vm-insights/insights-option.png" lightbox="./media/tutorial-enable-vm-insights/insights-option.png" alt-text="Screenshot of the Insights pane with the Configure button." border="false":::

1. On the **Configure monitor** page, leave **[Preview] OpenTelemetry metrics** selected. A default Log Analytics workspace is automatically selected. If one doesn't already exist, one is created in the same region as your connected machine. To use an existing workspace instead, select **Customize infrastructure monitoring** and select the workspace.

1. Select **Review + Enable**, and then select **Enable**.

   Status messages display while the configuration is performed and the Azure Monitor Agent extension is installed on your connected machine. This process takes a few minutes.

   When the process is complete, a message displays that the machine is onboarded and that Insights is successfully deployed.

## View data collected

After the Azure Monitor Agent is installed, it takes a few minutes for enough data to be collected to populate the portal.

1. In the service menu, under **Monitoring**, select **Insights** to return to the **Monitor** page.

1. If you enabled both the metrics-based and logs-based experiences, a selector appears at the top of the page. Select each experience to compare the available charts and insights:

   - **Metrics based visualizations (Preview)** - Shows performance indicators focused on CPU, memory, disk, and network utilization. Also incorporates status from Service Health and Resource Health.

   - **Log based visualizations (Classic)** - Uses summarized performance data collected in your Log Analytics workspace. Select the **Performance** tab to view counters for your machine. Scroll down to view more counters, or hover over a chart and view averages, and percentiles from the time the Azure Monitor Agent was installed.

     :::image type="content" source="./media/tutorial-enable-vm-insights/insights-performance-charts.png" lightbox="./media/tutorial-enable-vm-insights/insights-performance-charts.png" alt-text="Screenshot of Insights Performance tab with charts for selected machine in log-based classic view." border="false":::

1. Select the **Map** tab. The maps feature shows the processes running on the machine and their dependencies. The log-based classic view includes an extra tab. Select **Expand property panel** to open the property pane.

   > [!NOTE]
   > Even though the **Map** tab is visible in both experiences, see [VM Insights Map and Dependency Agent retirement guidance](/azure/azure-monitor/vm/vminsights-maps-retirement) to learn more.

1. To view log data, in the service menu, under **Monitoring**, select **Logs**. A list of tables stored in the Log Analytics workspace for the machine appears. The tables available differ between Windows and Linux machines. By default, VM insights populate the **Heartbeat** and **InsightsMetrics** tables.

1. (Optional) If you're using a Windows machine and configured Windows event log collection, select the **Event** table. Log Analytics opens with a simple query to retrieve collected event log entries. If this table isn't present, event log collection isn't configured. To add it, see [Collect Windows event logs with Azure Monitor Agent](/azure/azure-monitor/agents/data-collection-windows-events).

## Next steps

To learn more about Azure Monitor, see the following article:

[Azure Monitor overview](/azure/azure-monitor/overview)
