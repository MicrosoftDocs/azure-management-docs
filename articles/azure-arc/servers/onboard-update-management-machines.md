---
title: Connect machines from Azure Automation Update Management
description: In this article, you learn how to connect hybrid machines to Azure Arc managed by Automation Update Management.
ms.date: 11/06/2023
ms.topic: concept-article
# Customer intent: "As a systems administrator, I want to connect hybrid machines to Azure Arc using Automation Update Management, so that I can centrally manage and monitor my on-premises and cloud resources effectively."
---

# Connect hybrid machines to Azure from Automation Update Management

You can enable Azure Arc-enabled servers for one or more of your Windows or Linux virtual machines or physical servers hosted on-premises or other cloud environment that are managed with Azure Automation Update Management. This onboarding process automates the download and installation of the [Connected Machine agent](agent-overview.md). To connect the machines to Azure Arc-enabled servers, a Microsoft Entra [service principal](/azure/active-directory/develop/app-objects-and-service-principals) is used instead of your privileged identity to [interactively connect](onboard-portal.md) the machine. This service principal is created automatically as part of the onboarding process for these machines.

Before you get started, be sure to review the [prerequisites](prerequisites.md) and verify that your subscription and resources meet the requirements. For information about supported regions and other related considerations, see [supported Azure regions](overview.md#supported-regions).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## How it works

When the onboarding process is launched, an Active Directory [service principal](/azure/active-directory/fundamentals/service-accounts-principal) is created in the tenant.

To install and configure the Connected Machine agent on the target machine, a master runbook named **Add-UMMachinesToArc** runs in the Azure sandbox. Based on the operating system detected on the machine, the master runbook calls a child runbook named **Add-UMMachinesToArcWindowsChild** or **Add-UMMachinesToArcLinuxChild** that runs under the system [Hybrid Runbook Worker](/azure/automation/automation-hybrid-runbook-worker) role directly on the machine. Runbook job output is written to the job history, and you can view their [status summary](/azure/automation/automation-runbook-execution#job-statuses) or drill into details of a specific runbook job in the [Azure portal](/azure/automation/manage-runbooks#view-statuses-in-the-azure-portal) or using [Azure PowerShell](/azure/automation/manage-runbooks#retrieve-job-statuses-using-powershell). Execution of runbooks in Azure Automation writes details in an activity log for the Automation account. For details of using the log, see [Retrieve details from Activity log](/azure/automation/manage-runbooks#retrieve-details-from-activity-log).

The final step establishes the connection to Azure Arc using the `azcmagent` command using the service principal to register the machine as a resource in Azure.

## Prerequisites

This method requires that you are a member of the [Automation Job Operator](/azure/automation/automation-role-based-access-control#automation-job-operator) role or higher so you can create runbook jobs in the Automation account.

If you have enabled Azure Policy to [manage runbook execution](/azure/automation/enforce-job-execution-hybrid-worker) and enforce targeting of runbook execution against a Hybrid Runbook Worker group, this policy must be disabled. Otherwise, the runbook jobs that onboard the machine(s) to Arc-enabled servers will fail.

## Add machines from the Azure portal

Perform the following steps to configure the hybrid machine with Arc-enabled servers. The server or machine must be powered on and online in order for the process to complete successfully.

1. From your browser, go to the [Azure portal](https://portal.azure.com).

1. Navigate to the **Machines - Azure Arc** page, select **Add/Create**, and then select **Add a machine** from the drop-down menu.

1. On the **Add servers with Azure Arc** page, select **Add servers** from the **Add managed servers from Update Management** tile.

1. On the **Resource details** page, configure the following:

    1. Select the **Subscription** and **Resource group** where you want the server to be managed within Azure.
    1. In the **Region** drop-down list, select the Azure region to store the servers metadata.
    1. For **Connectivity method**:
        1. Choose either **Public endpoint** or **Private endpoint**. If you select **Private endpoint**, you can either select an existing private link scope or create a new one.
        1. If you want to use a **Proxy server URL**, enter the proxy server IP address or the name and port number that the machine will use in the format `http://<proxyURL>:<proxyport>`.
        1. If you selected **Public endpoint** and you want to use [Azure Arc Gateway](arc-gateway.md), select an existing **Gateway resource** or create a new one.
    1. Select **Next**.

1. On the **Servers** page, select **Add Servers**, then select the **Subscription** and **Automation account** from the drop-down list that has the Update Management feature enabled and includes the machines you want to onboard to Azure Arc-enabled servers.

   After specifying the Automation account, the list below returns non-Azure machines managed by Update Management for that Automation account. Both Windows and Linux machines are listed and for each one, select **add**.

   You can review your selection by selecting **Review selection**. If you want to remove a machine, select **remove** from under the **Action** column.

   Once you confirm your selection, select **Next**.

1. On the **Tags** page, specify one or more **Name**/**Value** pairs to support your standards. Select **Next: Review + add**.

1. Review the summary information, and then select **Add machines**. If you still need to make changes, select **Previous**.

## Verify the connection with Azure Arc

After the agent is installed and configured to connect to Azure Arc-enabled servers, go to the [Azure portal](https://aka.ms/hybridmachineportal) to verify that the server has successfully connected.

:::image type="content" source="./media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="Screenshot showing a successful machine connection in the Azure portal." border="false":::

## Next steps

- Troubleshooting information can be found in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md).

- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.

- Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview), for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verify the machine is reporting to the expected Log Analytics workspace, enable monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.
