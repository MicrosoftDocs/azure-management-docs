---
title: Use private connectivity for Azure Arc-enabled Kubernetes clusters with private link (preview)
ms.date: 02/25/2025
ms.topic: how-to
description: With Azure Arc, you can use a Private Link Scope model to allow multiple Kubernetes clusters to use a single private endpoint.
ms.custom: references_regions
# Customer intent: "As a cloud administrator, I want to set up private connectivity for my Azure Arc-enabled Kubernetes clusters using Private Link, so that I can securely manage resources without exposing sensitive data over public networks."
---

# Use private connectivity for Arc-enabled Kubernetes clusters with private link (preview)

[Azure Private Link](/azure/private-link/private-link-overview) allows you to securely link Azure services to your virtual network using private endpoints. This means you can connect your on-premises Kubernetes clusters with Azure Arc and send all traffic over an Azure ExpressRoute or site-to-site VPN connection instead of using public networks. In Azure Arc, you can use a Private Link Scope model to allow multiple Kubernetes clusters to communicate with their Azure Arc resources using a single private endpoint.

This document covers when to use and how to set up Azure Arc Private Link (preview).

> [!IMPORTANT]
> The Azure Arc Private Link feature is currently in PREVIEW in all regions where Azure Arc-enabled Kubernetes is present, except South East Asia.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Advantages

With Private Link you can:

* Connect privately to Azure Arc without opening up any public network access.
* Ensure data from the Arc-enabled Kubernetes cluster is only accessed through authorized private networks.
* Prevent data exfiltration from your private networks by defining specific Azure Arc-enabled Kubernetes clusters and other Azure services resources, such as Azure Monitor, that connects through your private endpoint.
* Securely connect your private on-premises network to Azure Arc using ExpressRoute and Private Link.
* Keep all traffic inside the Microsoft Azure backbone network.

For more information, see [Key benefits of Azure Private Link](/azure/private-link/private-link-overview#key-benefits).

## How it works

Azure Arc Private Link Scope connects private endpoints (and the virtual networks they're contained in) to an Azure resource, in this case Azure Arc-enabled Kubernetes clusters. When you enable an Arc-enabled Kubernetes [cluster extension](extensions-release.md), then connection to other Azure resources may be required for these scenarios. For example, with Azure Monitor, logs collected from the cluster are sent to Log Analytics workspace.

Connectivity to the other Azure resources from an Arc-enabled Kubernetes cluster listed earlier requires configuring Private Link for each service. For an example, see [Private Link for Azure Monitor](/azure/azure-monitor/logs/private-link-security).

## Current limitations

Consider these current limitations when planning your Private Link setup.

* You can associate only one Azure Arc Private Link Scope with a virtual network.
* An Azure Arc-enabled Kubernetes cluster can only connect to one Azure Arc Private Link Scope.
* All on-premises Kubernetes clusters need to use the same private endpoint by resolving the correct private endpoint information (FQDN record name and private IP address) using the same DNS forwarder. For more information, see [Azure Private Endpoint private DNS zone values](/azure/private-link/private-endpoint-dns). The Azure Arc-enabled Kubernetes cluster, Azure Arc Private Link Scope, and virtual network must be in the same Azure region. The Private Endpoint and the virtual network must also be in the same Azure region, but this region can be different from that of your Azure Arc Private Link Scope and Arc-enabled Kubernetes cluster.
* Traffic to Microsoft Entra ID, Azure Resource Manager, and Microsoft Container Registry service tags must be allowed through your on-premises network firewall during the preview.
* Azure Arc resources utilizing Private Link and those that do not cannot share the same VNET or DNS zone. Azure Arc resources without Private Link must resolve to public endpoints.
* Other Azure services that you use, such as Azure Monitor, may require their own private endpoints in your virtual network.

    > [!NOTE]
    > The [Cluster Connect](conceptual-cluster-connect.md) (and hence the [Custom location](custom-locations.md)) feature is currently not supported on Azure Arc-enabled Kubernetes clusters with private connectivity enabled. Network connectivity using private links for Azure Arc services such as Azure Arc-enabled data services and Azure container Apps on Azure Arc are also currently not supported.

## Cluster extensions that support network connectivity through private links

On Azure Arc-enabled Kubernetes clusters configured with private links, these extensions support end-to-end connectivity through private links:

* [GitOps (Flux v2)](conceptual-gitops-flux2.md#gitops-with-private-link)
* [Azure Monitor](/azure/azure-monitor/logs/private-link-security)

## Planning your Private Link setup

To connect your Kubernetes cluster to Azure Arc over a private link, configure your network as follows:

1. The subscription in which Private link Scope is present should be registered to `Microsoft.kubernetes` and `Microsoft.kubernetes.Configurations`.
1. Establish a connection between your on-premises network and an Azure virtual network using a [site-to-site VPN](/azure/vpn-gateway/tutorial-site-to-site-portal) or [ExpressRoute](/azure/expressroute/expressroute-howto-linkvnet-arm) circuit.
1. Deploy an Azure Arc Private Link Scope, which controls which Kubernetes clusters can communicate with Azure Arc over private endpoints, and associate it with your Azure virtual network using a private endpoint.
1. Update the DNS configuration on your local network to resolve the private endpoint addresses.
1. Configure your local firewall to allow access to Microsoft Entra ID, Azure Resource Manager, and Microsoft Container Registry.
1. Associate the Azure Arc-enabled Kubernetes clusters with the Azure Arc Private Link Scope.
1. Optionally, deploy private endpoints for other Azure services used with your Azure Arc-enabled Kubernetes cluster, such as Azure Monitor.

The rest of this article assumes you already set up your ExpressRoute circuit or site-to-site VPN connection.

## Network configuration

Azure Arc-enabled Kubernetes integrates with several Azure services to bring cloud management and governance to your hybrid Kubernetes clusters. Most of these services already offer private endpoints. However, you need to configure your firewall and routing rules to allow access to Microsoft Entra ID and Azure Resource Manager over the internet until those services offer private endpoints. You must also allow access to Microsoft Container Registry (and AzureFrontDoor.FirstParty as a precursor for Microsoft Container Registry) to pull images & Helm charts to enable services like Azure Monitor, and for initial setup of Azure Arc agents on Kubernetes clusters.

There are two ways to enable this configuration:

* If your network is configured to route all internet-bound traffic through the Azure VPN or ExpressRoute circuit, you can configure the network security group (NSG) associated with your subnet in Azure to allow outbound TCP 443 (HTTPS) access to Microsoft Entra ID, Azure Resource Manager, Azure Front Door, and Microsoft Container Registry using [service tags](/azure/virtual-network/service-tags-overview). The NSG rules should look like the following:

    | Setting                 | Microsoft Entra ID rule                                                 | Azure Resource Manager rule                                   | AzureFrontDoorFirstParty rule                                 | Microsoft Container Registry rule                            |
    |-------------------------|---------------------------------------------------------------|---------------------------------------------------------------|---------------------------------------------------------------|--------------------------------------------------------------- |
    | Source                  | Virtual Network      | Virtual Network       | Virtual Network                 | Virtual Network |
    | Source Port ranges      | *              | *                | *                  | * |
    | Destination             | Service Tag             | Service Tag               | Service Tag                | Service Tag |
    | Destination service tag | `AzureActiveDirectory`          | `AzureResourceManager`       | `AzureFrontDoor.FirstParty`       | `MicrosoftContainerRegistry` |
    | Destination port ranges | 443          | 443            | 443                 | 443 |
    | Protocol                | TCP      | TCP              | TCP             | TCP |
    | Action                  | Allow                 | Allow                     | Allow (Both inbound and outbound)             | Allow |
    | Priority                | 150 (must be lower than any rules that block internet access) | 151 (must be lower than any rules that block internet access) | 152 (must be lower than any rules that block internet access) | 153 (must be lower than any rules that block internet access) |
    | Name                    | `AllowAADOutboundAccess`   | `AllowAzOutboundAccess`    | `AllowAzureFrontDoorFirstPartyAccess`      | `AllowMCROutboundAccess` |

* Alternately, configure the firewall on your local network to allow outbound TCP 443 (HTTPS) access to Microsoft Entra ID, Azure Resource Manager, and Microsoft Container Registry, and inbound and outbound access to `AzureFrontDoor.FirstParty` using downloadable service tag files. The JSON file contains all the public IP address ranges used by Microsoft Entra ID, Azure Resource Manager, `AzureFrontDoor.FirstParty`, and Microsoft Container Registry and is updated monthly to reflect any changes. The Microsoft Entra service tag is `AzureActiveDirectory`, Azure Resource Manager's service tag is `AzureResourceManager`, Microsoft Container Registry's service tag is `MicrosoftContainerRegistry`, and Azure Front Door's service tag is `AzureFrontDoor.FirstParty`. Consult your network administrator and network firewall vendor to learn how to configure your firewall rules.

## Create an Azure Arc Private Link Scope

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Go to **Create a resource** in the Azure portal and search for **Azure Arc Private Link Scope**, then select **Create**. Alternately, go directly to the [**Azure Arc Private Link Scopes** page](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.HybridCompute%2FprivateLinkScopes) in the Azure portal, then select **Create Azure Arc Private link scope**.
1. Select a subscription and resource group. During the preview, your virtual network and Azure Arc-enabled Kubernetes clusters must be in the same subscription as the Azure Arc Private Link Scope.
1. Give the Azure Arc Private Link Scope a name.
1. To require every Arc-enabled Kubernetes cluster associated with this Azure Arc Private Link Scope to send data to the service through the private endpoint, select **Allow public network access**. If you do so, Kubernetes clusters associated with this Azure Arc Private Link Scope can communicate with the service over both private or public networks. You can change this setting after creating the scope as needed.

   :::image type="content" source="media/private-link/create-private-link-scope.png" lightbox="media/private-link/create-private-link-scope.png"  alt-text="Screenshot of the Azure Arc Private Link Scope creation screen in the Azure portal.":::

1. Select **Review + create**.
1. After the validation completes, select **Create**.

### Create a private endpoint

Once your Azure Arc Private Link Scope is created, you need to connect it with one or more virtual networks using a private endpoint. The private endpoint exposes access to the Azure Arc services on a private IP in your virtual network address space.

The Private Endpoint on your virtual network allows it to reach Azure Arc-enabled Kubernetes cluster endpoints through private IPs from your network's pool, instead of using to the public IPs of these endpoints. That allows you to keep using your Azure Arc-enabled Kubernetes clusters without opening your VNet to unrequested outbound traffic. Traffic from the Private Endpoint to your resources goes through Microsoft Azure, and isn't routed to public networks.

1. In the Azure portal, navigate to your Azure Arc Private Link Scope resource.
1. In the service menu, under **Configure**, select **Private Endpoint connections**.
1. Select **Add** to start the endpoint create process. You can also approve connections that were started in the Private Link center by selecting them, then selecting **Approve**.

    :::image type="content" source="media/private-link/create-private-endpoint.png" lightbox="media/private-link/create-private-endpoint.png" alt-text="Screenshot of the Private Endpoint connections screen in the Azure portal.":::

1. Select the subscription and resource group, and enter a name for the endpoint. Select the same region as your virtual network.
1. Select **Next: Resource**.
1. On the **Resource** page, ensure that the following values are selected:
   1. **Subscription**: the subscription that contains your Azure Arc Private Link Scope resource.
   1. **Resource type**: `Microsoft.HybridCompute/privateLinkScopes`.
   1. **Resource**: the Azure Arc Private Link Scope that you created earlier.
   1. **Target sub-resource**: `hybridcompute`.
1. Select **Next: Virtual Network**.
1. On the **Virtual Network** page:
    1. Select the virtual network and subnet from which you want to connect to Azure Arc-enabled Kubernetes clusters.
    1. Select **Next: DNS**.
1. On the **DNS** page:
    1. For **Integrate with private DNS zone**, select **Yes**. A new Private DNS Zone is created.

        Alternately, if you prefer to manage DNS records manually, select **No**, then complete setting up your Private Link, including this private endpoint and the Private Scope configuration. Then, [configure your DNS](#manual-dns-server-configuration) according to the instructions in [Azure Private Endpoint private DNS zone values](/azure/private-link/private-endpoint-dns). Make sure not to create empty records as preparation for your Private Link setup. The DNS records you create can override existing settings and impact your connectivity with Arc-enabled Kubernetes clusters.

        > [!IMPORTANT]
        >The same VNET/DNS zone can't be used for both Arc resources using private link and ones which don't use private link. Arc resources which aren't private link connected must resolve to public endpoints.

    1. Select **Review + create**.
    1. Let validation pass.
    1. Select **Create**.

## Configure on-premises DNS forwarding

Your on-premises Kubernetes clusters need to be able to resolve the private link DNS records to the private endpoint IP addresses. Configuration steps vary depending on whether you use Azure private DNS zones to maintain DNS records, or your own DNS server on-premises.

### DNS configuration using Azure-integrated private DNS zones

If you selected **Yes** for **Integrate with private DNS zone** when creating the private endpoint, your on-premises Kubernetes clusters must be able to forward DNS queries to the built-in Azure DNS servers to resolve the private endpoint addresses correctly. You need a DNS forwarder in Azure (either a purpose-built VM or an Azure Firewall instance with DNS proxy enabled), after which you can configure your on-premises DNS server to forward queries to Azure to resolve private endpoint IP addresses.

For more information, see [Azure Private Resolver with on-premises DNS forwarder](/azure/private-link/private-endpoint-dns-integration#on-premises-workloads-using-a-dns-forwarder).

### Manual DNS server configuration

If you opted out of using Azure private DNS zones during private endpoint creation, you must create the required DNS records in your on-premises DNS server.

1. In the Azure portal, navigate to the private endpoint resource associated with your virtual network and private link scope.
1. From the service menu, under **Settings**, select **DNS configuration** to see a list of the DNS records and corresponding IP addresses that you need to set up on your DNS server. The FQDNs and IP addresses will change based on the region you selected for your private endpoint and the available IP addresses in your subnet.
1. Follow the guidance from your DNS server vendor to add the necessary DNS zones and A records to match the table in the portal. Ensure that you select a DNS server that is appropriately scoped for your network. Every Kubernetes cluster that uses this DNS server now resolves the private endpoint IP addresses and must be associated with the Azure Arc Private Link Scope, or the connection will be refused.

## Configure private links

> [!NOTE]
> Configuring private links for Azure Arc-enabled Kubernetes clusters is supported starting from version 1.3.0 of the `connectedk8s` CLI extension, but requires Azure CLI version greater than 2.3.0. If you use a version greater than 1.3.0 for the `connectedk8s` CLI extension, we have introduced validations to check and successfully connect the cluster to Azure Arc only if you're running Azure CLI version greater than 2.3.0.  

You can configure private links for an existing Azure Arc-enabled Kubernetes cluster or when onboarding a Kubernetes cluster to Azure Arc for the first time using the following command:

```azurecli
az connectedk8s connect -g <resource-group-name> -n <connected-cluster-name> -l <location> --enable-private-link true --private-link-scope-resource-id <pls-arm-id>
```

| Parameter name | Description |
| -------------- | ----------- |
| `--enable-private-link` |Enables the private link feature if set to `True`. |
| `--private-link-scope-resource-id` | ID of the private link scope resource created earlier. For example: `/subscriptions//resourceGroups//providers/Microsoft.HybridCompute/privateLinkScopes/` |

For Azure Arc-enabled Kubernetes clusters that were set up prior to configuring the Azure Arc private link scope, configure private links in the Azure portal using the following steps:

1. In the Azure portal, navigate to your Azure Arc Private Link Scope resource.
1. From the service menu, under **Configure**, select **Azure Arc resources**. Then, select **Add**.
1. You'll see all of the Arc-enabled Kubernetes clusters in the same subscription and region as your Private Link Scope. Check the box for each Kubernetes cluster that you want to associate with the Private Link Scope. When you're finished, choose **Select** to save your changes.

## Troubleshooting

If you run into problems, the following suggestions may help:

* Check your on-premises DNS server to verify that it's either forwarding to Azure DNS, or is configured with appropriate A records in your private link zone. These lookup commands should return private IP addresses in your Azure virtual network. If they resolve public IP addresses, double check your server and network's DNS configuration.

  ```console
  nslookup gbl.his.arc.azure.com
  nslookup agentserviceapi.guestconfiguration.azure.com
  nslookup dp.kubernetesconfiguration.azure.com
  ```

* For issues onboarding your Kubernetes cluster, confirm that you added the Microsoft Entra ID, Azure Resource Manager, AzureFrontDoor.FirstParty, and Microsoft Container Registry service tags to your local network firewall.

For more troubleshooting tips, see [Troubleshoot Azure Private Endpoint connectivity problems](/azure/private-link/troubleshoot-private-endpoint-connectivity).

## Next steps

* Learn more about [Azure Private Endpoint](/azure/private-link/private-link-overview).
* Learn how to [troubleshoot Azure Private Endpoint connectivity problems](/azure/private-link/troubleshoot-private-endpoint-connectivity).
* Learn how to [configure Private Link for Azure Monitor](/azure/azure-monitor/logs/private-link-security).
