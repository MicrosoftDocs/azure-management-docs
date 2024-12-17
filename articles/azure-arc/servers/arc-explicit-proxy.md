---
title: How to access Azure services over Azure Firewall Explicit Proxy (Public Preview)
description: Learn how to access Azure services over Azure Firewall Explicit Proxy (Public Preview).
ms.date: 12/17/2024
ms.topic: how-to
---

# Access Azure services over Azure Firewall Explicit Proxy (Public Preview)

If you want to use Azure Arc without exposing your on-premises environment to the public internet, you can use the Azure Firewall explicit proxy feature to route all Arc traffic securely through your private connection (ExpressRoute or Site-to-Site VPN) to Azure.

This article explains the steps to configure Azure Firewall with the Explicit Proxy feature as the forward proxy for your Arc-enabled resources.

## Prerequisites and network requirements

- An existing Azure VNet 
- An existing ExpressRoute or Site-to-Site VPN connection from your on-premises environment to your Azure VNet 

## Restrictions and limitations

- This solution leverages Azure Firewall Explicit Proxy as a forward proxy. The Explicit Proxy feature doesn't support TLS Inspection.
- TLS certificates can't be applied to the Azure Firewall Explicit Proxy.
- This solution can't yet be used in conjunction with [Arc gateway](arc-gateway.md).
- This solution isn't supported yet by Azure Local or Azure Arc VMs running in Azure Local.

## How the Azure Firewall Explicit Proxy feature works

Azure Arc agents can utilize a forward proxy to connect to Azure services. The Azure Firewall Explicit Proxy feature enables you to leverage an Azure Firewall within your virtual network (VNet) as the forward proxy for your Connected Machine agents. As the Azure Firewall Explicit Proxy operates within your private VNet, and you have a secure connection to it via ExpressRoute or Site-to-Site VPN, all Azure Arc traffic can be routed to its intended destination within the Microsoft network without requiring any public internet access.

:::image type="content" source="media/arc-explicit-proxy/arc-explicit-proxy-overview.png" alt-text="Diagram showing the components and flow of the Azure Firewall Explicit Proxy." lightbox="media/arc-explicit-proxy/arc-explicit-proxy-overview.png":::



