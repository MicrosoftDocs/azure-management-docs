---
title: Use Azure Private Link to connect servers to Azure Arc by using a private endpoint
description: Learn how to use Azure Private Link to securely connect networks to Azure Arc.
ms.topic: how-to
ms.date: 04/29/2026
# Customer intent: "As a network administrator, I want to configure Azure Private Link to connect on-premises servers to Azure Arc so that I can securely manage my resources without exposing data to public networks."
---

# Use Azure Private Link to securely connect servers to Azure Arc

With [Azure Private Link](/azure/private-link/private-link-overview), you can securely link Azure platform-as-a-service (PaaS) services to your virtual network by using private endpoints. For many services, you set up an endpoint per resource. Then you can connect your on-premises or multicloud servers with Azure Arc and send all traffic over Azure [ExpressRoute](/azure/expressroute/expressroute-introduction) or a site-to-site [virtual private network (VPN) connection](/azure/vpn-gateway/vpn-gateway-about-vpngateways) instead of using public networks.

With Azure Arc-enabled servers, you can use a private link scope model to allow multiple servers or machines to communicate with their Azure Arc resources by using a single private endpoint.

This article covers when to use Azure Arc private link scope and how to set it up.

## Advantages

With Private Link, you can:

- Connect privately to Azure Arc without opening up any public network access.
- Ensure that data from the Azure Arc-enabled machine or server is accessed only through authorized private networks. This requirement also includes data from [virtual machine (VM) extensions](manage-vm-extensions.md) installed on the machine or server that provide post-deployment management and monitoring support.
- Prevent data exfiltration from your private networks by defining specific Azure Arc-enabled servers and other Azure services resources, such as Azure Monitor, which connects through your private endpoint.
- Securely connect your private on-premises network to Azure Arc by using ExpressRoute and Private Link.
- Keep all traffic inside the Microsoft Azure backbone network.

For more information, see [Key benefits of Azure Private Link](/azure/private-link/private-link-overview#key-benefits).

## How it works

Azure Arc private link scope connects private endpoints (and the virtual networks where they're contained) to an Azure resource. In this case, it's Azure Arc-enabled servers. When you enable any one of the supported VM extensions for Azure Arc-enabled servers, such as Azure Monitor, those resources connect other Azure resources, such as:

- Log Analytics workspace, which is required for Azure Automation Change Tracking and Inventory, Azure Monitor VM insights, and Azure Monitor log collection with Azure Monitor Agent.
- Azure Key Vault.
- Azure Blob Storage, which is required for Custom Script Extension.

:::image type="content" source="./media/private-link-security/private-link-topology.png" alt-text="Diagram that shows basic resource topology." border="true" lightbox="./media/private-link-security/private-link-topology.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

Connectivity to any other Azure resource from an Azure Arc-enabled server requires you to configure Private Link for each service, which is optional, but we recommend it. Private Link requires separate configuration per service.

For more information about how to configure Private Link for the Azure services listed earlier, see the articles on [Azure Automation](/azure/automation/how-to/private-link-security), [Azure Monitor](/azure/azure-monitor/logs/private-link-security), [Key Vault](/azure/key-vault/general/private-link-service), or [Blob Storage](/azure/private-link/tutorial-private-endpoint-storage-portal).

> [!IMPORTANT]
> Private Link is now generally available. Both private endpoint and private link services (service behind a standard load balancer) are generally available. Different Azure PaaS services onboard to Private Link following different schedules. For an updated status of Azure PaaS on Private Link, see [Private Link availability](/azure/private-link/availability). For known limitations, see [Private endpoint](/azure/private-link/private-endpoint-overview#limitations) and [Private link service](/azure/private-link/private-link-service-overview#limitations).

* The private endpoint on your virtual network allows it to reach Azure Arc-enabled servers endpoints through private IPs from your network's pool, instead of using the public IPs of these endpoints. In this way, you can keep using your Azure Arc-enabled servers resource without opening your virtual network to outbound traffic that wasn't requested.
* Traffic from the private endpoint to your resources goes over the Azure backbone and isn't routed to public networks.
* You can configure each of your components to allow or deny ingestion and queries from public networks. That provides a resource-level protection so that you can control traffic to specific resources.

## Prerequisites

- An Azure subscription. If you don't have one, [create a free account](https://azure.microsoft.com/free/).
- An active [ExpressRoute circuit](/azure/expressroute/expressroute-howto-linkvnet-arm) or [site-to-site VPN connection](/azure/vpn-gateway/tutorial-site-to-site-portal) between your on-premises network and an Azure virtual network.
- An Azure virtual network in the same region as your Azure Arc-enabled servers.
- [Azure Connected Machine agent version 1.4 or later](agent-release-notes.md) on each server to connect through Private Link.
- At least Contributor role on the Azure subscription or resource group, to create private link scope and private endpoint resources.
- If using Azure PowerShell, the [Az.ConnectedMachine module](/powershell/module/az.connectedmachine) must be installed. Run `Install-Module -Name Az.ConnectedMachine -AllowClobber`.
- If using Azure CLI, the [`connectedmachine` extension](/cli/azure/connectedmachine) must be installed. Run `az extension add --name connectedmachine`.

## Restrictions and limitations

The Azure Arc-enabled servers private link scope object has several limits that you should consider when you plan your Private Link setup:

- At most, one Azure Arc private link scope can be associated with a virtual network.
- An Azure Arc-enabled machine or server resource can connect to only one Azure Arc-enabled servers private link scope.
- All on-premises machines need to use the same private endpoint by resolving the correct private endpoint information, such as fully qualified domain name (FQDN) record name and private IP address. They need to use the same Domain Name System (DNS) forwarder. For more information, see [Azure private endpoint DNS configuration](/azure/private-link/private-endpoint-dns).
- The Azure Arc-enabled server and the Azure Arc private link scope must be in the same Azure region as each other. The private endpoint and the virtual network must be in the same Azure region as each other, but it can be different from the region of your Azure Arc private link scope and Azure Arc-enabled server.
- Network traffic to Microsoft Entra ID and Azure Resource Manager doesn't traverse the Azure Arc private link scope and continues to use your default network route to the internet. You can optionally [configure a resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) to send Resource Manager traffic to a private endpoint.
- Other Azure services that you use, for example, Azure Monitor, require their own private endpoints in your virtual network.
- Remote access to the server by using Windows Admin Center or SSH isn't supported over a private link at this time.

## Plan your private link setup

To connect your server to Azure Arc over a private link, you must configure your network to accomplish the following tasks:

- Deploy an Azure Arc private link scope, which controls the machines or servers that can communicate with Azure Arc over private endpoints. Associate it with your Azure virtual network by using a private endpoint.

- To resolve the private endpoint addresses, update the DNS configuration on your local network.

- Configure your local firewall to allow access to Microsoft Entra ID and Resource Manager.

- Associate the machines or servers that are registered with Azure Arc-enabled servers with the private link scope.

- Optionally, deploy private endpoints for other Azure services that manage your machine or server, such as:

  - Azure Monitor
  - Azure Automation
  - Azure Blob Storage
  - Azure Key Vault

## Network configuration

Azure Arc-enabled servers integrate with several Azure services to bring cloud management and governance to your hybrid machines or servers. Most of these services already offer private endpoints. You need to configure your firewall and routing rules to allow access to Microsoft Entra ID and Resource Manager over the internet until these services offer private endpoints.

There are two ways to allow access:

- Configure the firewall on your local network to allow outbound TCP 443 (HTTPS) access to Microsoft
  Entra ID and Azure by using the downloadable service tag files. The
  [JSON file](https://www.microsoft.com/en-us/download/details.aspx?id=56519) contains the public IP
  address ranges used by Microsoft Entra ID and Azure and is updated monthly to reflect any changes.

  The Microsoft Entra ID service tag is `AzureActiveDirectory`. The Azure service tag is
  `AzureResourceManager`. To learn how to configure your firewall rules, consult your network
  administrator and network firewall vendor.

- If your network is configured to route all internet-bound traffic through the Azure VPN or ExpressRoute circuit, you can configure the network security group (NSG) associated with your subnet in Azure. Use [service tags](/azure/virtual-network/service-tags-overview) to allow outbound TCP 443 (HTTPS) access to Microsoft Entra ID and Azure.

  The NSG rules should look like the following table:

  |Setting |Microsoft Entra ID rule | Azure rule |
  |--------|--------------|-----------------------------|
  |Source |Virtual network |Virtual network |
  |Source port ranges |* |* |
  |Destination |Service tag |Service tag |
  |Destination service tag |`AzureActiveDirectory` |`AzureResourceManager` |
  |Destination port ranges |443 |443 |
  |Protocol |TCP |TCP |
  |Action |Allow |Allow |
  |Priority |150 (Must be lower than any rules that block internet access.) |151 (Must be lower than any rules that block internet access.) |
  |Name |`AllowAADOutboundAccess` |`AllowAzOutboundAccess` |

  To create these NSG rules using the command line, use one of the following methods:

  # [Azure PowerShell](#tab/azure-powershell)

  ```azurepowershell
  $nsgName = "<nsg-name>"
  $resourceGroup = "<resource-group>"

  $nsg = Get-AzNetworkSecurityGroup -Name $nsgName -ResourceGroupName $resourceGroup

  # Microsoft Entra ID rule

  $nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "AllowAADOutboundAccess" `
    -Priority 150 `
    -Direction Outbound `
    -Access Allow `
    -Protocol Tcp `
    -SourceAddressPrefix VirtualNetwork `
    -SourcePortRange * `
    -DestinationAddressPrefix AzureActiveDirectory `
    -DestinationPortRange 443

  # Azure rule

  $nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "AllowAzOutboundAccess" `
    -Priority 151 `
    -Direction Outbound `
    -Access Allow `
    -Protocol Tcp `
    -SourceAddressPrefix VirtualNetwork `
    -SourcePortRange * `
    -DestinationAddressPrefix AzureResourceManager `
    -DestinationPortRange 443

  $nsg | Set-AzNetworkSecurityGroup
  ```

  # [Azure CLI](#tab/azure-cli)

  ```azurecli
  # Microsoft Entra ID rule

  az network nsg rule create \
    --resource-group "<resource-group>" \
    --nsg-name "<nsg-name>" \
    --name "AllowAADOutboundAccess" \
    --priority 150 \
    --direction Outbound \
    --access Allow \
    --protocol Tcp \
    --source-address-prefixes VirtualNetwork \
    --source-port-ranges "*" \
    --destination-address-prefixes AzureActiveDirectory \
    --destination-port-ranges 443

  # Azure rule

  az network nsg rule create \
    --resource-group "<resource-group>" \
    --nsg-name "<nsg-name>" \
    --name "AllowAzOutboundAccess" \
    --priority 151 \
    --direction Outbound \
    --access Allow \
    --protocol Tcp \
    --source-address-prefixes VirtualNetwork \
    --source-port-ranges "*" \
    --destination-address-prefixes AzureResourceManager \
    --destination-port-ranges 443
  ```

  ---

To understand more about the network traffic flows, see the diagram in the [How it works](#how-it-works) section of this article.

## Create an Azure Arc private link scope

# [Azure portal](#tab/azure-portal)

1. Sign in to the [Azure portal](https://portal.azure.com).

1. In the search bar, search for and select **Azure Arc Private Link Scopes**, and then select **Create**.

   :::image type="content" source="./media/private-link-security/private-scope-home.png" lightbox="./media/private-link-security/private-scope-home.png" alt-text="Screenshot that shows the Azure Arc private link scope with the Create button." border="true":::

1. On the **Basics** tab, select a subscription and resource group.

1. Enter a name for the Azure Arc private link scope. It's best to use a meaningful and clear name.

1. Optionally, to restrict all associated machines to communicate only through the private endpoint, clear the **Allow public network access** checkbox. By default, this checkbox is selected, which allows machines or servers to communicate over both private and public networks. You can change this setting after you create the scope.

1. Select the **Private endpoint** tab, and then select **Create**.

1. On the **Create private endpoint** pane:

   1. Enter a name for the endpoint.

   1. For **Integrate with private DNS zone**, select **Yes**, and let it automatically create a new private DNS zone.

      > [!NOTE]
      > If you choose **No** and prefer to manage DNS records manually, first finish setting up your private link, including this private endpoint and the private scope configuration. Then, configure your DNS according to the instructions in [Azure private endpoint DNS configuration](/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration). Make sure not to create empty records as preparation for your private link setup. The DNS records that you create can override existing settings and affect your connectivity with Azure Arc-enabled servers.
      >
      > You can't use the same virtual network/DNS zone for both Azure Arc resources that use private links and ones that don't use private links. Azure Arc resources that aren't connected to private links must resolve to public endpoints.

   1. Select **OK**.

1. Select **Review + create**.

   :::image type="content" source="./media/private-link-security/create-private-link-scope.png" alt-text="Screenshot that shows the Create Private Link Scope window." border="true":::

1. After validation passes, select **Create**.

# [Azure PowerShell](#tab/azure-powershell2)

1. Create the private link scope:

   ```azurepowershell
   New-AzConnectedMachinePrivateLinkScope `
     -ResourceGroupName "<resource-group>" `
     -Location "<location>" `
     -ScopeName "<scope-name>" `
     -PublicNetworkAccess "Enabled"
   ```

   Set `-PublicNetworkAccess` to `"Disabled"` to restrict machines to communicate only through the private endpoint.

1. Retrieve the scope resource ID for use in later steps:

   ```azurepowershell
   $scope = Get-AzConnectedMachinePrivateLinkScope `
     -ResourceGroupName "<resource-group>" `
     -ScopeName "<scope-name>"
   $scopeId = $scope.Id
   ```

1. Create the private endpoint and associate it with the scope:

   ```azurepowershell
   $privateEndpointConnection = New-AzPrivateLinkServiceConnection `
     -Name "<connection-name>" `
     -PrivateLinkServiceId $scopeId `
     -GroupId "hybridcompute"

   New-AzPrivateEndpoint `
     -ResourceGroupName "<resource-group>" `
     -Name "<endpoint-name>" `
     -Location "<location>" `
     -Subnet (Get-AzVirtualNetworkSubnetConfig -Name "<subnet-name>" -VirtualNetwork (Get-AzVirtualNetwork -Name "<vnet-name>" -ResourceGroupName "<resource-group>")) `
     -PrivateLinkServiceConnection $privateEndpointConnection
   ```

# [Azure CLI](#tab/azure-cli2)

1. Create the private link scope:

   ```azurecli
   az connectedmachine private-link-scope create \
     --resource-group "<resource-group>" \
     --location "<location>" \
     --scope-name "<scope-name>" \
     --public-network-access "Enabled"
   ```

   Set `--public-network-access` to `"Disabled"` to restrict machines to communicate only through the private endpoint.

1. Retrieve the scope resource ID for use in later steps:

   ```azurecli
   scopeId=$(az connectedmachine private-link-scope show \
     --resource-group "<resource-group>" \
     --scope-name "<scope-name>" \
     --query id -o tsv)
   ```

1. Create the private endpoint and associate it with the scope:

   ```azurecli
   az network private-endpoint create \
     --resource-group "<resource-group>" \
     --name "<endpoint-name>" \
     --location "<location>" \
     --vnet-name "<vnet-name>" \
     --subnet "<subnet-name>" \
     --private-connection-resource-id $scopeId \
     --group-id "hybridcompute" \
     --connection-name "<connection-name>"
   ```

---

## Configure on-premises DNS forwarding

Your on-premises machines or servers must be able to resolve the private link DNS records to the private endpoint IP addresses. How you configure this behavior depends on whether you're using:

- Azure private DNS zones to maintain DNS records.

- Your own DNS server on-premises and how many servers you configure.

### DNS configuration by using Azure-integrated private DNS zones

If you set up private DNS zones when you create the private endpoint, your on-premises DNS server must forward queries to Azure DNS to resolve the private endpoint addresses. Deploy a DNS forwarder in Azure, either a dedicated VM, or an Azure Firewall instance with DNS proxy enabled.

For more information, see [Azure DNS Private Resolver with on-premises DNS forwarder](/azure/private-link/private-endpoint-dns-integration#on-premises-workloads-using-a-dns-forwarder).

### Manual DNS server configuration

If you opted out of using Azure private DNS zones during private endpoint creation, you need to create the required DNS records in your on-premises DNS server.

# [Azure portal](#tab/azure-portal)

1. In the Azure portal, go to the private endpoint resource associated with your virtual network and private link scope.

1. On the service menu, under **Settings**, select **DNS configuration** to see a list of the DNS records and corresponding IP addresses that you need to set up on your DNS server. The FQDNs and IP addresses change based on the region that you selected for your private endpoint and the available IP addresses in your subnet.

# [Azure PowerShell](#tab/azure-powershell2)

Run the following command to retrieve the DNS records and IP addresses required for your DNS server:

```azurepowershell
$endpoint = Get-AzPrivateEndpoint `
  -ResourceGroupName "<resource-group>" `
  -Name "<endpoint-name>"

$endpoint.CustomDnsConfigs | Select-Object Fqdn, IpAddresses
```

# [Azure CLI](#tab/azure-cli2)

Run the following command to retrieve the DNS records and IP addresses required for your DNS server:

```azurecli
az network private-endpoint show \
  --resource-group "<resource-group>" \
  --name "<endpoint-name>" \
  --query "customDnsConfigs[].{FQDN:fqdn, IPAddresses:ipAddresses}" \
  --output table
```

---

After you have the list of FQDNs and IP addresses, follow the guidance from your DNS server vendor to add the necessary DNS zones and A records. Ensure that you select a DNS server that was appropriately scoped for your network. Every machine or server that uses this DNS server now resolves the private endpoint IP addresses. Each machine or server must be associated with the Azure Arc private link scope, or the connection is refused.

### Single server scenarios

If you plan to use private links to support only a few machines or servers, you might not want to update your entire network's DNS configuration. In this case, you can add the private endpoint host names and IP addresses to your operating system's **Hosts** file. Depending on the OS configuration, the Hosts file can be the primary or alternative method for resolving a hostname to an IP address.

# [Windows](#tab/windows)

1. Use an account with administrator privileges to open `C:\Windows\System32\drivers\etc\hosts`.

1. Add the private endpoint IPs and host names from the **DNS configuration** listing, as described in [Manual DNS server configuration](#manual-dns-server-configuration). The hosts file requires the IP address first, followed by a space and then the host name.

1. Save the file with your changes. You might need to save to another directory first, and then copy the file to the original path.

# [Linux](#tab/linux)

1. Open the `/etc/hosts` file in a text editor.

1. Add the private endpoint IPs and host names from the **DNS configuration** listing, as described in [Manual DNS server configuration](#manual-dns-server-configuration). The hosts file requires the IP address first, followed by a space and then the host name.

1. Save the file with your changes.

---

## Connect to an Azure Arc-enabled server

Using a private endpoint requires the [Azure Connected Machine agent version 1.4 or later](agent-release-notes.md). The Azure Arc-enabled servers deployment script generated in the portal downloads the latest version.

### Configure a new Azure Arc-enabled server to use Private Link

When you connect a machine or server with Azure Arc-enabled servers for the first time, you can optionally connect it to a private link scope.

> [!NOTE]
> The onboarding script must be generated from the Azure portal. After the machine is connected, you can associate it with a private link scope by using Azure PowerShell or Azure CLI. See [Configure an existing Azure Arc-enabled server](#configure-an-existing-azure-arc-enabled-server).

1. Sign in to the [Azure portal](https://portal.azure.com). In the search bar, search for and select **Azure Arc**.

1. Under **Infrastructure**, select **Machines**. In the command bar, select **Add/Create**, and then select **Add a machine**.

1. On the **Add servers with Azure Arc** page, select either **Add a single server** or **Add multiple servers** depending on your deployment scenario, and then select **Generate script**.

1. On the **Basics** page, provide the following information:

   1. Select the subscription and resource group for the machine.
   1. In the **Region** dropdown list, select the Azure region to store the machine or server metadata.
   1. In the **Operating system** dropdown list, select the operating system on which the script is configured to run.
   1. Under **Connectivity method**, select **Private endpoint** and select the Azure Arc private link scope that you created from the dropdown list.

      :::image type="content" source="./media/private-link-security/arc-enabled-servers-create-script.png" alt-text="Screenshot that shows selecting the Private endpoint connectivity option." border="true":::

   1. Select **Next: Tags**.

1. If you selected **Add multiple servers** on the **Authentication** page, select the service principal created for Azure Arc-enabled servers from the dropdown list. If you need to create a service principal for Azure Arc-enabled servers, review how to [create a service principal](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) to learn about permissions and the steps that are required to create one. Select **Next: Tags** to continue.

1. On the **Tags** page, review the default **Physical location tags** suggested and enter a value, or specify one or more custom tags to support your standards.

1. Select **Next: Download and run script**.

1. On the **Download and run script** page, review the summary information, and then select **Download**.

After you download the script, you have to run it on your machine or server by using a privileged (administrator or root) account. Depending on your network configuration, you might need to download the agent from a computer with internet access and transfer it to your machine or server. Then, modify the script with the path to the agent.

You can download the [Windows agent](https://aka.ms/AzureConnectedMachineAgent) and the [Linux agent](https://packages.microsoft.com). Look for the latest version of `azcmagent` under your OS distribution directory and installed with your local package manager.

The script returns status messages that let you know if onboarding was successful after it finishes.

[Network traffic from the Azure Connected Machine agent](network-requirements.md#urls) to Microsoft Entra ID (`login.windows.net`, `login.microsoftonline.com`, `pas.windows.net`) and Resource Manager (`management.azure.com`) continue to use public endpoints. If your server needs to communicate through a proxy server to reach these endpoints, [configure the agent with the proxy server URL](manage-agent.md#update-or-remove-proxy-settings) before you connect it to Azure. You might also need to [configure a proxy bypass](manage-agent.md#proxy-bypass-for-private-endpoints) for the Azure Arc services if your private endpoint isn't accessible from your proxy server.

### Configure an existing Azure Arc-enabled server

For Azure Arc-enabled servers that were set up before your private link scope, you can allow them to start by using the Azure Arc-enabled servers private link scope.

# [Azure portal](#tab/azure-portal)

1. In the Azure portal, go to your Azure Arc private link scope resource.

1. On the service menu, under **Configure**, select **Azure Arc resources**, and then select **+ Add**.

1. Select the servers in the list that you want to associate with the private link scope, and then select **Select** to save your changes.

   :::image type="content" source="./media/private-link-security/select-servers-private-link-scope.png" lightbox="./media/private-link-security/select-servers-private-link-scope.png" alt-text="Screenshot that shows selecting Azure Arc resources." border="true":::

# [Azure PowerShell](#tab/azure-powershell2)

1. Retrieve the private link scope resource ID:

   ```azurepowershell
   $scope = Get-AzConnectedMachinePrivateLinkScope `
     -ResourceGroupName "<resource-group>" `
     -ScopeName "<scope-name>"
   ```

1. Associate the machine with the private link scope:

   ```azurepowershell
   Update-AzConnectedMachine `
     -ResourceGroupName "<resource-group>" `
     -Name "<machine-name>" `
     -PrivateLinkScopeResourceId $scope.Id
   ```

# [Azure CLI](#tab/azure-cli2)

1. Retrieve the private link scope resource ID:

   ```azurecli
   scopeId=$(az connectedmachine private-link-scope show \
     --resource-group "<resource-group>" \
     --scope-name "<scope-name>" \
     --query id -o tsv)
   ```

1. Associate the machine with the private link scope:

   ```azurecli
   az connectedmachine update \
     --resource-group "<resource-group>" \
     --name "<machine-name>" \
     --private-link-scope-resource-id $scopeId
   ```

---

It might take up to 15 minutes for the private link scope to accept connections from the recently associated servers.

## Troubleshooting

If you run into problems, the following suggestions might help:

- Check your on-premises DNS server to verify that it's either forwarding to Azure DNS or is configured with appropriate A records in your private link zone. These lookup commands should return private IP addresses in your Azure virtual network. If they resolve public IP addresses, double-check your machine or server and network's DNS configuration.

  ```
  nslookup gbl.his.arc.azure.com
  nslookup agentserviceapi.guestconfiguration.azure.com
  ```

- For issues with onboarding a machine or server, confirm that you added the Microsoft Entra ID and Resource Manager service tags to your local network firewall. The agent needs to communicate with these services over the internet until private endpoints are available for these services.

For more troubleshooting help, see [Troubleshoot Azure private endpoint connectivity problems](/azure/private-link/troubleshoot-private-endpoint-connectivity).

## Related content

- Learn more about private endpoints in [What is a private endpoint?](/azure/private-link/private-endpoint-overview).
- Learn how to configure private links for [Azure Automation](/azure/automation/how-to/private-link-security), [Azure Monitor](/azure/azure-monitor/logs/private-link-security), [Azure Key Vault](/azure/key-vault/general/private-link-service), or [Azure Blob Storage](/azure/private-link/tutorial-private-endpoint-storage-portal).
