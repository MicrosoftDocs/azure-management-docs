---
title: Simplify Network Configuration Requirements with Azure Arc Gateway
description: Learn how to simplify network configuration requirements with Azure Arc gateway.
ms.date: 11/12/2025
ms.topic: how-to
# Customer intent: "As an IT administrator managing hybrid infrastructure, I want to simplify network configuration with Azure Arc gateway so that I can efficiently onboard and control Azure Arc-enabled servers through minimal endpoint access."
---

# Simplify network configuration requirements with Azure Arc gateway

If you use enterprise proxies to manage outbound traffic, Azure Arc gateway lets you onboard infrastructure to Azure Arc by using only seven endpoints. With Azure Arc gateway, you can:

- Connect to Azure Arc by opening public network access to only seven fully qualified domain names.
- View and audit all traffic that an Azure Connected Machine agent sends to Azure via Azure Arc gateway.

## How Azure Arc gateway works

Azure Arc gateway consists of two main components:

- **Azure Arc gateway resource:** An Azure resource that serves as a common front-end for Azure traffic. This gateway resource is served on a specific domain. After the Azure Arc gateway resource is created, the domain is returned to you in the success response.
- **Azure Arc proxy:** A new component added to the Azure Arc agents. This component runs within the context of an Azure Arc-enabled resource as a service called Azure Arc proxy. It acts as a forward proxy that the Azure Arc agents and extensions use. No configuration is required on your part for this proxy.

When the gateway is in place, traffic flows via **Azure Arc agents** > **Azure Arc proxy** > **Enterprise proxy** > **Azure Arc gateway**  > **Target service**. For more information, see [Azure Arc gateway forwarding protocol](#azure-arc-gateway-forwarding-protocol).

:::image type="content" source="media/arc-gateway/arc-gateway-overview.png" alt-text="Diagram that shows the route of traffic flow for Azure Arc gateway." lightbox="media/arc-gateway/arc-gateway-overview.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

## Current limitations

Azure Arc gateway has the following current limitations. Consider these factors when you plan your configuration:

- Proxy bypass isn't supported when Azure Arc gateway is in use. Even if you attempt to use the feature by running `azcmagent config set proxy.bypass`, traffic can't bypass the proxy.
- Each Azure subscription has a limit of five Azure Arc gateway resources.
- Azure Arc gateway is used only for connectivity in the Azure public cloud platform.
- Azure Arc gateway isn't recommended for use in environments where Transport Layer Security (TLS) termination or inspection is required. If your environment requires TLS termination or inspection, we recommend that you skip TLS inspection for your Azure Arc gateway endpoint (`<Your URL prefix>.gw.arc.azure.com`). For more information, see [Azure Arc gateway and TLS inspection](#azure-arc-gateway-and-tls-inspection).

Although Azure Arc gateway provides the connectivity required to use Azure Arc-enabled servers, you might still need to manually allow more endpoints in your environment to use some extensions and services with your connected machines. For more information, see [More scenarios](#more-scenarios). Over time, Azure Arc gateway gradually covers more endpoints and further removes the need for these manual allowances.

## Plan your Azure Arc gateway setup

- **Region choice**: Azure Arc gateway is a global service. Runtime connectivity is delivered through the Azure Front Door global edge network, which automatically routes clients to the nearest point of presence for low‑latency access and seamless failover. The region that you select when you create the gateway determines only the control plane. It's where the gateway resource and management metadata live and where create, update, and delete happen. It doesn't constrain the gateway's runtime endpoints or performance. For example, picking East US versus West Europe doesn't change where clients connect at runtime. It affects only management-plane placement and policy or role-based access control locality.
- **Azure Arc-enabled resources per Azure Arc gateway resource**: When you plan your Azure Arc deployment with Azure Arc gateway, you must determine how many gateway resources are required for your environment. This amount depends on the number of resources that you plan to manage in each Azure region. For Azure Arc-enabled servers only, a general rule is that one Azure Arc gateway resource can handle 2,000 resources per Azure region. You might use Azure Arc gateway with a combination of Azure Arc-enabled servers, Azure Arc-enabled Kubernetes clusters, and Azure Local instances. [The formula we provide](#azure-arc-gateway-resource-planning-for-multiple-resource-types) can help you calculate the number of Azure Arc gateway resources that you need.

## Required permissions

To create Azure Arc gateway resources and manage their association with Azure Arc-enabled servers, a user must have the Azure Arc gateway manager role.

## Create an Azure Arc gateway resource

You can create an Azure Arc gateway resource by using the Azure portal, the Azure CLI, or Azure PowerShell. It generally takes about 10 minutes to create an Azure Arc gateway resource after you finish these steps.

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/).

1. Go to **Azure Arc**. On the service menu, under **Management**, select **Azure Arc gateway**, and then select **Create**.

1. Select the subscription and resource group where you want the Azure Arc gateway resource to be managed within Azure. Any Azure Arc-enabled resource in the same Azure tenant can use an Azure Arc gateway resource.

1. For **Name**, enter the name for the Azure Arc gateway resource.

1. For **Location**, enter the region where the Azure Arc gateway resource should live. Any Azure Arc-enabled resource in the same Azure tenant can use an Azure Arc gateway resource.

1. Select **Next**.

1. On the **Tags** page, optionally specify one or more custom tags to support your standards.

1. Select **Review + create**.

1. Review your input details, and then select **Create**.

### [CLI](#tab/cli)

1. Add the Azure Arc gateway extension to the Azure CLI:

    `az extension add -n arcgateway`

1. On a machine with access to Azure, run the following commands to create your Azure Arc gateway resource:

    ```azurecli
    az arcgateway create `
        --gateway-name [Your gateway’s Name] `
        --resource-group <Your Resource Group> `
        --location [Location]
    ```

### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command to create your Azure Arc gateway resource:

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

After you successfully create the resource, the success response includes the Azure Arc gateway URL. Ensure that your Azure Arc gateway URL and all of these URLs are allowed in the environment where your Azure Arc resources live.

> [!IMPORTANT]
> This list was recently updated. If you previously enabled access to these URLs, you might need to review the list and update your network configuration to ensure that each endpoint is allowed.

|URL  |Purpose  |
|---------|---------|
|`<Your URL prefix>.gw.arc.azure.com`  |Your gateway URL (obtained by running `az arcgateway list` after you create your gateway resource)  |
|`management.azure.com`  |Azure Resource Manager endpoint, required for Azure Resource Manager control channel  |
|`login.microsoftonline.com`, `<region>.login.microsoft.com`  |Microsoft Entra ID endpoint for acquiring identity access tokens  |
|`gbl.his.arc.azure.com`  |The cloud service endpoint for communicating with Azure Arc agents  |
|`<region>.his.arc.azure.com`  |Used for the Azure Arc core control channel  |
|`packages.microsoft.com`  |Required to connect Linux servers to Azure Arc  |
|`download.microsoft.com` |Used to download the Windows installation package |

## Onboard new Azure Arc resources with your Azure Arc gateway resource

1. Generate the installation script.

    Follow the instructions at [Quickstart: Connect hybrid machines with Azure Arc-enabled servers](quick-enable-hybrid-vm.md) to create a script that automates the downloading and installation of the Azure Connected Machine agent and establishes the connection with Azure Arc.

    > [!IMPORTANT]
    > When you generate the onboarding script, ensure that **Public Endpoint** is selected in the **Connectivity method** section. Also make sure that your Azure Arc gateway resource is selected in the **Gateway Resource** dropdown list.

1. Run the installation script to onboard your servers to Azure Arc.

    In the script, the Azure Arc gateway resource's Azure Resource Manager ID is shown as `--gateway-id`.

## Configure existing Azure Arc resources to use Azure Arc gateway

You can associate existing Azure Arc resources with an Azure Arc gateway resource by using the Azure portal, the Azure CLI, or Azure PowerShell.

### [Portal](#tab/portal)

1. In the Azure portal, go to **Azure Arc - Azure Arc gateway**.

1. Select the Azure Arc gateway resource to associate with your Azure Arc-enabled server.

1. On the service menu for your gateway resource, select **Associated resources**.

1. Select **Add**.

1. Select the Azure Arc-enabled server resource to associate with your Azure Arc gateway resource.

1. Select **Apply**.

### [CLI](#tab/cli)

On a machine with access to Azure, run the following commands:

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

On a machine with access to Azure, run the following commands:

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

With the Connected Machine agent version 1.50 or earlier, you must also run `azcmagent config set connection.type gateway` to update your Azure Arc-enabled server to use Azure Arc gateway. For agent versions 1.51 and later, this step isn't required because the operation happens automatically. We recommend that you use the [latest version of the Connected Machine agent](agent-release-notes.md).

## Verify successful Azure Arc gateway setup

On the onboarded server, run the command `azcmagent show`.

The result should indicate the following values:

- **Agent Status**: Shows as **Connected**.
- **Using HTTPS Proxy**: Shows as `http://localhost:40343`.
- **Upstream Proxy**: Shows as your enterprise proxy (if you set one). The Azure Arc gateway URL should reflect your gateway resource's URL.

To verify successful setup, run the command `azcmagent check`.

The result should indicate that `connection.type` is set to the gateway, and the **Reachable** column should indicate **true** for all URLs.

## Remove Azure Arc gateway association

You can disable Azure Arc gateway and remove the association between the Azure Arc gateway resource and the Azure Arc-enabled cluster. The Azure Arc-enabled cluster then uses direct traffic instead.

This operation applies to Azure Arc gateway on Azure Arc-enabled servers only, not Azure Local. If you use Azure Arc gateway on Azure Local, see [About Azure Arc gateway for Azure Local](/azure/azure-local/deploy/deployment-azure-arc-gateway-overview) for removal information.

1. Set the connection type of the Azure Arc-enabled server to `direct` instead of `gateway` by running the following command:

   `azcmagent config set connection.type direct`

   > [!NOTE]
   > If you take this step, all [Azure Arc network requirements](/azure/azure-arc/network-requirements-consolidated?tabs=azure-cloud) must be met in your environment to continue using Azure Arc.

1. Detach the Azure Arc gateway resource from the machine:

   ### [Portal](#tab/portal)

   1. In the Azure portal, go to **Azure Arc - Azure Arc gateway**.

   1. Select the Azure Arc gateway resource.

   1. On the service menu for your gateway resource, select **Associated resources**.

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

   If you're running this Azure CLI command within Windows PowerShell, set `--gateway-resource-id` to null.

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

### Delete an Azure Arc gateway resource

You can delete an Azure Arc gateway resource by using the Azure portal, the Azure CLI, or Azure PowerShell. This operation might take up to 5 minutes to complete.

### [Portal](#tab/portal)

1. In the Azure portal, go to **Azure Arc - Azure Arc gateway**.

1. Select the Azure Arc gateway resource.

1. Select **Delete**, and then confirm the deletion.

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

## Monitor Azure Arc gateway traffic

You can audit your Azure Arc gateway traffic by viewing the Azure Arc proxy logs.

To view Azure Arc proxy logs on Windows:

1. Run `azcmagent logs` in PowerShell.
1. In the resulting .zip file, the `arcproxy.log` file is located in the `ProgramData\AzureConnectedMachineAgent\Log` folder.

To view Azure Arc proxy logs on Linux:

1. Run `sudo azcmagent logs`.
1. In the resulting .zip file, the `arcproxy.log` file is located in the `/var/opt/azcmagent/log/` folder.

## Azure Arc gateway resource planning for multiple resource types

To determine how many gateway resources you need per Azure region for multiple resource types, use the following formula:

  **Score** = (Servers ÷ 20) + (Kubernetes clusters ÷ 10) + (Azure Local instances ÷ 10)

  Where:

  - **Servers** = total standalone servers + provisioned virtual machines (on Azure Local)
  - **Kubernetes clusters** = total standalone Kubernetes clusters + Azure Kubernetes Service Azure Arc clusters (on Azure Local)
  - **Azure Local instances** = total Azure Local deployments

  If the score for each region from which you intend to manage your resources <100, one Azure Arc gateway resource is sufficient.
  
  If the score for any region from which you intend to manage your resources ≥100, more than one Azure Arc gateway resource is required for that region.

 The following examples provide more context.

#### Example 1

  | Region        | Servers | Kubernetes clusters | Azure Local instances | Score calculation                          | Score |
  |---------------|---------|--------------|-----------------------|--------------------------------------------|-------|
  | East US       | 300     | 20           | 5                     | 300/20 + 20/10 + 5/10                      | 17.5  |
  | West Europe   | 800     | 50           | 10                    | 800/20 + 50/10 + 10/10                     | 46.0  |
  | Japan East    | 100     | 5            | 2                     | 100/20 + 5/10 + 2/10                       | 5.7   |

 Each region's score is <100. One Azure Arc gateway resource is sufficient.

#### Example 2

  | Region          | Servers | Kubernetes clusters | Azure Local instances | Score calculation                          | Score |
  |-----------------|---------|--------------|-----------------------|--------------------------------------------|-------|
  | East US         | 6,000   | 300          | 40                    | 6000/20 + 300/10 + 40/10                   | 334.0 |
  | West Europe     | 2,500   | 120          | 25                    | 2500/20 + 120/10 + 25/10                   | 139.5 |  
  | Southeast Asia  | 900     | 30           | 8                     | 900/20 + 30/10 + 8/10                      | 48.8  |
  
  - The East US score is >100. Three Azure Arc gateway resources are needed to support the load in this region.
  - The West Europe score is >100. Two Azure Arc gateway resources are needed to support the load in this region.  
  - The Southeast Asia score is <100. One Azure Arc gateway resource is needed to support the load in this region.

In this scenario, only three gateway resources are required in total because calculations are based on the maximum load per region, not the combined load across all regions.

## More scenarios

Azure Arc gateway covers the endpoints that are required for onboarding a server, plus endpoints to support several more Azure Arc-enabled scenarios. Based on the scenarios that you adopt, you might need to allow more endpoints in your environment.

### Scenarios that don't require more endpoints

- Windows Admin Center
- Secure Shell
- Extended Security Updates
- Azure Extension for SQL Server

### Scenarios that require more endpoints

Endpoints listed with the following scenarios must be allowed in your enterprise proxy when you use Azure Arc gateway:

- Azure Arc-enabled data services:

  - `*.ods.opinsights.azure.com`
  - `*.oms.opinsights.azure.com`
  - `*.monitoring.azure.com`

- Azure Monitor Agent:

  - `<log-analytics-workspace-id>.ods.opinsights.azure.com`
  - `<data-collection-endpoint>.<virtual-machine-region>.ingest.monitor.azure.com`

- Azure Key Vault certificate sync:

  - `<vault-name>.vault.azure.net`

- Azure Automation Hybrid Runbook Worker extension:

  - `*.azure-automation.net`

- Windows OS Update Extension/Azure Update Manager:

  - Your environment must meet all the [prerequisites](/windows/privacy/manage-windows-11-endpoints) for Windows Update.

- Microsoft Defender:

  - Your environment must meet all the [prerequisites](/defender-endpoint/configure-device-connectivity) for Microsoft Defender.

## Azure Arc gateway architecture

Review the following information to understand more about the architecture of Azure Arc gateway.

### Azure Arc gateway forwarding protocol

:::image type="content" source="media/arc-gateway/arc-gateway-architecture.png" lightbox="media/arc-gateway/arc-gateway-architecture.png" alt-text="Diagram that shows the architecture for Azure Arc gateway with Azure Arc-enabled servers.":::

### Azure Arc gateway and TLS inspection

Azure Arc gateway works by establishing a TLS session between Azure Arc proxy and Azure Arc gateway in Azure. Within this TLS session, Azure Arc proxy sends a nested HTTP connect request to Azure Arc gateway resource. The connect request instructs the resource to forward the connection to the intended target destination. Then, if the target destination itself is on TLS, an inner end-to-end TLS session is established between the Azure Arc agent and the target destination.

When you use terminating proxies with Azure Arc gateway, the proxy sees the nested HTTP connect request. It might allow such a request, but it can't intercept TLS encrypted traffic to the target destination unless it does nested TLS termination. This behavior is outside the capabilities of standard TLS terminating proxies. When you use a terminating proxy, we recommend that you skip TLS inspection for your Azure Arc gateway endpoint.

### Endpoints accessible through Azure Arc gateway

Arc Gateway uses a set of endpoints to enable all Arc features to function seamlessly. Currently, this includes over 200 endpoints, which represent the cumulative requirements for all supported capabilities. For the full list, see [Azure Arc gateway endpoints](arc-gateway-endpoints.md).

Some endpoints are wildcarded to simplify connectivity and ensure feature coverage. We recommend reviewing these with your network security team to confirm they align with your organization’s policies. These endpoints are essential for secure and reliable operation of Arc services.
