---
title: How to simplify network configuration requirements with Azure Arc gateway (Public Preview)
description: Learn how to simplify network configuration requirements with Azure Arc gateway (Public Preview).
ms.date: 08/20/2025
ms.topic: how-to
# Customer intent: "As an IT administrator managing hybrid infrastructure, I want to simplify network configuration with Azure Arc gateway, so that I can efficiently onboard and control Arc-enabled servers through minimal endpoint access."
---

# Simplify network configuration requirements with Azure Arc gateway

If you use enterprise proxies to manage outbound traffic, the Azure Arc gateway lets you onboard infrastructure to Azure Arc using only seven endpoints. With Azure Arc gateway, you can:

- Connect to Azure Arc by opening public network access to only seven fully qualified domain names (FQDNs).
- View and audit all traffic an Azure Connected Machine agent sends to Azure via the Arc gateway.

## How Azure Arc gateway works

Azure Arc gateway consists of two main components:

- **Arc gateway resource:** An Azure resource that serves as a common front-end for Azure traffic. This gateway resource is served on a specific domain. Once the Arc gateway resource is created, the domain is returned to you in the success response.

- **Arc proxy:** A new component added to the Azure Arc agents. This component runs within the context of an Arc-enabled resource as a service called "Azure Arc Proxy". It acts as a forward proxy used by the Azure Arc agents and extensions. No configuration is required on your part for this proxy.

When the gateway is in place, traffic flows via the following hops: **Arc agents → Arc proxy → Enterprise proxy → Arc gateway  → Target service**. For more information on Arc gateway's Forwarding Protocol, see [Arc gateway Forwarding Protocol Architecture](#arc-gateway-forwarding-protocol).

:::image type="content" source="media/arc-gateway/arc-gateway-overview.png" alt-text="Diagram showing the route of traffic flow for Azure Arc gateway." lightbox="media/arc-gateway/arc-gateway-overview.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

## Current limitations

Arc gateway has the following current limitations. Consider these factors when planning your configuration.

- Proxy bypass isn't supported when Arc gateway is in use; even if you attempt to use the feature by running `azcmagent config set proxy.bypass`, traffic won't bypass the proxy.
- There's a limit of five (5) Arc gateway resources per Azure subscription.
- Arc gateway can only be used for connectivity in the Azure public cloud.
- Arc gateway is not receommended to be used in environments where TLS termination / inspection is required. If your environment requires TLS termination / inspection, it is recommended to skip TLS inspection for your Arc gateway endpoint(s). For more information, see [Arc gateway & TLS Inspection](#arc-gateway--tls-inspection).

> [!IMPORTANT]
> While Azure Arc gateway provides the connectivity required to use Azure Arc-enabled servers, you may still need to manually allow-list additional endpoints in your environment to use some extensions and services with your connected machines. For details, see [Additional scenarios](#additional-scenarios). Over time, Arc gateway will gradually cover more endpoints, further removing the need for these manual allowances.

## Planning your Arc gateway setup

- **Region choice:** Arc gateway is a global service. Runtime connectivity is delivered through Microsoft’s Azure Front Door global edge network, which automatically routes clients to the nearest point of presence for low‑latency access and seamless failover. The region you select when creating the gateway determines only the control plane (where the gateway resource and management metadata live, and where create/update/delete happens); it doesn't constrain the gateway’s runtime endpoints or performance. For example, picking East US versus West Europe doesn't change where clients connect at runtime; it only affects management-plane placement and policy/RBAC locality.
- **Arc-enabled Resources Per Arc gateway Resource:** When planning your Azure Arc deployment with Arc gateway, you must determine how many gateway resources are required for your environment. This depends on the number of resources you plan to manage in each Azure region. For Arc-enabled Servers only, a general rule of thumb is that one Arc gateway resource can handle 2,000 resources per Azure Region. If you are leveraging Arc gateway with any combination of Arc-enabled Servers, Arc-enabled Kubernetes, and Azure Local instances, please refer to our [provided formula](#arc-gateway-resource-planning-for-multiple-resource-types) to calculate the number of Arc gateway resources you need.  

## Required permissions

To create Arc gateway resources and manage their association with Arc-enabled servers, a user must have the **Arc gateway Manager** Role. 

## Create the Arc gateway resource

You can create an Arc gateway  resource by using the Azure portal, Azure CLI, or Azure PowerShell. It generally takes about 10 minutes to create the Arc gateway resource after you complete these steps.

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Azure Arc**. In the service menu, under **Management**, select **Azure Arc gateway **, then select **Create**.

1. Select the subscription and resource group where you want the Arc gateway resource to be managed within Azure. An Arc gateway resource can be used by any Arc-enabled resource in the same Azure tenant.

1. For **Name**, input the name that for the Arc gateway resource.

1. For **Location**, input the region where the Arc gateway resource should live. An Arc gateway resource can be used by any Arc-enabled Resource in the same Azure tenant.  

1. Select **Next**.

1. On the **Tags** page, optionally specify one or more custom tags to support your standards.  

1. Select **Review + create**.

1. Review your input details, and then select **Create**.

### [CLI](#tab/cli)

1. Add the Arc gateway extension to your Azure CLI:

    `az extension add -n arcgateway`

1. On a machine with access to Azure, run the following commands to create your Arc gateway resource:

    ```azurecli
    az arcgateway create `
        --gateway-name [Your gateway’s Name] `
        --resource-group <Your Resource Group> `
        --location [Location]
    ```

### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command to create your Arc gateway resource:

```powershell
    New-AzArcgateway `
        -name <gateway’s name> `
        -resource-group <resource group> `
        -location <region> `
        -subscription <subscription name or id> `
        -gateway-type public
```

---

## Confirm access to required URLs

After the resource is created successfully, the success response will include the Arc gateway URL. Ensure your Arc gateway URL and all of these URLs are allowed in the environment where your Arc resources live.

> [!IMPORTANT]
> This list has recently been updated. If you previously enabled access to these URLs, you may need to review the list and update your network configuration to ensure each endpoint is allowed.

|URL  |Purpose  |
|---------|---------|
|`<Your URL prefix>.gw.arc.azure.com`  |Your gateway URL (obtained by running `az arcgateway list` after you create your gateway resource)  |
|`management.azure.com`  |Azure Resource Manager endpoint, required for Azure Resource Manager control channel  |
|`login.microsoftonline.com`, `<region>.login.microsoft.com`  |Microsoft Entra ID endpoint for acquiring identity access tokens  |
|`gbl.his.arc.azure.com`  |The cloud service endpoint for communicating with Azure Arc agents  |
|`<region>.his.arc.azure.com`  |Used for Arc's core control channel  |
|`packages.microsoft.com`  |Required to connect Linux servers to Arc  |

## Onboard new Azure Arc resources with your Arc gateway resource

1. Generate the installation script.

    Follow the instructions at [Quickstart: Connect hybrid machines with Azure Arc-enabled servers](quick-enable-hybrid-vm.md) to create a script that automates the downloading and installation of the Azure Connected Machine agent and establishes the connection with Azure Arc.

    > [!IMPORTANT]
    > When generating the onboarding script, ensure **Public Endpoint** is selected the **Connectivity method** section, and ensure your Arc gateway resources is selected in the **Gateway Resource** dropdown. 

1. Run the installation script to onboard your servers to Azure Arc.

    In the script, the Arc gateway resource's ARM ID is shown as `--gateway-id`.

## Configure existing Azure Arc resources to use Arc gateway

You can associate existing Azure Arc resources with an Arc gateway resource by using the Azure portal, Azure CLI, or Azure PowerShell.

### [Portal](#tab/portal)

1. In the Azure portal, go to **Azure Arc - Azure Arc gateway **.

1. Select the Arc gateway resource to associate with your Arc-enabled server.

1. In the service menu for your gateway resource, select **Associated resources**.

1. Select **Add**.

1. Select the Arc-enabled server resource to associate with your Arc gateway resource.

1. Select **Apply**.

### [CLI](#tab/cli)

1. On a machine with access to Azure, run the following commands:

     ```azurecli
    az arcgateway settings update `
        --resource-group <resource_group> `
        --subscription <subscription_name> `
        --base-provider Microsoft.HybridCompute `
        --base-resource-type machines `
        --base-resource-name <server_name> `
        --gateway-resource-id <gateway_resource_id>
    ```

### [PowerShell](#tab/powershell)

1. On a machine with access to Azure, run the following commands:

    ```powershell
    Update-AzArcSetting `
        -ResourceGroupName <Resource Group> `
        -SubscriptionId <Subscription ID> `
        -BaseProvider Microsoft.HybridCompute `
        -BaseResourceType machine `
        -BaseResourceName <Server Name> `
        -GatewayResourceId <resource ID>
    ```

---

With 1.50 or earlier of the Connected Machine agent, you must also run `azcmagent config set connection.type gateway` to update your Arc-enabled server to use Arc gateway. For agent versions 1.51 and later, this step isn't required, as the operation happens automatically. We recommend using the [latest version of the Connected Machine agent](agent-release-notes.md).

## Verify successful Arc gateway set-up

On the onboarded server, run the following command: `azcmagent show`

The result should indicate the following values:

- **Agent Status** should show as **Connected**.
- **Using HTTPS Proxy** should show as **`http://localhost:40343`**.
- **Upstream Proxy** should show as your enterprise proxy (if you set one). Gateway URL should reflect your gateway resource's URL.

Additionally, to verify successful set-up, run the following command: `azcmagent check`

The result should indicate that the `connection.type` is set to gateway, and the **Reachable** column should indicate **true** for all URLs.

## Remove Arc gateway association

You can disable Arc gateway and remove the association between the Arc gateway resource and the Arc-enabled cluster. This results in the Arc-enabled cluster using direct traffic instead.

> [!NOTE]
> This operation only applies to Azure Arc gateway on Azure Arc-enabled servers, not Azure Local. If you're using Azure Arc gateway on Azure Local, see [About Azure Arc gateway for Azure Local](/azure/azure-local/deploy/deployment-azure-arc-gateway-overview) for removal information.

1. Set the connection type of the Arc-enabled Server to "direct" instead of "gateway" by running the following command:

   `azcmagent config set connection.type direct`

   > [!NOTE]
   > If you take this step, all [Azure Arc network requirements](/azure/azure-arc/network-requirements-consolidated?tabs=azure-cloud) must be met in your environment to continue using Azure Arc.

1. Detach the Arc gateway resource from the machine:

   ### [Portal](#tab/portal)

   1. In the Azure portal, go to **Azure Arc - Azure Arc gateway **.

   1. Select the Arc gateway Resource.

   1. In the service menu for your gateway resource, select **Associated resources**.

   1. Select the server.

   1. Select **Remove**.

   ### [CLI](#tab/cli)

   On a machine with access to Azure, run the following Azure CLI command:

   ```azurecli
   az arcgateway settings update `
        --resource-group <resource_group> `
        --subscription <subscription_name> `
        --base-provider Microsoft.HybridCompute `
        --base-resource-type machines `
        --base-resource-name <server_name> `
        --gateway-resource-id <gateway_resource_id>
   ```

   > [!NOTE]
   > If you’re running this Azure CLI command within Windows PowerShell, set the `--gateway-resource-id` to null.

   ### [PowerShell](#tab/powershell)

   On a machine with access to Azure, run the following PowerShell command:

   ```powershell
   Update-AzArcSetting  `
        -ResourceGroupName <Resource Group> `
        -SubscriptionId <Subscription ID> `
        -BaseProvider Microsoft.HybridCompute `
        -BaseResourceType machine `
        -BaseResourceName <Server Name> `
        -GatewayResourceId <resource ID>
   ```

   ---

### Delete an Arc gateway resource

You can delete an Arc gateway resource by using the Azure portal, Azure CLI, or Azure PowerShell. This operation may take up to 5 minutes to complete.

### [Portal](#tab/portal)

1. In the Azure portal, go to the **Azure Arc - Azure Arc gateway**.

1. Select the Arc gateway resource.

1. Select **Delete**, then confirm the deletion.

### [CLI](#tab/cli)

On a machine with access to Azure, run the following Azure CLI command:

```azurecli-interactive
az arcgateway delete `
    --resource-group <resource_group> `
    --subscription <subscription_name> `
    --gateway-name <gateway_resource_name>
```

### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command:

```powershell
Remove-AzArcGateway  
    -ResourceGroup <Resource Group> `
    -SubscriptionId <Subscription ID> `
    -GatewayName <Gateway Resource Name>
```

---

## Monitor Arc gateway traffic

You can audit your Arc gateway’s traffic by viewing the Azure Arc proxy logs.

To view Arc proxy logs on Windows:

1. Run `azcmagent logs` in PowerShell.
1. In the resulting .zip file, the `arcproxy.log` file is located in the `ProgramData\AzureConnectedMachineAgent\Log` folder.

To view Arc proxy logs on Linux:

1. Run `sudo azcmagent logs`.
1. In the resulting .zip file, the `arcproxy.log` file is located in the `/var/opt/azcmagent/log/` folder.





## Arc gateway Resource planning for Multiple Resource Types

To determine how many gateway resources you need per Azure region when you're using multiple Resource types, use the following formula:

  **Score = (Servers ÷ 20) + (K8s Clusters ÷ 10) + (Azure Local Instances ÷ 10)** 

  Where:
  - **Servers** = Total standalone servers + provisioned VMs (on Azure Local)
  - **K8s Clusters** = Total standalone Kubernetes clusters + AKS Arc clusters (on Azure Local)
  - **Azure Local Instances** = Total Azure Local deployments

  If **Score** for each region from which you intend to manage your resources  < 100, one Arc gateway resource is sufficient
  
  If **Score** for any region from which you intend to manage your resources  ≥ 100, more than one Arc gateway resource is required for that region

  The following examples have more context on this:

**Example 1:**

  | Region        | Servers | K8s Clusters | Azure Local Instances | Score Calculation                          | Score |
  |---------------|---------|--------------|-----------------------|--------------------------------------------|-------|
  | East US       | 300     | 20           | 5                     | 300/20 + 20/10 + 5/10                      | 17.5  |
  | West Europe   | 800     | 50           | 10                    | 800/20 + 50/10 + 10/10                     | 46.0  |
  | Japan East    | 100     | 5            | 2                     | 100/20 + 5/10 + 2/10                       | 5.7   |

 - Each Region's Score < 100. One Arc gateway resource is sufficient. 

**Example 2:**
  | Region          | Servers | K8s Clusters | Azure Local Instances | Score Calculation                          | Score |
  |-----------------|---------|--------------|-----------------------|--------------------------------------------|-------|
  | East US         | 6,000   | 300          | 40                    | 6000/20 + 300/10 + 40/10                   | 334.0 |
  | West Europe     | 2,500   | 120          | 25                    | 2500/20 + 120/10 + 25/10                   | 139.5 |  
  | Southeast Asia  | 900     | 30           | 8                     | 900/20 + 30/10 + 8/10                      | 48.8  |
  
  - East US Score > 100, 3 Arc gateway resources are needed to support the load in this region. 
  - West Europe's Score > 100, 2 Arc gateway resources are needed to support the load in this region.  
  - Southeast Asia's Score < 100, 1 Arc gateway resources are needed to support the load in this region.

 Note, this scenario only requires 3 gateway resources total, because calculations are based on the maximum load per region, not the combined load across all regions.



## Additional scenarios

During public preview, Arc gateway covers the endpoints required for onboarding a server, plus endpoints to support several additional Arc-enabled scenarios. Based on the scenarios you adopt, you may need to allow additional endpoints in your proxy.

### Scenarios that don’t require additional endpoints

- Windows Admin Center
- SSH
- Extended Security Updates
- Azure Extension for SQL Server

### Scenarios that require additional endpoints

Endpoints listed with the following scenarios must be allowed in your enterprise proxy when using Arc gateway:

- Azure Arc-enabled Data Services

  - `*.ods.opinsights.azure.com`
  - `*.oms.opinsights.azure.com`
  - `*.monitoring.azure.com`

- Azure Monitor Agent

  - `<log-analytics-workspace-id>.ods.opinsights.azure.com`
  - `<data-collection-endpoint>.<virtual-machine-region>.ingest.monitor.azure.com`

- Azure Key Vault Certificate Sync

  - `<vault-name>.vault.azure.net`

- Azure Automation Hybrid Runbook Worker extension

  - `*.azure-automation.net`

- Windows OS Update Extension / Azure Update Manager

  - Your environment must meet all the [prerequisites](/windows/privacy/manage-windows-11-endpoints) for Windows Update

- Microsoft Defender

  - Your environment must meet all the [prerequisites](/defender-endpoint/configure-device-connectivity) for Microsoft Defender
 
## Arc gateway Acrhitecture 

### Arc gateway Forwarding Protocol 

<img width="1280" height="720" alt="Arc Gateway Diagrams" src="https://github.com/user-attachments/assets/0d450bb0-b1b1-436e-b30c-581972a7ceee" />

### Arc gateway & TLS inspection

Arc gateway works by establishing a TLS session between Arc proxy and Arc gateway in Azure. Within this TLS session, Arc proxy sends a nested HTTP connect request to the Arc gateway resource, requesting it to forward the connection to the intended target destination. Subsequently, if the target destination itself is on TLS, an inner end-to-end TLS session is established between Arc agent and the target destination. 
 
When using terminating proxies with Arc gateway, the proxy will see the nested HTTP connect request. It may allow such a request, but it won't be able to intercept TLS encrypted traffic to the target destination unless it does nested TLS termination. This is outside the capabilities of standard TLS terminating proxies. Therefore, when using a terminating proxy, the recommendation is to skip TLS inspection for your Arc gateway endpoint. 

### Arc gateway endpoint list

To understand which endpoints prevents you from having to allow in your environment, please see INSERT ENDPOINT REPO LIST
