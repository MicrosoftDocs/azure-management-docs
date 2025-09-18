---
title: How to simplify network configuration requirements with Azure Arc gateway (Public Preview)
description: Learn how to simplify network configuration requirements with Azure Arc gateway (Public Preview).
ms.date: 08/20/2025
ms.topic: how-to
# Customer intent: "As an IT administrator managing hybrid infrastructure, I want to simplify network configuration with Azure Arc gateway, so that I can efficiently onboard and control Arc-enabled servers through minimal endpoint access."
---

# Simplify network configuration requirements with Azure Arc gateway (preview)

If you use enterprise proxies to manage outbound traffic, the Azure Arc gateway lets you onboard infrastructure to Azure Arc using only seven endpoints. With Azure Arc gateway, you can:

- Connect to Azure Arc by opening public network access to only seven fully qualified domain names (FQDNs).
- View and audit all traffic an Azure Connected Machine agent sends to Azure via the Arc gateway.

This article explains how to set up and use Arc gateway (preview).

> [!IMPORTANT]
> The Arc gateway feature for Azure Arc-enabled servers is currently in Public Preview in all regions where Azure Arc-enabled servers is present. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, Public Preview, or otherwise not yet released into general availability.

## How the Azure Arc gateway works

Azure Arc gateway consists of two main components:

- **Arc gateway resource:** An Azure resource that serves as a common front-end for Azure traffic. This gateway resource is served on a specific domain. Once the Arc gateway resource is created, the domain is returned to you in the success response.

- **Arc proxy:** A new component added to the Azure Arc agents. This component runs within the context of an Arc-enabled resource as a service called "Azure Arc Proxy". It acts as a forward proxy used by the Azure Arc agents and extensions. No configuration is required on your part for this proxy.

When the gateway is in place, traffic flows via the following hops: **Arc agents → Arc proxy → Enterprise proxy → Arc gateway  → Target service**.

:::image type="content" source="media/arc-gateway/arc-gateway-overview.png" alt-text="Diagram showing the route of traffic flow for Azure Arc gateway." lightbox="media/arc-gateway/arc-gateway-overview.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

## Current limitations

During the public preview, the following limitations apply. Consider these factors when planning your configuration.

- TLS Terminating Proxies aren't supported.
- Proxy bypass isn't supported when Arc gateway is in use; even if you attempt to use the feature by running `azcmagent config set proxy.bypass`, traffic won't bypass the proxy.
- There's a limit of five (5) Arc gateway resources per Azure subscription.
- Arc gateway can only be used for connectivity in the Azure public cloud.

> [!IMPORTANT]
> While Azure Arc gateway provides the connectivity required to use Azure Arc-enabled servers, you may need to enable additional endpoints in order to use some extensions and services with your connected machines. For details, see [Additional scenarios](#additional-scenarios).

## Required permissions

To create Arc gateway resources and manage their association with Arc-enabled servers, the following permissions are required:

- Microsoft.HybridCompute/settings/write
- Microsoft.hybridcompute/gateways/read
- Microsoft.hybridcompute/gateways/write

## Create the Arc gateway resource

You can create an Arc gateway (preview) resource by using the Azure portal, Azure CLI, or Azure PowerShell. It generally takes about 10 minutes to create the Arc gateway resource after you complete these steps.

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Azure Arc**. In the service menu, under **Management**, select **Azure Arc gateway (preview)**, then select **Create**.

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
|`download.microsoft.com` |Used to download the Windows installation package |

## Onboard new Azure Arc resources with your Arc gateway resource

1. Generate the installation script.

    Follow the instructions at [Quickstart: Connect hybrid machines with Azure Arc-enabled servers](quick-enable-hybrid-vm.md) to create a script that automates the downloading and installation of the Azure Connected Machine agent and establishes the connection with Azure Arc.

    > [!IMPORTANT]
    > When generating the onboarding script, select the **Gateway resource** in the **Connectivity method** section.

1. Run the installation script to onboard your servers to Azure Arc.

    In the script, the Arc gateway resource's ARM ID is shown as `--gateway-id`.

## Configure existing Azure Arc resources to use Arc gateway

You can associate existing Azure Arc resources with an Arc gateway resource by using the Azure portal, Azure CLI, or Azure PowerShell.

### [Portal](#tab/portal)

1. In the Azure portal, go to **Azure Arc - Azure Arc gateway (preview)**.

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

   1. In the Azure portal, go to **Azure Arc - Azure Arc gateway (preview)**.

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

## Monitor traffic

You can audit your Arc gateway’s traffic by viewing the Azure Arc proxy logs.

To view Arc proxy logs on Windows:

1. Run `azcmagent logs` in PowerShell.
1. In the resulting .zip file, the `arcproxy.log` file is located in the `ProgramData\AzureConnectedMachineAgent\Log` folder.

To view Arc proxy logs on Linux:

1. Run `sudo azcmagent logs`.
1. In the resulting .zip file, the `arcproxy.log` file is located in the `/var/opt/azcmagent/log/` folder.

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

  - `\<log-analytics-workspace-id\>.ods.opinsights.azure.com`
  - `\<data-collection-endpoint\>.\<virtual-machine-region-name\>.ingest.monitor.azure.com`

- Azure Key Vault Certificate Sync

  - `\<vault-name\>.vault.azure.net`

- Azure Automation Hybrid Runbook Worker extension

  - `*.azure-automation.net`

- Windows OS Update Extension / Azure Update Manager

  - Your environment must meet all the [prerequisites](/windows/privacy/manage-windows-11-endpoints) for Windows Update

- Microsoft Defender

  - Your environment must meet all the [prerequisites](/defender-endpoint/configure-device-connectivity) for Microsoft Defender
