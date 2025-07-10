---
title:  Azure Arc-enabled servers disconnected scenarios
description: 
ms.date: 07/10/2025
ms.topic: overview
# Customer intent: "As a system administrator managing a hybrid cloud environment, I want to understand how Azure Arc-enabled servers works when they are disconnected from the internet, so that I can resolve problems and manage my environment more effectively."
---

# Azure Arc-enabled servers disconnected scenarios

Azure Arc-enabled servers can operate in disconnected scenarios, allowing you to manage your servers even when they are not connected to the internet. This capability is essential for maintaining control over your hybrid cloud environment, especially in scenarios where network connectivity is intermittent or unavailable.

This article describes how Azure Arc-enabled servers behave when they are disconnected from the internet, including the impact on management capabilities and how to stay notified when servers become disconnected.

## Arc-enabled servers when disconnected

The [Connected Machine agent](agent-overview.md) sends a regular heartbeat message to Azure to confirm connectivity. When Azure stops receiving these heartbeat messages from a machine for more than 15 minutes, the machine

It's important to understand the impacts when an Azure Arc-enabled server has no internet connectivity.

- While the Hybrid Instance Metadata Service will keep running, the service won't receive Microsoft Entra tokens or IMDS metadata while a server is disconnected. Any services dependent on these items will be disrupted. After 45 to 90 days of being disconnected, the Microsoft Entra management ID expires, and the Connected Machine Agent can no longer connect to the Arc-enabled server.
- Azure Policy assignments that target disconnected machines continue to run. However, guest assignment is stored locally for 14 days. Within the 14-day period, if the Connected Machine agent reconnects to the service, policy assignments are reapplied. After 14 days, assignments are deleted and aren't reassigned to the machine.
- Existing extensions continue to run, but operations on extensions (such as install, uninstall, or update) aren't possible while the server is disconnected. These operations are queued on the service for up to 6 hours, and can only be executed when connectivity is restored.

When an Azure Arc-enabled server loses connectivity, other Azure services running on the server are impacted. For example, if you have the Azure Monitor agent installed on the server, it continues to run even when disconnected run, with logs cached for up to 5 minutes. No data is sent to Azure while the connection is offline.

Because Microsoft Sentinel depends on the Azure Monitor agent, it won't function while the server is disconnected.

## Disconnected server alerts

You can set up [Resource Health alerts](/azure/service-health/resource-health-alert-monitor-guide) to notify you when an Arc-enabled server becomes disconnected. Specify the following settings in the alert rule:

- **Resource type**: Azure Arc-enabled servers
- **Current resource status**: Unavailable
- **Previous resource status**: Available

For more information, see [Create and configure Resource Health alerts](/azure/service-health/resource-health-alert-arm-template-guide).

## Contingency options for disconnected servers

When connectivity is unavailable, you can still use local tools deployed on your machines, such as Windows Admin Center and Configuration Manager.

If connectivity is available but intermittent or reduced, such as when using an alternate internet connection, you may not be able to use all Azure Arc-enabled servers features, but you can still perform some management tasks. For instance, you may want to avoid extension installs, upgrades, or removal when connectivity is inconsistent. You can also reduce the amount of logged data and apply policies to prioritize critical traffic.
