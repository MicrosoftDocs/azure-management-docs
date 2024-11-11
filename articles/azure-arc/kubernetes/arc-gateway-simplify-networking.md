---
title: "Simplify network configuration requirements with Azure Arc gateway (preview)"
ms.custom: devx-track-azurecli
ms.date: 11/11/2024
ms.topic: how-to
description: "The Azure Arc gateway (preview) lets you onboard Kubernetes clusters to Azure Arc, requiring access to only seven endpoints."
---

# Simplify network configuration requirements with Azure Arc Gateway (preview)

If you use enterprise proxies to manage outbound traffic, the Azure Arc gateway (preview) can help simplify the process of enabling connectivity.

The Azure Arc gateway (preview) lets you:

- Connect to Azure Arc by opening public network access to only seven fully qualified domain names (FQDNs).
- View and audit all traffic that the Arc agents send to Azure via the Arc gateway.

> [!IMPORTANT]
> Azure Arc gateway is currently in preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How the Azure Arc gateway works

The Arc gateway works by introducing two new components

The **Arc gateway resource** is an Azure Resource that serves as a common front end for Azure traffic. The gateway resource is served on a specific domain/URL. You must create this resource by following the steps outlined in this article. After you successfully create the gateway resource, this domain/URL is included in the success response.  

The **Arc Proxy** is a new component that runs as its own pod (called "Azure Arc Proxy"). This component acts as a forward proxy used by Azure Arc agents and extensions. There is no configuration required on your part for the Azure Arc Proxy. As of [version 1.21.10 of the Arc-enabled Kubernetes agents](release-notes.md), this pod is now part of the core Arc agents, and it runs within the context of an Arc-enabled Kubernetes Cluster.  

When the gateway is in place, traffic flows via the following hops: Arc Agents → Azure Arc Proxy → Enterprise Proxy → Arc gateway → Target Service.

:::image type="content" source="media/arc-gateway-simplify-networking/arc-gateway-kubernetes-architecture.svg" lightbox="media/arc-gateway-simplify-networking/arc-gateway-kubernetes-architecture.svg" alt-text="Diagram showing the architecture for Azure Arc gateway (preview) with Arc-enabled Kubernetes.":::

## Current limitations

During the public preview, the following limitations apply. Consider these factors when planning your configuration.

- TLS terminating proxies are not supported with the Arc gateway.
- You can't use ExpressRoute/site-to-site VPN or private endpoints in addition to the Arc gateway.
- There is a limit of five Arc gateway resources per Azure subscription.

You can create an Arc gateway resource by using Azure CLI or Azure PowerShell.

When you create the Arc gateway resource, you specify the subscription and resource group in which the resource is created, along with an Azure region. However, all Arc-enabled resources in the same tenant can use the resource, regardless of their own subscription or region.

## Create the Arc gateway resource

### [Azure CLI](#tab/azure-cli)

1. On a machine with access to Azure, run the following Azure CLI command:

   ```azurecli
   az extension add -n arcgateway
   ```

1. Next, run the following Azure CLI Command to create your Arc gateway resource, replacing the placeholders with your desired values:

   ```azurecli
   az arcgateway create --name <gateway's name> --resource-group <resource group> --location <region> --gateway-type public --allowed-features * --subscription <subscription name or id>
   ```

### [Azure PowerShell](#tab/azure-powershell)

1. On a machine with access to Azure, run the following Azure PowerShell command to create your Arc gateway resource, replacing the placeholders with your desired values:

   ```azurepowershell
   New-AzArcgateway 
   -name <gateway's name> 
   -resource-group <resource group> 
   -location <region> 
   -subscription <subscription name or id> 
   -gateway-type public  
   -allowed-features *
   ```

---

It generally takes about ten minutes to finish creating the Arc gateway resource.

## Confirm access to required URLs

After the resource is created successfully, the success response will include the Arc gateway URL. Ensure your Arc gateway URL and all of the URLs below are allowed in the environment where your Arc resources live.  

|URL  |Purpose  |
|---------|---------|
|`[Your URL prefix].gw.arc.azure.com`       | Your gateway URL. This URL can be obtained by running `az arcgateway list` after you create the resource.         |
|`management.azure.com`    |Azure Resource Manager Endpoint, required for ARM control channel.         |
|`<region>.obo.arc.azure.com`     |Required when [Cluster connect](conceptual-cluster-connect.md) is configured.         |
|`login.microsoftonline.com`, `<region>.login.microsoft.com`     | Microsoft Entra ID endpoint, used for acquiring identity access tokens.         |
|`gbl.his.arc.azure.com`, `<region>.his.arc.azure.com`   |The cloud service endpoint for communicating with Arc Agents. Uses short names, for example `eus` for East US.          |
|`mcr.microsoft.com`, `*.data.mcr.microsoft.com`     |Required to pull container images for Azure Arc agents.         |

## Onboard Kubernetes clusters to Azure Arc with your Arc gateway resource

### [Azure CLI](#tab/azure-cli)

1. Ensure your environment meets all of the [required prerequisites for Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli#connect-using-an-outbound-proxy-server). Since you're using Azure Arc gateway, you don't need to meet the full set of network requirements.
1. On the deployment machine, set the environment variables needed for Azure CLI to use the outbound proxy server:

   `export HTTP_PROXY=<proxy-server-ip-address>:<port>`
   `export HTTPS_PROXY=<proxy-server-ip-address>:<port>`
   `export NO_PROXY=<cluster-apiserver-ip-address>:<port>`

1. On the Kubernetes cluster, run the connect command with the `proxy-https` and `proxy-http` parameters specified. If your proxy server is set up with both HTTP and HTTPS, be sure to use `--proxy-http` for the HTTP proxy and `--proxy-https` for the HTTPS proxy. If your proxy server only uses HTTP, you can use that value for both parameters.

   `az connectedk8s connect -g <resource_group> -n <cluster_name> --gateway-resource-id <gateway_resource_id> --proxy-https <proxy_value> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --location <region>`

   > [!NOTE]
   >
   > Some network requests, such as those involving in-cluster service-to-service communication, must be separated from traffic that's routed via the proxy server for outbound communication. The `--proxy-skip-range` parameter can be used to specify the CIDR range and endpoints in a comma-separated way, so that communication from the agents to these endpoints doesn't go via the outbound proxy. At a minimum, the CIDR range of the services in the cluster should be specified as value for this parameter. For example, if `kubectl get svc -A` returns a list of services where all the services have `ClusterIP` values in the range `10.0.0.0/16`, then the value to specify for `--proxy-skip-range` is `10.0.0.0/16,kubernetes.default.svc,.svc.cluster.local,.svc`.
   >
   > `--proxy-http`, `--proxy-https`, and `--proxy-skip-range` are expected for most outbound proxy environments. `--proxy-cert` is only required if you need to inject trusted certificates expected by proxy into the trusted certificate store of agent pods.
   >
   > The outbound proxy has to be configured to allow websocket connections.

### [Azure PowerShell](#tab/azure-powershell)

1. Ensure your environment meets all of the [required prerequisites for Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-powershell#connect-using-an-outbound-proxy-server). Since you're using Azure Arc gateway, you don't need to meet the full set of network requirements.
1. Connect your Kubernetes cluster, using one of the following methods:

    - If your cluster is not behind an outbound proxy server, run the following Azure PowerShell command:

     `New-AzConnectedKubernetes -ClusterName <clustername> -ResourceGroupName <rg> -Location <region>> –GatewayResourceId <resourceId>`

   - If your cluster is behind an outbound proxy server:

     1. On the deployment machine, set the environment variables needed for Azure PowerShell to use the outbound proxy server:

       `$Env:HTTP_PROXY = "<proxy-server-ip-address>:<port>"`
       `$Env:HTTPS_PROXY = "<proxy-server-ip-address>:<port>"`
       `$Env:NO_PROXY = "<cluster-apiserver-ip-address>:<port>"`

     1. On the Kubernetes cluster, run the connect command with the proxy parameter specified:

       `New-AzConnectedKubernetes -ClusterName <cluster-name> -ResourceGroupName <resource-group> -Location <region> -HttpProxy 'http://<proxy-server-ip-address>:<port>', -HttpsProxy 'https://<proxy-server-ip-address>:<port>' -NoProxy <excludedIP>,<excludedCIDR>`

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

After your clusters have been updated to use the Arc gateway, some of the Arc endpoints that were previously allowed in your enterprise proxy or firewalls are no longer needed and can be removed. We recommend waiting at least one hour before you remove any endpoints that are no longer needed. Be sure not to remove any of the [endpoints that are required for Arc gateway](#confirm-access-to-required-urls).

## Remove the Arc gateway

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

## Monitoring traffic

To audit your gateway's traffic, view the gateway router's logs:

1. Run `kubectl get pods -n azure-arc`
1. Identify the Arc Proxy pod (its name will begin with `arc-proxy-`).
1. Run `kubectl logs -n azure-arc <Arc Proxy pod name>`
