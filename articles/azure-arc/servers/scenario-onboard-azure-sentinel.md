---
title: Onboard Azure Arc-enabled server to Microsoft Sentinel
description: Learn how to add your Azure Arc-enabled servers to Microsoft Sentinel and proactively monitor their security status.
ms.date: 12/04/2024
ms.topic: concept-article
---

# Onboard Azure Arc-enabled servers to Microsoft Sentinel

This article helps you onboard your Azure Arc-enabled machines to [Microsoft Sentinel](/azure/sentinel/overview) to start collecting security-related events. Microsoft Sentinel provides a single solution for alert detection, threat visibility, proactive hunting, and threat response across the enterprise.

## Prerequisites

Before you start, make sure you meet the following requirements:

- A [Log Analytics workspace](/azure/azure-monitor/logs/data-platform-logs). For more information about Log Analytics workspaces, see [Designing your Azure Monitor Logs deployment](/azure/azure-monitor/logs/workspace-design)

- Microsoft Sentinel [enabled in your subscription](/azure/sentinel/quickstart-onboard)

- Your machine is connected to Azure Arc-enabled servers

## Onboard Azure Arc-enabled servers to Microsoft Sentinel

Microsoft Sentinel comes with many connectors for Microsoft solutions, available out of the box and providing real-time integration. For physical and virtual machines, you can install the Log Analytics agent that collects the logs and forwards them to Microsoft Sentinel. Azure Arc-enabled servers supports deploying the Log Analytics agent using the following methods:

- Using the VM extensions framework.

    This feature in Azure Arc-enabled servers allows you to deploy the Log Analytics agent VM extension to a non-Azure Windows and/or Linux server. VM extensions can be managed using the following methods on your hybrid machines or servers managed by Azure Arc-enabled servers:

    - The [Azure portal](manage-vm-extensions-portal.md)
    - The [Azure CLI](manage-vm-extensions-cli.md)
    - [Azure PowerShell](manage-vm-extensions-powershell.md)
    - Azure [Resource Manager templates](manage-vm-extensions-template.md)

- Using Azure Policy.

    Using this approach, you use the Azure Policy [Deploy Log Analytics agent to Linux or Azure Arc machines](/azure/governance/policy/samples/built-in-policies#monitoring) built-in policy to audit if the Azure Arc-enabled server has the Log Analytics agent installed. If the agent isn't installed, it automatically deploys it using a remediation task. Alternatively, if you plan to monitor the machines with Azure Monitor for VMs, instead use the [Enable Azure Monitor for VMs](/azure/governance/policy/samples/built-in-initiatives#monitoring) initiative to install and configure the Log Analytics agent.

We recommend installing the Log Analytics agent for Windows or Linux using Azure Policy.

After your Arc-enabled servers are connected, your data starts streaming into Microsoft Sentinel and is ready for you to start working with. You can view the logs in the [built-in workbooks](/azure/sentinel/get-visibility) and start building queries in Log Analytics to [investigate the data](/azure/sentinel/investigate-cases).

## Next steps

Get started [detecting threats with Microsoft Sentinel](/azure/sentinel/detect-threats-built-in).
