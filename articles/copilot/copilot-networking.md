---
title: Azure Copilot for networking
description: Learn how Azure Copilot can assist you in planning, designing, operating, and troubleshooting your Azure network with ease and efficiency.
author: JnHs
ms.author: jenhayes
ms.reviewer: duau
ms.date: 11/21/2025
ms.service: copilot-for-azure
ms.topic: how-to
ms.collection:
  - ce-skilling-ai-copilot
ms.custom:
  - ignite-2024
  - ai-assisted
#customer intent: As a network administrator, I want to utilize an intelligent assistant to design, troubleshoot, and secure my Azure network, so that I can enhance operational efficiency and effectively resolve connectivity issues.
---

# Azure Copilot for networking

Azure Copilot for networking is an AI-powered assistant that helps you design, migrate, monitor, optimize, and troubleshoot your Azure network infrastructure. It provides intelligent assistance for Azure networking services, offering contextual responses and actionable insights based on Microsoft's extensive networking knowledge and your specific Azure environment.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## How Azure Copilot works for networking

Azure Copilot for networking integrates directly into the Azure portal, providing natural language interaction to help you manage your network resources. When you ask Copilot a networking question, it analyzes your Azure environment, accesses Microsoft's networking documentation and best practices, and provides contextual guidance specific to your infrastructure.

Copilot leverages your existing Azure role-based access control (RBAC) permissions, ensuring it has the same access to resources as you do. This means Copilot can help you troubleshoot and manage only the resources you have permission to access.

## Features of Azure Copilot for networking

Azure Copilot for networking provides capabilities across multiple scenarios:

### Design, plan, and migrate

[**Network product information queries**](#information) - This skill enables Azure Copilot for Networking to answer questions about Azure Networking products and services using information from published documentation.

[**Network product selection and architecture guidance queries**](#selection) - This skill allows Azure Copilot for Networking to assist with questions about product usage, product selection for your networking needs, and guidance on network planning, resilience, and migration from on-premises environments.

Currently, product selection guidance responses are limited to:

- Azure Load Balancer
- Azure Firewall

And resiliency related queries are limited to the following networking services:

- Azure Application Gateway
- Azure Firewall
- Azure Front Door
- Azure Load Balancer
- Azure NAT Gateway
- Azure Private Endpoint
- Azure Public IP
- Azure Traffic Manager
- Azure Virtual Network Gateways (ExpressRoute and VPN)

## Monitor, secure, and troubleshoot

[**Network resource inventory, topology, traffic path queries**](#inventory) - Azure Copilot can discover customer network resources, network topology, and traffic paths from source to destination. Currently, Azure Copilot can respond to questions about network topology and traffic paths with topology maps and network connectivity graphs.

[**Network connectivity troubleshooting and service diagnostics queries**](#troubleshoot) - Azure Copilot can perform customer network troubleshooting across various connectivity, configuration, and environmental issues across your network data and control plane. Troubleshooting is supported at the network level and individual network service level. Azure Copilot supports RBAC (Role-base access control) and has the same access to resources as you do.

## Example prompts

You can interact with Copilot in Azure through the Copilot pane in the Azure portal. The following sections provide example prompts for different networking scenarios. Modify these prompts based on your real-life scenarios, or try additional prompts to explore Copilot's capabilities.

<a id="information"></a>

### Get network product information

Use these prompts to get information about Azure networking products and services:

- "What are the different types of private network connectivity services offered by Azure?"
- "What is the difference between Azure Application Gateway and Azure Front Door?"
- "What is an Azure Firewall and how is it different from Azure Network Security Groups?"

<a id="selection"></a>

### Get help with network product selection and architecture guidance

Use these prompts to get assistance with product selection and architecture planning:

- "Suggest an Azure Firewall SKU for my topology"
- "Which type of load balancer should I use?"
- "Suggest a network architecture when migrating to Azure"
- "Is my application gateway resilient?"
- "How to make my gateway highly available?"

<a id="inventory"></a>

### Understand network resource inventory, topology, and traffic path information

Use these prompts to discover and visualize your network resources:

**Data path visualization:**
- "What is the data path between my source VM and destination VM?"
- "Show me the data path between my VM and storage account"

**Network inventory:**
- "List all my Azure networking services in my subscription"
- "Help me discover my network inventory"
- "How large is my network?"
- "Display the network resources in my subscription"

**Resource metrics:**
- "How many flows are going through the gateway?"
- "How many VMs are behind the gateway?"

When you ask about data paths, Copilot presents a resource selection screen from your subscription. After you select the source and destination resources, Copilot discovers and visualizes the data path, showing all network elements and services in the path.

:::image type="content" source="./media/copilot-networking/trace-path-result.png" alt-text="Screenshot showing Azure Copilot displaying the path your traffic takes from the source VM to the destination VM, including all traversed network services." lightbox="./media/copilot-networking/trace-path-result.png":::

<a id="troubleshoot"></a>

### Troubleshoot network connectivity and diagnose service issues

Use these prompts to troubleshoot connectivity and configuration issues:

**Connectivity troubleshooting:**
- "Why can't my VM connect to the internet?"
- "Why is my storage account not reachable from my VM?"
- "Why can't I connect my on-premises VM to Azure VM?"
- "Why am I unable to send mail and SMTP is failing?"

**Virtual network peering:**
- "Why does the Azure portal show that my peering connection is established but we aren't receiving any traffic?"
- "Troubleshoot my virtual network peerings"

**Resource health and deployment:**
- "What is the health status of my NIC and its public IP?"
- "Why did the deployment of ExpressRoute Gateway fail?"
- "Is some NSG blocking my traffic to the internet from my VM in Azure?"
- "Troubleshoot the virtual network to find any gateways and their associated public IPs that are missing a service tag"

**DDoS protection:**
- "Who are the top 10 DDoS attackers from my VM?"
- "How many times has my public IP been DDoS attacked in the past 14 days?"
- "List the top DDoS attack vectors on my public IP"

For Azure Firewall-specific queries, see [Copilot for Azure Firewall](/azure/firewall/firewall-copilot).

## How to use Copilot for network troubleshooting

Copilot guides you through an interactive troubleshooting process:

1. **Describe the problem** - Ask your connectivity or performance question
2. **Select resources** - Choose the relevant resources (VMs, storage accounts, ExpressRoute circuits, gateways)
3. **Provide details** - Enter requested information (IP addresses, ports, protocols)
4. **Review analysis** - Copilot examines the data path and identifies issues
5. **Apply fixes** - Follow the recommended steps to resolve the problem

Copilot analyzes your network infrastructure, examining:

- Network Security Groups (NSG) rules and effective security rules
- Route tables and user-defined routes (UDR)
- Azure Firewall policies and rules
- Virtual network peering configurations
- Service endpoints and private endpoints
- ExpressRoute and VPN gateway connectivity
- Load balancer configurations
- DDoS protection settings

## Limitations

Copilot in Azure for networking has the following limitations:

- For organizations with large or complex networks, queries might take longer than usual to process
- Copilot has the same access permissions as you do through Azure RBAC, so it can only help with resources you have permission to access
- Product selection guidance is currently limited to Azure Load Balancer and Azure Firewall
- Resiliency queries support a specific set of networking services (see the [Features](#features-of-copilot-in-azure-networking) section for the complete list)

For information about general limitations of Azure Copilot, see [Current limitations](capabilities.md#current-limitations).

## Related content

- [Capabilities of Microsoft Copilot in Azure](capabilities.md)
- [Azure Networking overview](/azure/networking/fundamentals/networking-overview)
- [Copilot for Azure Firewall](/azure/firewall/firewall-copilot)
