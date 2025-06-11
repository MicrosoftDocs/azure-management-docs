---
title: Access Azure Arc resources over Azure Firewall Explicit Proxy (public preview)
description: Learn about using Azure Arc without exposing your on-premises environment to the public internet.
ms.date: 04/22/2025
ms.topic: how-to
# Customer intent: "As a network administrator, I want to configure Azure Firewall with the Explicit Proxy feature for Azure Arc resources, so that I can securely route traffic without exposing my on-premises environment to the public internet."
---

# Access Azure services over Azure Firewall Explicit Proxy (Public Preview)

The [Azure Firewall Explicit proxy feature](/azure/firewall/explicit-proxy) can route all Azure Arc traffic securely through your private connection (ExpressRoute or Site-to-Site VPN) to Azure. This feature allows you to use Azure Arc without exposing your on-premises environment to the public internet.

This article explains the steps to configure Azure Firewall with the Explicit Proxy feature as the forward proxy for your Arc-enabled servers or Kubernetes resources.

> [!IMPORTANT]
> Azure Firewall Explicit proxy is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How the Azure Firewall Explicit proxy feature works

Azure Arc agents can use a forward proxy to connect to Azure services. The Azure Firewall Explicit proxy feature enables you to use an Azure Firewall within your virtual network (VNet) as the forward proxy for your Arc agents.

As the Azure Firewall Explicit proxy operates within your private VNet, and you have a secure connection to it via ExpressRoute or Site-to-Site VPN, all Azure Arc traffic can be routed to its intended destination within the Microsoft network, without requiring any public internet access.

:::image type="content" source="media/azure-firewall-explicit-proxy/arc-explicit-proxy-servers.png" alt-text="Diagram showing how Azure Firewall Explicit proxy works with Arc-enabled servers." lightbox="media/azure-firewall-explicit-proxy/arc-explicit-proxy-servers.png":::

:::image type="content" source="media/azure-firewall-explicit-proxy/arc-explicit-proxy-kubernetes.png" alt-text="Diagram showing how Azure Firewall Explicit proxy works with Arc-enabled Kubernetes." lightbox="media/azure-firewall-explicit-proxy/arc-explicit-proxy-kubernetes.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

### Restrictions and current limitations

- This solution uses Azure Firewall Explicit proxy as a forward proxy. The Explicit proxy feature doesn't support TLS Inspection.
- TLS certificates can't be applied to the Azure Firewall Explicit proxy.
- This solution isn't currently supported by Azure Local or Azure Arc VMs running in Azure Local.

### Azure Firewall costs

Azure Firewall pricing is based on deployment hours and total data processed. Details on pricing for Azure Firewall can be found on the [Azure Firewall Pricing page](https://azure.microsoft.com/pricing/details/azure-firewall/?msockid=1c55508c2bbf693b0bf545c52ad26864).

## Prerequisites and network requirements

To use this solution, you must have:

- An existing Azure VNet.
- An existing ExpressRoute or site-to-site VPN connection from your on-premises environment to your Azure VNet.

## Configure the Azure Firewall

Follow these steps to enable the Explicit proxy feature on your Azure Firewall.

### Create the Azure Firewall resource

If you have an existing Azure Firewall in your VNet, you can skip this section. Otherwise, follow these steps to create a new Azure Firewall resource.

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/) and navigate to the [**Azure Firewalls** page](https://portal.azure.com/#view/Microsoft_Azure_HybridNetworking/FirewallManagerMenuBlade/~/azureFirewallsMenuItem).
1. Select **Create** to create a new firewall.
1. Enter your **Subscription**, **Resource group**, **Name**, and **Region**.
1. For the **Firewall SKU**, select **Standard** or **Premium** .
1. Complete the rest of the **Basics** tab as needed for your configuration.
1. Select **Review + create**, then select **Create** to create the firewall.

For more information, see [Deploy and configure Azure Firewall](/azure/firewall/deploy-firewall-basic-portal-policy).

### Enable the Explicit proxy (preview) feature

1. Navigate to your Azure Firewall resource, then go to the Firewall Policy.
1. In **Settings**, navigate to the **Explicit Proxy (Preview)** pane.
1. Select **Enable Explicit Proxy**.
1. Enter the desired values for the HTTP and HTTPS ports.

    > [!NOTE]
    > It's common to use *8080* for the HTTP Port, and *8443* for the HTTPS port.

1. Select **Apply** to save the changes.  

### Create an application rule

If you want to create an allowlist for your Azure Firewall Explicit proxy, you can optionally create an application rule to allow communication to the required endpoints for your scenarios.

1. Navigate to the applicable firewall policy.  
1. In **Settings**, navigate to the **Application Rules** pane.  
1. Select **Add a rule collection**.  
1. Provide a **Name** for the rule collection.
1. Set the rule **Priority** based on other rules you may have.
1. Provide a **Name** for the rule.
1. For the **Source**, enter “*”, or any source IPs you may have.
1. Set **Protocol** as **http:80,https:443**.  
1. Set **Destination** as a comma-separated list of URLs required for your scenario. For details on required URLs, see [Azure Arc network requirements](/azure/azure-arc/network-requirements-consolidated?tabs=azure-cloud).
1. Select **Add** to save the rule collection and rule.  

## Set your Azure Firewall as the forward proxy

Follow these steps to set your Azure Firewall as the forward proxy for your Arc resources.

### Arc-enabled servers

To set your Azure Firewall as the forward proxy while onboarding new Arc servers:

1. [Generate the onboarding script](/azure/azure-arc/servers/onboard-portal).
1. Set **Connectivity Method** as **Proxy Server**, and set the **Proxy Server URL** as `http://<Your Azure Firewall’s Private IP>:<Explicit Proxy HTTPS Port>`.
1. Onboard your servers using the script.

To set the forward proxy for existing Arc-enabled servers, run the following command using the local Azure Connected Machine agent CLI:

```azurecli
azcmagent config set proxy.url http://<Your Azure Firewall's Private IP>:<Explicit Proxy HTTPS Port>`
```

### Arc-enabled Kubernetes

To set your Azure Firewall as the forward proxy while [onboarding new Kubernetes clusters](kubernetes/quickstart-connect-cluster.md), run the connect command with the `proxy-https` and `proxy-http` parameters specified:

```azurecli
az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-https http://<Your Azure Firewall's Private IP>:<Explicit Proxy HTTPS Port>  --proxy-http http://<Your Azure Firewall’s Private IP>:<Explicit Proxy HTTPS Port> 
```

To set the forward proxy for existing Arc-enabled Kubernetes clusters, run the following command:

```azurecli
az connectedk8s update --proxy-https http://<Your Azure Firewall’s Private IP>:<Explicit Proxy HTTPS Port>  
```

## Troubleshooting

To verify that traffic is successfully being proxied via your Azure Firewall Explicit Proxy, you should first ensure that the Explicit proxy is accessible and working as expected from your network. To do so, run the following command: `curl -x <proxy IP> <target FQDN>`  

Additionally, you can view the Azure Firewall Application rule logs to verify traffic. Explicit proxy relies on Application rules, so all the logs are available in the [AZFWApplicationRules table](/azure/azure-monitor/reference/tables/azfwapplicationrule), as shown in this example:

:::image type="content" source="media/azure-firewall-explicit-proxy/arc-explicit-proxy-troubleshooting.png" alt-text="Screenshot showing the AZFWApplicationRule information.":::

## Private Link integration

You can use Azure Firewall Explicit proxy in conjunction with Azure Private Link. To use these solutions together, configure your environment so that traffic to endpoints that don’t support Private Link route via the Explicit proxy, while allowing traffic to Azure Arc endpoints that do support Private Link to bypass the Explicit proxy and instead route traffic directly to the relevant private endpoint:

- For [Azure Private Link for Arc-enabled servers](servers/private-link-security.md), use the [Proxy Bypass feature](/azure/azure-arc/servers/manage-agent?tabs=windows#proxy-bypass-for-private-endpoints).
- For [Azure Private Link for Arc-enabled Kubernetes (preview)](kubernetes/private-link.md), include Microsoft Entra ID, Azure Resource Manager, Azure Front Door, and Microsoft Container Registry endpoints in your cluster's proxy skip range.

## Next steps

- Learn more about [Azure Firewall Explicit proxy (preview)](/azure/firewall/explicit-proxy) 
- Read [Demystifying Explicit proxy: Enhancing Security with Azure Firewall](https://techcommunity.microsoft.com/blog/azurenetworksecurityblog/demystifying-explicit-proxy-enhancing-security-with-azure-firewall/3873445) 
