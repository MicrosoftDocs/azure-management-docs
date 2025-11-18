---
title: Design, troubleshoot, and secure networks using Azure Copilot
description: Learn how Azure Copilot can assist you in planning, designing, operating, and troubleshooting your Azure network with ease and efficiency.
ms.date: 04/21/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2024
ms.author: jenhayes
author: JnHs
# Customer intent: As a network administrator, I want to utilize an intelligent assistant to design, troubleshoot, and secure my Azure network, so that I can enhance operational efficiency and effectively resolve connectivity issues.
---

# Design, troubleshoot, and secure networks using Azure Copilot

Azure Copilot is an intelligent assistant designed to help you design, migrate, monitor, and optimize your Azure environment. Azure Copilot can answer questions about Azure networking services and troubleshoot network connectivity issues. It can also help you plan and design your network using Azure networking products and services and help troubleshoot network connectivity issues. It offers contextual responses and actionable insights based on Microsoft's extensive networking knowledge and your Azure environment.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

Azure Copilot currently provides capabilities to work with network resources in multiple scenarios, described in the following sections.

## Design, plan, and migrate

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

## Sample prompts and examples

The following are some sample prompts and examples for each of the scenarios.

### <a name = "information"></a>Get network product information

Here are a few examples of the kinds of prompts you can use to get information about your network. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "What are the different types of private network connectivity services offered by Azure?"
- "What is the difference between Azure Application Gateway and Azure Front Door?"
- "What is an Azure Firewall and how is it different from Azure Network Security Groups?"

In this example, the prompt **"What is the difference between Application Gateway and Azure Front Door"** provides a detailed comparison between Azure Application Gateway and Azure Front Door.

:::image type="content" source="./media/network-management/compare-application-gateway-front-door.png" alt-text="Screenshot of Azure Copilot describing the difference between Azure Application Gateway and Azure Front Door.":::

### <a name = "selection"></a>Get help with network product selection and architecture guidance

Here are a few examples of the kinds of prompts you can use to get help with network product selection and architecture guidance. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Suggest an Azure Firewall SKU for my topology."
- "Which type of load balancer should I use?"
- "Suggest a network architecture when migrating to Azure."
- "Is my application gateway resilient?"
- "How to make my gateway highly available?"

In this example, the prompt **"Suggest an Azure Firewall SKU for my topology"** results in a request to provide more information about your use case, so that Azure Copilot can suggest the right Azure Firewall SKU.

:::image type="content" source="./media/network-management/firewall-sku-suggestion.png" alt-text="Screenshot of Azure Copilot suggesting an Azure Firewall SKU for my topology.":::

In this example, the prompt **"Is my application gateway resilient?"** analyzes the selected Azure Application Gateway and suggests ways to make it more resilient.

:::image type="content" source="./media/network-management/application-gateway-resiliency.png" alt-text="Screenshot of Azure Copilot analyzing the resiliency of an Azure Application Gateway." lightbox="./media/network-management/application-gateway-resiliency.png":::

### <a name = "inventory"></a>Understand network resource inventory, topology, and traffic path information

Here are a few examples of the kinds of prompts you can use to understand  network resource inventory, topology, and traffic paths. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "What is the data path between my source VM and destination VM?"
- "Show me the data path between my VM and storage account."
- "List all my Azure networking services in my subscription."
- "How many flows are going through the gateway?"
- "How many VMs are behind the gateway?"
- "Help me discover my network inventory."
- "How large is my network?"
- "Display the network resources in my subscription."

In this example, the prompt **"What is the data path between my source VM and destination VM?"**  results in a request for you to select the source and destination VMs by presenting the resource selection choice screen from your subscription. Once you select the source and destination VMs, Azure Copilot discovers the data path between the source and destination, drawing a connected graph showing all the network elements/services in the path.

:::image type="content" source="./media/network-management/trace-path-result.png" alt-text="Screenshot showing Azure Copilot displaying the path your traffic takes from the source VM to the destination VM, including all traversed network services." lightbox="./media/network-management/trace-path-result.png":::

If you say **"Troubleshoot my virtual network peerings"**, Azure Copilot asks you to select the Virtual Network. Then, it analyzes the state, status, and configuration information of all the peered connections to see if there are any configuration or data path issues impacting the configured virtual network peerings, and displays the findings.

:::image type="content" source="./media/network-management/troubleshooting-analysis.png" alt-text="Screenshot of Copilot  in Azure analyzing the Virtual Network peering and displaying the findings.":::

### <a name = "troubleshoot"></a>Respond to network connectivity troubleshooting and service diagnostics queries

Here are a few examples of the kinds of prompts you can use to troubleshoot network connectivity and service diagnostics. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Why can't my VM connect to the internet?"
- "Why is my storage account not reachable from my VM?"
- "Why can’t I connect my on-premises VM to Azure VM?"
- "Why does the Azure portal show that my peering connection is established but we aren't receiving any traffic?"
- "Troubleshoot my virtual network peerings."
- "What is the health status of my NIC and its public IP"
- "Why did the deployment of ExpressRoute Gateway fail?"
- "Is some NSG blocking my traffic to the internet from my VM in Azure?"
- "Why am I unable to send mail and SMTP is failing?"
- "Troubleshoot the virtual network to find any gateways and their associated public IPs that are missing a service tag."
- "Who are the top 10 DDoS attackers from my VM"
- "How many times has my public IP been DDoS attacked in the past 14 days"
- "List the top DDoS attack vectors on my public IP"
- "[Has there been any malicious traffic intercepted by my Azure Firewall](/azure/firewall/firewall-copilot)"
 
When you ask **"Why can't my VM connect to the internet?"**, Azure Copilot asks you to confirm the source VM in Azure from where you're trying to connect to the internet. It then analyzes the path your traffic takes to the internet and identifies any configuration or data path issues that are blocking the traffic from reaching the internet.

Next, you're prompted to enter the internet destination IP address:

:::image type="content" source="./media/network-management/destination-ip-address-internet.png" alt-text="Screenshot of Azure Copilot prompting for the internet destination IP address for troubleshooting.":::

Finally, Azure Copilot analyzes the path and identifies issues impacting the connectivity:

:::image type="content" source="./media/network-management/troubleshoot-internet-result.png" alt-text="Screenshot of Azure Copilot analyzing the path and identifying issues impacting the internet connectivity." lightbox="./media/network-management/troubleshoot-internet-result.png":::

In this example, the prompt **"Why can’t I connect from my on-premise VM to my Azure VM?"** results in a request for you to select the ExpressRoute circuit that connects your on-premises site to Azure. Azure Copilot then analyzes the data path to your Azure VM to identify any configuration or data path issues that may be blocking the traffic from reaching your Azure VM.

:::image type="content" source="./media/network-management/select-expressroute-circuit.png" alt-text="Screenshot of Azure Copilot prompting to select the ExpressRoute circuit to a VM." lightbox="./media/network-management/select-expressroute-circuit.png":::

Next, Azure Copilot asks you to confirm the destination VM, then analyzes the data path to the VM and displays the possible reasons for the connectivity issue:

:::image type="content" source="./media/network-management/expressroute-connectivity-result.png" alt-text="Screenshot of Azure Copilot analyzing the data path to the VM and showing the possible reasons for the connectivity issue.":::

In this example, the prompt **"Why is my storage account not reachable from my VM?"** results in a request for you to select the source VM from where you're trying to connect to the storage account, and confirm which storage account is unreachable. Azure Copilot then analyzes the path your traffic takes to the storage account and identifies any configuration or data path issues that are blocking the traffic from reaching the storage account.

:::image type="content" source="./media/network-management/select-storage-account.png" alt-text="Screenshot showing Azure Copilot prompting to select the storage account for connectivity troubleshooting." lightbox="./media/network-management/select-storage-account.png":::

Finally, Azure Copilot analyzes the path and identifies issues impacting the connectivity to the storage account:

:::image type="content" source="./media/network-management/storage-account-analysis-result.png" alt-text="Screenshot showing Azure Copilot analyzing the path and identifying issues impacting the connectivity to the storage account." lightbox="./media/network-management/storage-account-analysis-result.png":::

## Current limitations

For organizations with large or complex networks, queries may take longer than usual.

For information about general limitations of Azure Copilot, see [Current limitations](capabilities.md#current-limitations).

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure Networking](/azure/networking/fundamentals/networking-overview).