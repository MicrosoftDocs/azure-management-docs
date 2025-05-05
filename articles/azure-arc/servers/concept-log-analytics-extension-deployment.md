---
title: Deploy Azure Monitor agent on Arc-enabled servers
description: This article reviews the different methods to deploy the Azure Monitor agent on Windows and Linux-based machines registered with Azure Arc-enabled servers in your local datacenter or other cloud environment.
ms.date: 05/08/2024
ms.topic: concept-article
ms.custom: linux-related-content
---

# Deployment options for Azure Monitor agent on Azure Arc-enabled servers

Azure Monitor supports multiple methods to install the Azure Monitor agent and connect your machine or server registered with Azure Arc-enabled servers to the service. Azure Arc-enabled servers support the Azure VM extension framework, which provides post-deployment configuration and automation tasks, enabling you to simplify management of your hybrid machines like you can with Azure VMs.

The Azure Monitor agent is required if you want to:

* Monitor the operating system and any workloads running on the machine or server using [VM insights](/azure/azure-monitor/vm/vminsights-overview)
* Analyze and alert using [Azure Monitor](/azure/azure-monitor/overview)
* Perform security monitoring in Azure by using [Microsoft Defender for Cloud](/azure/defender-for-cloud/defender-for-cloud-introduction) or [Microsoft Sentinel](/azure/sentinel/overview)
* Collect inventory and track changes by using [Azure Monitor agent](/azure/automation/change-tracking/enable-vms-monitoring-agent?tabs=singlevm%2Cmultiplevms%2Carcvm&pivots=single-portal).

> [!NOTE]
> Azure Monitor agent logs are stored locally and are updated after temporary disconnection of an Arc-enabled machine.
> 

This article reviews the deployment methods for the Azure Monitor agent VM extension, across multiple production physical servers or virtual machines in your environment, to help you determine which works best for your organization. If you are interested in the new Azure Monitor agent and want to see a detailed comparison, see [Azure Monitor agents overview](/azure/azure-monitor/agents/agents-overview).  

## Installation options

Review the different methods to install the VM extension using one method or a combination and determine which one works best for your scenario.

### Use Azure Arc-enabled servers

This method supports managing the installation, management, and removal of VM extensions (including the Azure Monitor agent) from the [Azure portal](manage-vm-extensions-portal.md), using [PowerShell](manage-vm-extensions-powershell.md), the [Azure CLI](manage-vm-extensions-cli.md), or with an [Azure Resource Manager (ARM) template](manage-vm-extensions-template.md).

#### Advantages

* Can be useful for testing purposes
* Useful if you have a few machines to manage

#### Disadvantages

* Limited automation when using an Azure Resource Manager template
* Can only focus on a single Arc-enabled server, and not multiple instances
* Only supports specifying a single workspace to report to; requires using PowerShell or the Azure CLI to configure the Log Analytics Windows agent VM extension to report to up to four workspaces
* Doesn't support deploying the Dependency agent from the portal; you can only use PowerShell, the Azure CLI, or ARM template

### Use Azure Policy

You can use Azure Policy to deploy the Azure Monitor agent VM extension at-scale to machines in your environment, and maintain configuration compliance. This is accomplished by using either the [**Configure Linux Arc-enabled machines to run Azure Monitor Agent**](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F845857af-0333-4c5d-bbbc-6076697da122) or the [**Configure Windows Arc-enabled machines to run Azure Monitor Agent**](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F94f686d6-9a24-4e19-91f1-de937dc171a4) policy definition.

Azure Policy includes several prebuilt definitions related to Azure Monitor. For a complete list of the built-in policies in the  **Monitoring** category, see [Azure Policy built-in definitions for Azure Monitor](/azure/azure-monitor/policy-reference).

#### Advantages

* Reinstalls the VM extension if removed (after policy evaluation)
* Identifies and installs the VM extension when a new Azure Arc-enabled server is registered with Azure

#### Disadvantages

* The **Configure** *operating system* **Arc-enabled machines to run Azure Monitor Agent** policy only installs the Azure Monitor agent extension and configures the agent to report to a specified Log Analytics workspace.
* Standard compliance evaluation cycle is once every 24 hours. An evaluation scan for a subscription or a resource group can be started with Azure CLI, Azure PowerShell, a call to the REST API, or by using the Azure Policy Compliance Scan GitHub Action. For more information, see [Evaluation triggers](/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

### Use Azure Automation

The process automation operating environment in Azure Automation and its support for PowerShell and Python runbooks can help you automate the deployment of the Azure Monitor agent VM extension at scale to machines in your environment.

#### Advantages

* Can use a scripted method to automate its deployment and configuration using scripting languages you're familiar with
* Runs on a schedule that you define and control
* Authenticate securely to Arc-enabled servers from the Automation account using a managed identity

#### Disadvantages

* Requires an Azure Automation account
* Experience authoring and managing runbooks in Azure Automation
* Must create a runbook based on PowerShell or Python, depending on the target operating system

### Use Azure portal

The Azure Monitor agent VM extension can be installed using the Azure portal. See [Automatic extension upgrade for Azure Arc-enabled servers](manage-automatic-vm-extension-upgrade.md) for more information about installing extensions from the Azure portal.

#### Advantages

* Point and click directly from Azure portal
* Useful for testing with small set of servers
* Immediate deployment of extension

#### Disadvantages

* Not scalable to many servers
* Limited automation

## Next steps

* To start collecting security-related events with Microsoft Sentinel, see [onboard to Microsoft Sentinel](scenario-onboard-azure-sentinel.md), or to collect with Microsoft Defender for Cloud, see [onboard to Microsoft Defender for Cloud](/azure/security-center/quickstart-onboard-machines).

* Read the VM insights [Monitor performance](/azure/azure-monitor/vm/vminsights-performance) and [Map dependencies](/azure/azure-monitor/vm/vminsights-maps) articles to see how well your machine is performing and view discovered application components.
