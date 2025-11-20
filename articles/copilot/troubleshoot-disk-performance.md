---
title: Troubleshoot disk performance using Azure Copilot
description: Learn how to use Azure Copilot to help troubleshoot issues with disk performance.
ms.date: 11/20/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
# Customer intent: As a cloud administrator, I want to analyze disk performance issues using an AI tool, so that I can identify and resolve bottlenecks impacting my virtual machines and applications efficiently.
---

# Troubleshoot disk performance using Azure Copilot

Azure Copilot can help you troubleshoot issues with your disk performance when your application requires higher performance than what you have configured for your VM and disks. For more details about these issues, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance#how-does-disk-performance-work).

When you ask Azure Copilot questions about disk performance, it prompts you to select the VM and disks that are struggling with performance, along with the time period when the problems started. For best results, when specifying a time frame for the analysis, ensure that the disk and VM have been active during that entire period.

Using the information you provide, Azure Copilot analyses your current configuration and performance metrics to determine whether your application is experiencing slowness due to reaching the configured performance limits for the VM or disk. It then provides a summary of the analysis and recommendations to resolve the performance issue, which you can apply directly in the Azure portal through Azure Copilot's guided recommendations. The recommendations include a primary recommendation, which will be the least disruptive to your application, along with other possible options. For example, Azure Copilot might determine that you can improve performance by upgrading your disks, enabling on-demand bursting, or adding another disk to your VM.

To diagnose performance issues when your application requires higher performance than what you have configured for your VM and disks, Azure Copilot analyzes the following [disk metrics](/azure/virtual-machines/monitor-vm-reference#metrics):  

- Data Disk IOPS Consumed Percentage
- Data Disk Bandwidth Consumed Percentage
- OS Disk IOPS Consumed Percentage
- OS Disk Bandwidth Consumed Percentage
- VM Cached and Uncached IOPS Consumed Percentage
- VM Cached and Uncached Bandwidth Percentage

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to troubleshoot disk performance. Modify these prompts based on your real-life scenarios, or try additional prompts to meet your needs.

- "Why is my disk slow?"
- "Why is my VM attached to disks running slow?"
- "Are there any bottlenecks affecting my disk performance?"
- "What are the reasons for my disk's slow performance?"
- "Help me with my VM-Disk performance."

## Examples

When you ask Azure Copilot, "**Why is my disk slow**", Copilot runs an analysis of your disk and VM performance metrics to determine if your application performance is being capped due to requesting more IOPS or throughput than what is allotted for the virtual machines or attached disks. It starts by asking you to select the affected VM.

:::image type="content" source="media/troubleshoot-disk-performance/troubleshoot-disk-slow.png" lightbox="media/troubleshoot-disk-performance/troubleshoot-disk-slow.png" alt-text="Screenshot of Azure Copilot responding to a prompt about slow disk performance.":::

After you select a VM, you're prompted to select one or more disks for Azure Copilot to analyze.

:::image type="content" source="media/troubleshoot-disk-performance/troubleshoot-disk-slow-select.png" lightbox="media/troubleshoot-disk-performance/troubleshoot-disk-slow-select.png"alt-text="Screenshot of Azure Copilot prompting to select disks to analyze.":::

Next, tell Azure Copilot when the issues began. enter an exact or approximate timeframe. For best results, be sure the VM and disk you selected have been active during the period you specify.

Azure Copilot then shows you the VM and disks you selected and the metrics to be analyzed. After you confirm, Azure Copilot runs the analysis to determine if your application performance is being capped due to requesting more IOPS or throughput than what is allotted for the virtual machines or attached disks.

:::image type="content" source="media/troubleshoot-disk-performance/troubleshoot-disk-slow-analyze.png" alt-text="Screenshot of Azure Copilot preparing to run an analysis of slow disk performance.":::

If Azure Copilot detects a performance issue with your VM-Disk configuration due to hitting IOPS or throughput limits, it provides you with a summary of the analysis, a primary recommendation based on the least downtime to your application, and other recommendation options. You can also view metric details from the analysis metrics by selecting **Show additional details**. This option provides more information such as the VM IOPS/MBPS limits, total time period when limits were hit, and the top three time intervals when disk limits were hit. If your VM has caching enabled, the VM IOPS/MBPS limits shown here reflect your cached limits.

If you choose to enact any of the recommendation options provided by Azure Copilot, you're directed to the location in the Azure portal where you can implement the recommendation. If you're dissatisfied with the recommendations, but still need help, you can choose to submit a support request to get more assistance.

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn [how disk performance works when you combine Azure Virtual Machines and Azure Disks](/azure/virtual-machines/disks-performance).
- Learn about building high-performance applications by using [Azure premium storage](/azure/virtual-machines/premium-storage-performance).
- Learn about [performance tiers for premium SSD managed disks](/azure/virtual-machines/disks-change-performance).
