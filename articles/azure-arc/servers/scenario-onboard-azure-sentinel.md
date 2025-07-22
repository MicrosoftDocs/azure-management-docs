---
title: Onboard Azure Arc-enabled server to Microsoft Sentinel
description: Learn how to add your Azure Arc-enabled servers to Microsoft Sentinel and proactively monitor their security status.
ms.date: 05/08/2025
ms.topic: how-to
# Customer intent: "As a security administrator, I want to onboard my Azure Arc-enabled servers to a centralized monitoring solution, so that I can collect and analyze security events for proactive threat detection and response."
---

# Onboard Azure Arc-enabled servers to Microsoft Sentinel

This article helps you onboard your Azure Arc-enabled machines to [Microsoft Sentinel](/azure/sentinel/overview) to start collecting security-related events. Microsoft Sentinel provides a single solution for alert detection, threat visibility, proactive hunting, and threat response across the enterprise.

## Prerequisites

Before you start, make sure you meet the following requirements:

- One or more machines onboarded to Azure Arc.
- The [Azure Monitor Agent](/azure/azure-monitor/logs/data-platform-logs) must be installed and enabled on your Arc-enabled machines. For more information, see [Deployment options for Azure Monitor agent on Azure Arc-enabled servers](azure-monitor-agent-deployment.md).
- A [Log Analytics workspace](/azure/azure-monitor/logs/data-platform-logs). For more information about Log Analytics workspaces, see [Design a Log Analytics workspace architecture](/azure/azure-monitor/logs/workspace-design).
- Microsoft Sentinel must be [enabled in your subscription](/azure/sentinel/quickstart-onboard).

## Enable the Azure Monitor Agent on your Arc-enabled servers

Microsoft Sentinel comes with many [data connectors](/azure/sentinel/connect-data-sources) for Microsoft solutions, available out of the box and providing real-time integration. For physical and virtual machines, the Azure Monitor Agent can forward information to Microsoft Sentinel.

You can deploy the Azure Monitor Agent to your Arc-enabled servers by installing the Azure Monitor Agent extension. This can be done individually on each machine, or at scale via Azure Policy or Azure Automation. For more information, see [Deployment options for Azure Monitor Agent on Azure Arc-enabled servers](azure-monitor-agent-deployment.md).

## Enable Microsoft Sentinel and set up a data connector

Once the Azure Monitor Agent is installed, you can enable Microsoft Sentinel and set up a data connector to start collecting security-related events from your Arc-enabled servers. For more information, see [Quickstart: Onboard Microsoft Sentinel](/azure/sentinel/quickstart-onboard?tabs=azure-portal).

After your Arc-enabled servers are connected, your data starts streaming into Microsoft Sentinel and is ready for you to start working with. You can view the logs in the [built-in workbooks](/azure/sentinel/get-visibility) and start building queries in Log Analytics to [investigate the data](/azure/sentinel/investigate-cases).

## Next steps

Get started [detecting threats with Microsoft Sentinel](/azure/sentinel/detect-threats-built-in).
