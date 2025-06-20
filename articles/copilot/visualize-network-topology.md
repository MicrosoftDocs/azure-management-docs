---
title: Visualize network topology using Microsoft Copilot in Azure
description: Use Microsoft Copilot in Azure to visualize specific networking resources or your entire network topology.
ms.date: 04/08/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
# Customer intent: As a network administrator, I want to visualize my network topology and specific resources, so that I can better understand relationships, troubleshoot issues, and optimize performance across my Azure environment.
---

# Visualize network topology using Microsoft Copilot in Azure

You can use Microsoft Copilot in Azure to get a visual view of your virtual networks and connected resources. Copilot in Azure uses the [topology feature](/azure/network-watcher/network-insights-topology) of [Azure Network Watcher](/azure/network-watcher/network-watcher-overview), providing an interactive interface to view networking resources and their relationships in Azure. This topology can help you troubleshoot network issues by providing contextual access to Network Watcher diagnostic tools.

When you ask Copilot in Azure to visualize your network resources, you can ask about a specific network resource, or about your entire network.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Visualize specific network resources

Copilot in Azure can provide a visualization of specific networking resources. The visualization also provides insights related to the resource's metrics, resource health, connectivity and traffic.

Currently, Copilot in Azure can provide visualizations for virtual networks and load balancers. When you ask about visualizations, Copilot in Azure will prompt you to confirm which resources you want to include.

## Visualize network resources sample prompts

Here are a few examples of the kinds of prompts you can use to visualize topology of a specific virtual network or load balancer. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Visualize my Virtual Network"
- "Visualize my Load Balancer"
- "Show me topology of my Load Balancer"
- "Show topology of my Virtual Network"
- "Visualize this Virtual Network"
- "Visualize a load balancer"

## Visualize your entire network topology

Copilot in Azure can show you a visualization of your entire network topology. This view shows a unified and interconnected representation of network deployment across subscriptions, regions, and resource groups, including most networking resources, Virtual Machines (VMs), and Virtual Machine Scale Sets. Insights related to it metrics, resource health, connectivity and traffic are also included.

## Visualize entire network sample prompts

Here are a few examples of the kinds of prompts you can use to visualize your entire networking topology. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Visualize my entire network"
- "Visualize my networking resources using tool of Network Watcher"
- "Show me a network topology across all my subscriptions"
- "Show me a network topology across 5 subscriptions"
- "Show me a network topology for subscription id - {sub-id}"

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.

