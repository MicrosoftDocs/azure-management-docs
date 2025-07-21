---
title: Deploy Azure Monitor agent on Arc-enabled servers
description: This article reviews the different methods to deploy the Azure Monitor Agent on Windows and Linux-based machines registered with Azure Arc-enabled servers.
ms.date: 05/01/2025
ms.topic: concept-article
ms.custom: linux-related-content
# Customer intent: "As an IT administrator managing hybrid environments, I want to deploy the Azure Monitor agent on Arc-enabled servers, so that I can monitor performance, ensure security, and maintain compliance across both virtual and physical machines."
---

# Deployment options for Azure Monitor agent on Azure Arc-enabled servers

Azure Monitor supports multiple methods to install the [Azure Monitor agent](/azure/azure-monitor/agents/agents-overview) as an extension on Azure Arc-enabled servers. Azure Arc-enabled servers support the [Azure VM extension framework](manage-vm-extensions.md), which provides post-deployment configuration and automation tasks, enabling you to simplify management of your hybrid machines just as you can with Azure VMs.

The Azure Monitor Agent is required if you want to:

* Monitor the operating system and any workloads running on the machine or server using [VM insights](/azure/azure-monitor/vm/vminsights-overview)
* Analyze and alert using [Azure Monitor](/azure/azure-monitor/overview)
* Perform security monitoring in Azure by using [Microsoft Defender for Cloud](/azure/defender-for-cloud/defender-for-cloud-introduction) or [Microsoft Sentinel](scenario-onboard-azure-sentinel.md).
* Collect inventory and track changes by using [Azure Monitor agent](/azure/automation/change-tracking/enable-vms-monitoring-agent?tabs=singlevm%2Cmultiplevms%2Carcvm&pivots=single-portal).

> [!NOTE]
> Azure Monitor agent logs are stored locally and are updated after temporary disconnection of an Arc-enabled machine.

This article reviews recommended deployment methods for the Azure Monitor agent VM extension, across multiple production physical servers or virtual machines in your environment, to help you determine which works best for your organization.

> [!TIP]
> For more information about the Azure Monitor agent, see [Install and manage the Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-manage).

## Deploy the extension individually on each machine

This method supports managing the installation, management, and removal of VM extensions (including the Azure Monitor agent) from the [Azure portal](manage-vm-extensions-portal.md), using [PowerShell](manage-vm-extensions-powershell.md), the [Azure CLI](manage-vm-extensions-cli.md), or with an [Azure Resource Manager (ARM) template](manage-vm-extensions-template.md). You can enable [automatic extension upgrade](manage-automatic-vm-extension-upgrade.md), which automatically updates the extension to the latest version when available.

Advantages of deploying the extension individually include:

* Can be useful for testing purposes
* Useful if you have a few machines to manage, or for testing with a small set of servers
* Immediate deployment of extension

Disadvantages of deploying the extension individually include:

* Limited automation
* Not scalable to many servers
* Doesn't create a Data Collection Rule (DCR); you must create a DCR separately and associate it with the agent before data collection begins.

## Use Azure Policy

You can use [Azure Policy](/azure/governance/policy) to deploy the Azure Monitor agent VM extension at-scale to machines in your environment, and maintain configuration compliance. For detailed steps, see [Deploy and configure Azure Monitor Agent using Azure Policy](deploy-ama-policy.md).

Advantages of using Azure Policy include:

* Reinstalls the extension if removed (after policy evaluation)
* Identifies and installs the extension when a new Azure Arc-enabled server is registered with Azure

Disadvantages of using Azure Policy include:

* The **Configure** *operating system* **Arc-enabled machines to run Azure Monitor Agent** policy only installs the Azure Monitor agent extension and configures the agent to report to a specified Log Analytics workspace.
* Standard compliance evaluation cycle is once every 24 hours. An evaluation scan for a subscription or a resource group can be started with Azure CLI, Azure PowerShell, a call to the REST API, or by using the Azure Policy Compliance Scan GitHub Action. For more information, see [Evaluation triggers](/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

## Use Azure Automation

The process automation operating environment in [Azure Automation](/azure/automation) and its support for PowerShell and Python runbooks can help automate the deployment of the Azure Monitor agent VM extension at scale to machines in your environment.

Advantages of using Azure Automation include:

* Can use a scripted method to automate deployment and configuration using scripting languages you're familiar with
* Runs on a schedule that you define and control
* Authenticate securely to Arc-enabled servers from the Automation account using a managed identity

Disadvantages of using Azure Automation include:

* Requires an Azure Automation account
* Requires authoring and managing runbooks in Azure Automation
* Must create a runbook based on PowerShell or Python, depending on the target operating system

## Next steps

* Read the VM insights [Monitor performance](/azure/azure-monitor/vm/vminsights-performance) and [Map feature](/azure/azure-monitor/vm/vminsights-maps) articles to see how well your machine is performing and view discovered application components.
* To start collecting security-related events with Microsoft Sentinel, see [onboard to Microsoft Sentinel](scenario-onboard-azure-sentinel.md), or to collect with Microsoft Defender for Cloud, see [onboard to Microsoft Defender for Cloud](/azure/defender-for-cloud/quickstart-onboard-machines?toc=%2Fazure%2Fazure-arc%2Fservers%2Ftoc.json&bc=%2Fazure%2Fazure-arc%2Fservers%2Fbreadcrumb%2Ftoc.json).


