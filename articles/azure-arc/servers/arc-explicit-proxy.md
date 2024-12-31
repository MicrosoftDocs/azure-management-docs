---
title: How to access Azure services over Azure Firewall Explicit Proxy (Public Preview)
description: Learn how to access Azure services over Azure Firewall Explicit Proxy (Public Preview).
ms.date: 12/17/2024
ms.topic: how-to
---

# Access Azure services over Azure Firewall Explicit Proxy (Public Preview)

If you want to use Azure Arc without exposing your on-premises environment to the public internet, you can use the Azure Firewall Explicit Proxy feature to route all Azure Arc traffic securely through your private connection (ExpressRoute or Site-to-Site VPN) to Azure.

This article explains the steps to configure Azure Firewall with the Explicit Proxy feature as the forward proxy for your Arc-enabled resources.

## Prerequisites and network requirements

- An existing Azure VNet 
- An existing ExpressRoute or Site-to-Site VPN connection from your on-premises environment to your Azure VNet 

## Restrictions and limitations

- This solution uses Azure Firewall Explicit Proxy as a forward proxy. The Explicit Proxy feature doesn't support TLS Inspection.
- TLS certificates can't be applied to the Azure Firewall Explicit Proxy.
- This solution can't yet be used with [Arc gateway](arc-gateway.md).
- This solution isn't supported yet by Azure Local or Azure Arc VMs running in Azure Local.

## How the Azure Firewall Explicit Proxy feature works

Azure Arc agents can utilize a forward proxy to connect to Azure services. The Azure Firewall Explicit Proxy feature enables you to use an Azure Firewall within your virtual network (VNet) as the forward proxy for your Connected Machine agents. As the Azure Firewall Explicit Proxy operates within your private VNet, and you have a secure connection to it via ExpressRoute or Site-to-Site VPN, all Azure Arc traffic can be routed to its intended destination within the Microsoft network without requiring any public internet access.

:::image type="content" source="media/arc-explicit-proxy/arc-explicit-proxy-overview.png" alt-text="Diagram showing the components and flow of the Azure Firewall Explicit Proxy." lightbox="media/arc-explicit-proxy/arc-explicit-proxy-overview.png":::

Using the Azure Firewall Explicit Proxy feature to route Azure Arc traffic through your private connection requires the following steps:

1. Set up and configure the Azure Firewall.
1. Set your Azure Firewall as the forward proxy for your Arc-enabled resources.

## Step 1: Configure the Azure Firewall

### Configure the Azure Firewall resource

> [!NOTE]
> If you have an existing Azure Firewall in your VNet, you can skip this step.
> 

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/) and navigate to the **Azure Firewalls** page.

1. Select **Create** to create a new firewall.

1. Enter your **Subscription**, **Resource Group**, **Name**, and **Region**.

1. For the **Firewall SKU**, select **Standard** or **Premium** .

1. Select **Review + create**, then select **Create** to create the firewall.

For more information, see [Deploy and configure Azure Firewall](/azure/firewall/deploy-firewall-basic-portal-policy).

### Enable the Explicit Proxy (Preview) feature

1. Navigate to the Azure Firewall resource, then go to the Firewall Policy.

1. In **Settings**, navigate to the **Explicit Proxy (Preview)** pane. 

1. Select **Enable Explicit Proxy**.  

1. Enter the desired values for the HTTP and HTTPS ports.

    > [!NOTE]
    > It’s common to use *8080* for the HTTP Port, and *8443* for the HTTPS port.

1. Select **Apply** to save the changes.  

### Create an application rule

Create an application rule to allow communication to the required endpoints for your scenarios.

> [!NOTE]
> Creating an application rule is an optional step if you want to create an allowlist for your Azure Firewall Explicit Proxy.   
> 

1. Navigate to the applicable firewall policy.  

1. In **Settings**, navigate to the **Application Rules** pane.  

1. Select **Add a rule collection**.  

1. Provide a **Name** for the rule collection. 

1. Set the rule **Priority** based on other rules you may have. 

1. Provide a **Name** for the rule. 

1. For the **Source**, enter “*”, or any source IPs you may have. 

1. Set **Protocol** as **http:80,https:443**.  

1. Set **Destination** as a comma-separated list of URLs required for your scenario.
    
    For more information on the URLs required for your scenario, see [Azure Arc network requirements](/azure/azure-arc/network-requirements-consolidated?tabs=azure-cloud).    

1. Select **Add** to save the rule collection and rule.  


## Step 2: Set your Azure Firewall as the forward proxy

To set your Azure Firewall as the forward proxy while onboarding new servers:

1. [Generate the onboarding script](/azure/azure-arc/servers/onboard-portal).

1. Set **Connectivity Method** as **Proxy Server**, and set the **Proxy Server URL** as `http://<Your Azure Firewall’s Private IP>:<Explicit Proxy HTTPS Port>`.

1. Onboard your servers using the script.

To set the forward proxy for existing Arc-enabled servers, run the following command using the local Azure Connected Machine agent CLI:

`azcmagent config set proxy.url http://<Your Azure Firewall’s Private IP>:<Explicit Proxy HTTPS Port>`

## Troubleshooting

To verify that traffic is successfully being proxied via your Azure Firewall Explicit Proxy, you should first ensure that the Explicit proxy is accessible and working as expected from your network. To do so, run the following command: `curl -x <proxy IP> <target FQDN>`  

Additionally, you can view the Azure Firewall Application Rule Logs to verify traffic. Explicit proxy relies on Application rules, so all the logs are available at *AZFWApplicationRules* table, as in this example:

:::image type="content" source="media/arc-explicit-proxy/arc-explicit-proxy-troubleshooting.png" alt-text="Screenshot of the application rules table highlighting source IP, source port, destination port, and FQDN.":::

## Azure Firewall Costs 

Azure Firewall pricing is based on deployment hours and total data processed. Details on pricing for Azure Firewall can be found on the [Azure Firewall Pricing Page](https://azure.microsoft.com/pricing/details/azure-firewall/?msockid=1c55508c2bbf693b0bf545c52ad26864). 

## Integration with Private Link  

You can use this solution with Azure Private Link for Arc-enabled Servers. To use these two solutions together, it’s recommended to use the [Proxy Bypass feature](/azure/azure-arc/servers/manage-agent?tabs=windows) available for Azure Arc-enabled Servers. This way, traffic to Azure Arc endpoints that support Private Link bypasses the Explicit Proxy and have their traffic routed directly to the relevant Private Endpoint, and traffic to endpoints that don’t support Private Link route via the Explicit Proxy.

## More resources

- [Azure Firewall Explicit proxy (preview)](/azure/firewall/explicit-proxy) 

- [Demystifying Explicit proxy: Enhancing Security with Azure Firewall](https://techcommunity.microsoft.com/blog/azurenetworksecurityblog/demystifying-explicit-proxy-enhancing-security-with-azure-firewall/3873445) 

