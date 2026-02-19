---
title: "Simplify network configuration requirements with Azure Arc gateway"
ms.custom: devx-track-azurecli
ms.date: 02/17/2025
ms.topic: how-to
description: "The Azure Arc gateway lets you onboard Kubernetes clusters to Azure Arc with access to a reduced number of endpoints."
# Customer intent: As a network administrator, I want to configure the Azure Arc gateway for my Kubernetes clusters, so that I can simplify network access requirements and manage outbound traffic through enterprise proxies efficiently.
---

# Simplify network configuration requirements with Azure Arc Gateway

If your environment requires restricting outbound traffic by using an enterprise proxy or firewall, the Azure Arc gateway can help simplify the process of enabling connectivity.

By using the Azure Arc gateway, you can:

- Connect to Azure Arc by opening public network access to only nine fully qualified domain names (FQDNs).
- View and audit all traffic that an Azure Connected Machine agent sends to Azure via the Azure Arc gateway.

## How the Azure Arc gateway works

The Arc gateway works by introducing two new components.

The **Arc gateway resource** is an Azure resource that serves as a common front end for Azure traffic. The gateway resource is served on a specific domain/URL. You must create this resource by following the steps outlined in this article. After you successfully create the gateway resource, the domain/URL is included in the success response.  

The **Arc Proxy** is a new component that runs as its own pod (called "Azure Arc Proxy"). This component acts as a forward proxy used by Azure Arc agents and extensions. There's no configuration required on your part for the Azure Arc Proxy. As of [version 1.21.10 of the Arc-enabled Kubernetes agents](release-notes.md), this pod is now part of the core Arc agents, and it runs within the context of an Arc-enabled Kubernetes Cluster.  

When the gateway is in place, traffic flows via the following hops: Arc Agents → Azure Arc Proxy → Enterprise Proxy → Arc gateway → Target Service.

:::image type="content" source="media/arc-gateway-simplify-networking/arc-gateway-kubernetes-architecture.png" lightbox="media/arc-gateway-simplify-networking/arc-gateway-kubernetes-architecture.png" alt-text="Diagram showing the architecture for Azure Arc gateway with Arc-enabled Kubernetes.":::

## Current limitations

Azure Arc gateway currently has the following limitations. Consider these factors when planning your configuration.

- Each Azure subscription has a limit of five Azure Arc gateway resources.
- Azure Arc gateway supports connectivity only in the Azure public cloud.
- Azure Arc gateway isn't recommended for use in environments where Transport Layer Security (TLS) termination or inspection is required. If your environment requires TLS termination or inspection, skip TLS inspection for your Azure Arc gateway endpoint (`<your URL prefix>.gw.arc.azure.com`). For more information, see [Azure Arc gateway and TLS inspection](#azure-arc-gateway-and-tls-inspection).

> [!IMPORTANT]
> Azure Arc gateway provides the connectivity required to use Azure Arc-enabled Kubernetes, but you may need to enable additional endpoints in order to use some extensions and services with your clusters. For details, see [Additional scenarios](#additional-scenarios).

## Plan your Azure Arc gateway setup

Before you configure Azure Arc gateway, there are a few factors to consider.

### Region choice

Runtime connectivity is delivered through the Azure Front Door global edge network, which automatically routes clients to the nearest point of presence for low-latency access and seamless failover. The region that you select when you create the gateway contains the gateway resource and management metadata. The region doesn't constrain the gateway's runtime endpoints or performance, or where clients connect at runtime.

### Azure Arc-enabled resources per Azure Arc gateway

When you plan your Azure Arc deployment with Azure Arc gateway, consider how many gateway resources your environment requires. This decision depends on the number of resources that you plan to manage in each Azure region.

For Azure Arc-enabled Kubernetes resources only, a general rule is that one Azure Arc gateway resource can handle 1,000 resources per Azure region. However, you might use Azure Arc gateway with a combination of Azure Arc-enabled servers, Azure Arc-enabled Kubernetes clusters, and Azure Local instances. [We provide a formula](/azure/azure-arc/servers/arc-gateway#azure-arc-gateway-resource-planning-for-multiple-resource-types) to help you calculate the number of Azure Arc gateway resources that you need across your environment.

## Required permissions

To create Arc gateway resources and manage their association with Arc-enabled Kubernetes clusters, you need the following permissions:

- `Microsoft.Kubernetes/connectedClusters/settings/default/write`
- `Microsoft.hybridcompute/gateways/read`
- `Microsoft.hybridcompute/gateways/write`

## Create the Arc gateway resource

You can create an Arc gateway resource by using Azure CLI or Azure PowerShell.

When you create the Arc gateway resource, you specify the subscription and resource group in which the resource is created, along with an Azure region. However, all Arc-enabled resources in the same tenant can use the resource, regardless of their own subscription or region. Each subscription that contains Arc-enabled resources that use an Azure Arc gateway must have the Microsoft.Kubernetes resource provider registered.

### [Azure CLI](#tab/azure-cli)

1. On a machine with access to Azure, run the following Azure CLI command:

   ```azurecli
   az extension add -n arcgateway
   ```

1. Next, run the following Azure CLI Command to create your Arc gateway resource, replacing the placeholders with your desired values:

   ```azurecli
   az arcgateway create --name <gateway name> --resource-group <resource group> --location <region> --gateway-type public --allowed-features * --subscription <subscription name or id>
   ```

### [Azure PowerShell](#tab/azure-powershell)

1. On a machine with access to Azure, run the following Azure PowerShell command to create your Arc gateway resource. Replace the placeholders with your desired values:

   ```azurepowershell
   New-AzArcgateway 
   -name <gateway name> 
   -resource-group <resource group> 
   -location <region> 
   -subscription <subscription name or id> 
   -gateway-type public  
   -allowed-features *
   ```

---

It generally takes about 30 minutes to finish creating the Arc gateway resource.

## Confirm access to required URLs

After the resource is created successfully, the success response includes the Arc gateway URL. Ensure your Arc gateway URL and all of the URLs in the following table are allowed in the environment where your Arc resources live.  

|URL  |Purpose  |
|---------|---------|
|`<Your URL prefix>.gw.arc.azure.com`       | Your gateway URL. You can get this URL by running `az arcgateway list` after you create the resource.         |
|`management.azure.com`    |Azure Resource Manager Endpoint, required for ARM control channel.         |
|`<region>.obo.arc.azure.com`     |Required when [Cluster connect](conceptual-cluster-connect.md) is configured.         |
|`login.microsoftonline.com`, `<region>.login.microsoft.com`     | Microsoft Entra ID endpoint, used for acquiring identity access tokens.         |
|`gbl.his.arc.azure.com`, `<region>.his.arc.azure.com`   |The cloud service endpoint for communicating with Arc Agents. Uses short names, for example `eus` for East US.          |
|`mcr.microsoft.com`, `*.data.mcr.microsoft.com`     |Required to pull container images for Azure Arc agents.         |

## Onboard Kubernetes clusters to Azure Arc with your Arc gateway resource

### [Azure CLI](#tab/azure-cli)

1. Ensure your environment meets all of the [required prerequisites for Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli#connect-using-an-outbound-proxy-server). Since you're using Azure Arc gateway, you don't need to meet the full set of network requirements.
1. On the deployment machine, set the environment variables needed for Azure CLI to use the outbound proxy server:

   `export HTTP_PROXY=<proxy-server-ip-address>:<port>
   export HTTPS_PROXY=<proxy-server-ip-address>:<port>
   export NO_PROXY=<cluster-apiserver-ip-address>:<port>`

1. On the Kubernetes cluster, run the connect command with the `proxy-https` and `proxy-http` parameters specified. If your proxy server is set up with both HTTP and HTTPS, be sure to use `--proxy-http` for the HTTP proxy and `--proxy-https` for the HTTPS proxy. If your proxy server only uses HTTP, you can use that value for both parameters.

   `az connectedk8s connect -g <resource_group> -n <cluster_name> --gateway-resource-id <gateway_resource_id> --proxy-https <proxy_value> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --location <region>`

   > [!NOTE]
   >
   > Some network requests, such as those involving in-cluster service-to-service communication, must be separated from traffic that's routed via the proxy server for outbound communication. Use the `--proxy-skip-range` parameter to specify the CIDR range and endpoints in a comma-separated way, so that communication from the agents to these endpoints doesn't go via the outbound proxy. At a minimum, specify the CIDR range of the services in the cluster as the value for this parameter. For example, if `kubectl get svc -A` returns a list of services where all the services have `ClusterIP` values in the range `10.0.0.0/16`, then the value to specify for `--proxy-skip-range` is `10.0.0.0/16,kubernetes.default.svc,.svc.cluster.local,.svc`.
   >
   > Most outbound proxy environments require `--proxy-http`, `--proxy-https`, and `--proxy-skip-range`. Only use `--proxy-cert` if you need to inject trusted certificates that the proxy expects into the trusted certificate store of agent pods.
   >
   > The outbound proxy must be configured to allow websocket connections.

### [Azure PowerShell](#tab/azure-powershell)

1. Ensure your environment meets all of the [required prerequisites for Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-powershell#connect-using-an-outbound-proxy-server). Since you're using Azure Arc gateway, you don't need to meet the full set of network requirements.
1. Connect your Kubernetes cluster, using one of the following methods:

    - If your cluster isn't behind an outbound proxy server, run the following Azure PowerShell command:

     `New-AzConnectedKubernetes -ClusterName <clustername> -ResourceGroupName <rg> -Location <region> –GatewayResourceId <resourceId>`

   - If your cluster is behind an outbound proxy server:

     1. On the deployment machine, set the environment variables needed for Azure PowerShell to use the outbound proxy server:

          `$Env:HTTP_PROXY = "<proxy-server-ip-address>:<port>"`
          `$Env:HTTPS_PROXY = "<proxy-server-ip-address>:<port>"`
          `$Env:NO_PROXY = "<cluster-apiserver-ip-address>:<port>"`

     1. On the Kubernetes cluster, run the connect command with the proxy parameter specified:

       `New-AzConnectedKubernetes -ClusterName <cluster-name> -ResourceGroupName <resource-group> -Location <region> -HttpProxy 'http://<proxy-server-ip-address>:<port>' -HttpsProxy 'https://<proxy-server-ip-address>:<port>' -NoProxy <excludedIP>,<excludedCIDR>`

---

## Configure existing clusters to use the Arc gateway

To update existing clusters so that they use the Arc gateway, run the following command:

### [Azure CLI](#tab/azure-cli)

```azurecli
az connectedk8s update -g <resource_group> -n <cluster_name> --gateway-resource-id <gateway_resource_id>
```

To verify that the update was successful, run the following command and confirm that the response is `true`:

```azurecli
 az connectedk8s show -g <resource_group> -n <cluster_name> --query 'gateway.enabled' 
```

### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$connectedCluster = Get-AzConnectedKubernetes -ClusterName <clustername> -ResourceGroupName <rg> 
Get-AzConnectedKubernetes -InputObject $connectedCluster -GatewayResourceId <resourceId> 
```

---

After you update your clusters to use the Arc gateway, you can remove some of the Arc endpoints that your enterprise proxy or firewalls previously allowed. Wait at least one hour before you remove any endpoints. Don't remove any of the [endpoints that are required for Arc gateway](#confirm-access-to-required-urls).

## Remove the Arc gateway

> [!IMPORTANT]
> The step described here only applies to Arc gateway on Arc-enabled Kubernetes. For details on detaching Arc gateway on Azure Local, see [About Azure Arc gateway for Azure Local](/azure/azure-local/deploy/deployment-azure-arc-gateway-overview#detach-or-change-the-arc-gateway-association-from-the-machine).

To disable Arc gateway and remove the association between the Arc gateway resource and the Arc-enabled cluster, run the following command:

### [Azure CLI](#tab/azure-cli)

```azurecli
az connectedk8s update -g <resource_group> -n <cluster_name> --disable-gateway 
```

### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzConnectedKubernetes -ClusterName <cluster_name> -ResourceGroupName <resource_group> | Set-AzConnectedKubernetes -DisableGateway   
```

---

## Monitor traffic

To audit your gateway's traffic, view the gateway router's logs:

1. Run `kubectl get pods -n azure-arc`.
1. Identify the Arc Proxy pod (its name starts with `arc-proxy-`).
1. Run `kubectl logs -n azure-arc <Arc Proxy pod name>`.

## Additional scenarios

Arc gateway covers endpoints required for onboarding a cluster, and a portion of endpoints required for additional Arc-enabled scenarios. Based on the scenarios you adopt, you might need to allow additional endpoints in your proxy.

When Arc gateway is in use, you must also allow all endpoints listed for any of the following scenarios:

- [Container insights in Azure Monitor](/azure/azure-monitor/containers/kubernetes-monitoring-firewall):
  - `*.ods.opinsights.azure.com`
  - `*.oms.opinsights.azure.com`
  - `*.monitoring.azure.com`
- [Azure Key Vault](/azure/key-vault/general/access-behind-firewall):
  - `<vault-name>.vault.azure.net`
- [Azure Policy](/azure/governance/policy/concepts/policy-for-kubernetes):
  - `data.policy.core.windows.net`
  - `store.policy.core.windows.net`
- [Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-containers-enable?pivots=defender-for-container-arc&toc=%2Fazure%2Fazure-arc%2Fkubernetes%2Ftoc.json&bc=%2Fazure%2Fazure-arc%2Fkubernetes%2Fbreadcrumb%2Ftoc.json&tabs=aks-deploy-portal%2Ck8s-deploy-asc%2Ck8s-verify-asc%2Ck8s-remove-arc%2Caks-removeprofile-api):
  - `*.ods.opinsights.azure.com`
  - `*.oms.opinsights.azure.com`
- [Azure Arc-enabled data services](/azure/azure-arc/network-requirements-consolidated?tabs=azure-cloud)
  - `*.ods.opinsights.azure.com`
  - `*.oms.opinsights.azure.com`
  - `*.monitoring.azure.com`

## Azure Arc gateway architecture

To learn more about the architecture of Azure Arc gateway, review the following information.

### Azure Arc gateway forwarding protocol

The following diagram illustrates the forwarding protocol used by Azure Arc gateway.

:::image type="content" source="media/arc-gateway-simplify-networking/arc-gateway-forwarding-protocol.png" lightbox="media/arc-gateway-simplify-networking/arc-gateway-forwarding-protocol.png" alt-text="Diagram that shows the forwarding protocol architecture for Azure Arc gateway with Azure Arc-enabled Kubernetes.":::

### Azure Arc gateway and TLS inspection

Azure Arc gateway works by establishing a TLS session between Azure Arc proxy and Azure Arc gateway in Azure. Within this TLS session, Azure Arc proxy sends a nested HTTP connect request to Azure Arc gateway resource. The connect request instructs the resource to forward the connection to the intended target destination. Then, if the target destination itself is on TLS, an inner end-to-end TLS session is established between the Azure Arc agent and the target destination.

When you use terminating proxies with Azure Arc gateway, the proxy sees the nested HTTP connect request. It might allow such a request, but it can't intercept TLS encrypted traffic to the target destination unless it does nested TLS termination. This behavior is outside the capabilities of standard TLS terminating proxies. When you use a terminating proxy, skip TLS inspection for your Azure Arc gateway endpoint.

### Endpoints accessible through Azure Arc gateway

Arc Gateway uses a set of endpoints to enable all Arc features to function seamlessly. Currently, this set includes over 200 endpoints, which represent the cumulative requirements for all supported capabilities. For the full list, see [Azure Arc gateway endpoints](../servers/arc-gateway-endpoints.md).

Some endpoints use wildcards to simplify connectivity and ensure feature coverage. We recommend reviewing these endpoints with your network security team to confirm they align with your organization's policies. These endpoints are essential for secure and reliable operation of Arc services.
