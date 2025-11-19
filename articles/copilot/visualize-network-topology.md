---
title: Visualize network topology using Azure Copilot
description: Use Azure Copilot to visualize specific networking resources or your entire network topology.
ms.date: 11/18/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
# Customer intent: As a network administrator, I want to visualize my network topology and specific resources, so that I can better understand relationships, troubleshoot issues, and optimize performance across my Azure environment.
---

# Visualize network topology using Azure Copilot

You can use Azure Copilot to get a visual view of your virtual networks and connected resources. Azure Copilot uses the [topology feature](/azure/network-watcher/network-insights-topology) of [Azure Network Watcher](/azure/network-watcher/network-watcher-overview), providing an interactive interface to view networking resources and their relationships in Azure. This topology can help you troubleshoot network issues by providing contextual access to Network Watcher diagnostic tools.

When you ask Azure Copilot to visualize your network resources, you can ask about a specific network resource, or about your entire network.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Visualize specific network resources

Azure Copilot can provide a visualization of specific networking resources. The visualization also provides insights related to the resource's metrics, resource health, connectivity and traffic.

Currently, Azure Copilot can provide visualizations for virtual networks and load balancers. When you ask about visualizations, Azure Copilot will prompt you to confirm which resources you want to include.

## Visualize network resources sample prompts

Here are a few examples of the kinds of prompts you can use to visualize topology of a specific virtual network or load balancer. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Visualize my Virtual Network"
- "Visualize my Load Balancer"
- "Show me topology of my Load Balancer"
- "Show topology of my Virtual Network"
- "Visualize this Virtual Network"
- "Visualize a load balancer"

## Visualize your entire network topology

Azure Copilot can show you a visualization of your entire network topology. This view shows a unified and interconnected representation of network deployment across subscriptions, regions, and resource groups, including most networking resources, Virtual Machines (VMs), and Virtual Machine Scale Sets. Insights related to it metrics, resource health, connectivity and traffic are also included.

## Visualize entire network sample prompts

Here are a few examples of the kinds of prompts you can use to visualize your entire networking topology. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Visualize my entire network"
- "Visualize my networking resources using tool of Network Watcher"
- "Show me a network topology across all my subscriptions"
- "Show me a network topology across 5 subscriptions"
- "Show me a network topology for subscription id - {sub-id}"

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.

