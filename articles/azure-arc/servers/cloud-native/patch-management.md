---
title: Cloud-native patch management with Azure Arc-enabled servers
description: Azure Update Manager provides a unified patch management solution for your Arc-enabled servers.
ms.date: 08/19/2025
ms.topic: concept-article
# Customer intent: "As an administrator managing a hybrid cloud environment, I want to use Azure Arc-enabled servers for patch management and assessment, so I can ensure my Windows and Linux servers are up to date and compliant ."
---

# Cloud-native patch management with Azure Arc-enabled servers

Patching operating systems is a critical (if unglamorous) duty for any system administrators. Traditionally, one might use Windows Server Update Services (WSUS) or System Center Configuration Manager (SCCM) to manage Windows updates and schedule maintenance windows for servers. In the cloud-native approach, [Azure Update Manager](/azure/update-manager/overview) takes on this role, providing a unified patch management solution for Windows and Linux servers, whether they're Azure virtual machines (VMs) or Arc-enabled on-premises machines.

Cloud-native OS patching via Azure Update Manager means centralized, policy-driven patching with greater flexibility and insight. You get features like maintenance windows and automation for complex workflows. Because of innovations like hotpatching, you can reduce downtime during updates. For a system administrator, familiar jobs are made easier: you set the rules, and Azure carries out the patching. You can reduce time spent manually managing updates, and can monitor all your environments via a centralized compliance report.

For example, suppose you have a large number of servers connected to Azure Arc. After you enable Azure Update Manager across your servers, you can define maintenance configurations. For example, you might create Group A (Dev) to get patched every week, Group B (Prod) to patch monthly with a two-hour window, and so on. You then assign servers (or whole resource groups) to these schedules. You can optionally specify pre or post tasks using Automation runbooks. When patches are delivered, you can monitor progress in the Azure portal, then later check to see if any updates are still outstanding.

> [!TIP]
> To deploy applications to Azure VMs, [Virtual Machine Applications](/azure/virtual-machines/vm-applications) (VM Applications) offers a modern, flexible approach to managing applications. Application installation is decoupled from base VM images, eliminating the need to rebuild and republish VM images for every application change. Currently, Azure Arc-enabled servers don't support VM Applications, but [Run Command](../run-command.md) scripts and customized Machine Configuration can help deploy applications at scale to your on-premises environment.

Let's take a closer look at the features that enable a modernized approach to patch management for your Arc-enabled servers.

## Unified patch compliance dashboard

Azure Update Manager gives you a single dashboard to monitor update compliance across all your servers. You can see which updates are missing on each machine, filter by critical/security updates, and get an overview of your patch posture. This dashboard shows similar info to that found in WSUS reports or SCCM's compliance reporting, but integrated into the Azure portal and including on-premises servers alongside native Azure VMs. For a hybrid environment, this capability can save time by reducing the need to aggregate reports from multiple systems.

## Scheduling and maintenance windows

You can create maintenance configurations that define when patches should be applied to groups of machines. For example, you might set a window every Saturday at 2 AM for your development servers, and another window on the last Sunday of the month for production servers. Azure Update Manager allows scheduling updates within these customer-defined maintenance windows This approach is very analogous to SCCM's Maintenance Windows or change management windows. You have control over frequency, day/time, and which updates to include. When the window arrives, Azure orchestrates installing the updates on the target machines. You can also use Azure Policy to automatically schedule newly added machines into a default patch window.

## Pre and post event update scripts

One advanced feature is the ability to hook in [pre and post events](/azure/update-manager/pre-post-scripts-overview) around a maintenance window. You can automate tasks to run before patching begins and after patching completes. For example, common pre-tasks might be "snapshot the VM" or "stop specific services" before updates. Post-tasks might be "start those services back up" or "send an email notification" after patching. You might also want to turn on VMs that were powered off (so they can get patched) and then turn them off again after updates. The granular control provided by pre and post tasks can help you maintain maximum uptime, even when updates are needed.

Azure Update Manager uses Event Grid to trigger pre and post events. This means you can invoke [Azure Functions](/azure/azure-functions/functions-overview), [Azure Logic Apps](/azure/logic-apps/logic-apps-overview), or [Azure Automation runbooks](/azure/automation/overview) for pre and post events. These capabilities enable application-aware patching; just like you might use custom scripts in SCCM or orchestrated patch sequences, you can achieve the same capabilities in Azure across your entire hybrid estate.

## Granular patch controls and sequencing

Azure Update Manager lets you group machines in a maintenance configuration, and even sequence the order of patching if needed. For example, you can use separate maintenance configurations with offset to ensure that web servers are patched first, then app servers, and then database servers.

You can also use inclusion or exclusion lists for updates, letting you enforce granular requirements. For example, you could use these lists to exclude a particular update known to cause issues, or to apply only security-related updates to your servers.

## Hotpatching

Azure Update Manager integrates [Hotpatch](/windows-server/get-started/hotpatch) for Arc-enabled servers running Windows Server 2025 Datacenter Edition or Standard Edition. Hotpatching enables certain security updates to be installed without restarting the machines, eliminating the downtime that would be required by a restart. As a system administrator, you'd still schedule a maintenance window, but if all patches in a cycle can be delivered through Hotpatching, that window might pass with no reboot needed, maximizing service availability.

Azure Update Manager manages the Hotpatch cycles, with Hotpatch updates delivered monthly. A reboot is only required for new baselines (Cumulative Updates), typically delivered every three months.

## OS and hybrid support

Azure Update Manager isn't just for Windows. Arc-enabled servers that run Linux are also supported. Update Manager can apply updates from Linux package repositories, and even do reboots if needed. You get unified reporting for your Linux machines alongside Windows. So if you're also responsible for some Ubuntu or RHEL servers, Azure now covers those natively. This broad support underscores the "cloud admin" advantage, with one tool for all OS versions.

## Linux auto-update service

You can also use your Linux distribution’s built-in service to automatically update the installed Azure connected machine agent package. This option allows you to integrate the agent package update seamlessly with your package manager and configured repositories. Since package updates are published to [Microsoft package repository](/linux/packages), you can update the agent using your usual distribution tools, whether that's manual, remotely managed, or on a schedule managed by tools like dnf-automatic for RPM-based distributions and unattended-upgrades for Debian-based distributions.
