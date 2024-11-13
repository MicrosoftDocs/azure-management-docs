---
title: How to simplify network configuration requirements with Azure Arc gateway (Public Preview)
description: Learn how to simplify network configuration requirements with Azure Arc gateway (Public Preview).
ms.date: 11/11/2024
ms.topic: how-to
---

# Simplify network configuration requirements with Azure Arc gateway (Public Preview)

If you use enterprise proxies to manage outbound traffic, the Azure Arc gateway lets you onboard infrastructure to Azure Arc using only seven (7) endpoints. With Azure Arc gateway, you can:

- Connect to Azure Arc by opening public network access to only seven (7) Fully Qualified Domains (FQDNs).
- View and audit all traffic an Azure Connected Machine agent sends to Azure via the Arc gateway.

This article explains how to set up and use Arc gateway (Public Preview).

> [!IMPORTANT]
> The Arc gateway feature for Azure Arc-enabled servers is currently in Public Preview in all regions where Azure Arc-enabled servers is present. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, Public Preview, or otherwise not yet released into general availability
>  

## How the Azure Arc gateway works

Azure Arc gateway consists of two main components:

- **The Arc gateway resource:** An Azure resource that serves as a common front-end for Azure traffic. This gateway resource is served on a specific domain. Once the Arc gateway resource is created, the domain is returned to you in the success response.

- **The Arc Proxy:** A new component added to Arc agentry. This component runs as a service called "Azure Arc Proxy" and acts as a forward proxy used by the Azure Arc agents and extensions. No configuration is required on your part for the Arc Proxy. This Proxy is part of Arc core agentry and runs within the context of an Arc-enabled resource.

When the gateway is in place, traffic flows via the following hops: **Arc agentry → Arc Proxy → Enterprise proxy → Arc gateway  → Target service**

:::image type="content" source="media/arc-gateway/arc-gateway-overview.png" alt-text="Diagram showing the route of traffic flow for Azure Arc gateway." lightbox="media/arc-gateway/arc-gateway-overview.png":::

## Current limitations

The Arc gateway object has limits you should consider when planning your setup. These limitations apply only to the public preview. These limitations might not apply when the Arc gateway feature is generally available.

- TLS Terminating Proxies aren't supported (Public Preview)
- ExpressRoute/Site-to-Site VPN or private endpoints used with the Arc gateway (Public Preview) isn't supported.
- There's a limit of five (5) Arc gateway (Public Preview) resources per Azure subscription.

## Required permissions

To create Arc gateway resources and manage their association with Arc-enabled servers, the following permissions are required:

- Microsoft.HybridCompute/settings/write
- Microsoft.hybridcompute/gateways/read
- Microsoft.hybridcompute/gateways/write

## How to use the Arc gateway (Public Preview)

There are four steps to use the Arc gateway:

1. Create an Arc gateway resource.
1. Ensure the required URLs are allowed in your environment.
1. Onboard Azure Arc resources with your Arc gateway resource or configure existing Azure Arc resources to use Arc gateway.
1. Verify that the setup succeeded.

### Step 1: Create an Arc gateway resource

You can create an Arc gateway resource using the Azure portal, Azure CLI, or Azure PowerShell.

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to the **Azure Arc | Azure Arc gateway** page, and then select **Create**.

1. Select the subscription and resource group where you want the Arc gateway resource to be managed within Azure. An Arc gateway resource can be used by any Arc-enabled resource in the same Azure tenant.

1. For **Name**, input the name that for the Arc gateway resource. 

1. For **Location**, input the region where the Arc gateway resource should live. An Arc gateway resource can be used by any Arc-enabled Resource in the same Azure tenant.  

1. Select **Next**.

1. On the **Tags** page, specify one or more custom tags to support your standards.  

1. Select **Review & Create**.

1. Review your input details, and then select **Create**.
    
    The gateway creation process takes 9-10 minutes to complete.


### [CLI](#tab/cli)

1. Add the arc gateway extension to your Azure CLI:

    `az extension add -n arcgateway`

1. On a machine with access to Azure, run the following commands to create your Arc gateway resource:

    ```azurecli
    az arcgateway create --name [Your gateway’s Name] --resource-group [Your Resource Group] --location [Location]
    ```
    The gateway creation process takes 9-10 minutes to complete.

### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command to create your Arc gateway resource:
    
```powershell
New-AzArcgateway 
-name <gateway’s name> 
-resource-group <resource group> 
-location <region> 
-subscription <subscription name or id> 
-gateway-type public  
-allowed-features *
```
The gateway creation process takes 9-10 minutes to complete.

---

### Step 2: Ensure the required URLs are allowed in your environment

When the resource is created, the success response includes the Arc gateway URL. Ensure your Arc gateway URL and all URLs in the following table are allowed in the environment where your Arc resources live. The required URLs are:

|URL  |Purpose  |
|---------|---------|
|[Your URL Prefix].gw.arc.azure.com  |Your gateway URL (This URL can be obtained by running `az arcgateway list` after you create your gateway Resource)  |
|management.azure.com  |Azure Resource Manager Endpoint, required for Azure Resource Manager control channel  |
|login.microsoftonline.com  |Microsoft Entra ID’s endpoint, for acquiring Identity access tokens  |
|gbl.his.arc.azure.com  |The cloud service endpoint for communicating with Azure Arc agents  |
|\<region\>.his.arc.azure.com  |Used for Arc’s core control channel  |
|packages.microsoft.com  |Required to acquire Linux based Arc agentry payload, only needed to connect Linux servers to Arc  |

### Step 3a: Onboard Azure Arc resources with your Arc gateway resource.

1. Generate the installation script.
 
    Follow the instructions at [Quickstart: Connect hybrid machines with Azure Arc-enabled servers](learn/quick-enable-hybrid-vm.md) to create a script that automates the downloading and installation of the Azure Connected Machine agent and establishes the connection with Azure Arc.
     
    > [!IMPORTANT]
    >   When generating the onboarding script, select **Proxy Server** under **Connectivity method** to reveal the dropdown for **Gateway resource**.
    >       

1. Run the installation script to onboard your servers to Azure Arc.
    
    In the script, the Arc gateway resource's ARM ID is shown as `--gateway-id`.

### Step 3b: Configure existing Azure Arc resources to use Arc gateway

You can configure existing Azure Arc resources to use Arc gateway by using the Azure portal, Azure CLI, or Azure PowerShell.

### [Portal](#tab/portal)

1. On the Azure portal, go to the **Azure Arc - Azure Arc gateway** page.

1. Select the Arc gateway Resource to associate with your Arc-enabled server.

1. Go to the Associated Resources page for your gateway resource.

1. Select **Add**.

1. Select the Arc-enabled resource to associate with your Arc gateway resource. 

1. Select **Apply**.

1. Update your Arc-enabled server to use Arc gateway by running `azcmagent config set connection.type gateway`.

### [CLI](#tab/cli)

1. On a machine with access to Azure, run the following commands:

     ```azurecli
    az arcgateway settings update
        --resource-group [Resource Group]
        --subscription [subscription name]
        --base-provider Microsoft.HybridCompute
        --base-resource-type machines
        --base-resource-name [Arc-Server’s name]
        --settings-resource-name default--gateway-resource-id [Full Arm resource id of the new Arc gateway resource]
    ```
1. Update your Arc-enabled server to use Arc gateway by running the following command:

    `azcmagent config set connection.type gateway`

### [PowerShell](#tab/powershell)

1. On a machine with access to Azure, run the following commands:
    
    ```powershell
    Update-AzArcSetting  
        -ResourceGroup <res-group>  
        -Subscription <subscription name>  
        -BaseProvider <RP Name>  
        -BaseResourceType <Resource Type>  
        -Name <Arc-server's resource name>  
        -SettingsResourceName "default"  
        -GatewayResourceId <Full Arm resourceid>
    ```
    
1. Update your Arc-enabled server to use Arc gateway by running the following command:

    `azcmagent config set connection.type gateway`

---

### Step 4: Verify that the setup succeeded
On the onboarded server, run the following command: `azcmagent show`
The result should indicate the following values:

- **Agent Status** should show as **Connected**.
- **Using HTTPS Proxy** should show as **http://localhost:40343**.
- **Upstream Proxy** should show as your enterprise proxy (if you set one). Gateway URL should reflect your gateway resource's URL.

Additionally, to verify successful set-up, you can run the following command: `azcmagent check`
The result should indicate that the `connection.type` is set to gateway, and the **Reachable** column should indicate **true** for all URLs.


## Associate a machine with a new Arc gateway

To associate a machine with a new Arc gateway:

### [Portal](#tab/portal)

1. On the Azure portal, go to the **Azure Arc - Azure Arc gateway** page.

1. Select the new Arc gateway Resource to associate with the machine.

1. Go to the Associated Resources page for your gateway resource.

1. Select **Add**.

1. Select the Arc-enabled machine to associate with the new Arc gateway resource. 

1. Select **Apply**.

1. Update your Arc-enabled server to use Arc gateway by running `azcmagent config set connection.type gateway`.

### [CLI](#tab/cli)

1. On the machine you want to associate with a new Arc gateway, run the following commands:

     ```azurecli
    az arcgateway settings update
        --resource-group [Resource Group]
        --subscription [subscription name]
        --base-provider Microsoft.HybridCompute
        --base-resource-type machines
        --base-resource-name [Arc-Server’s name]
        --settings-resource-name default--gateway-resource-id [Full Arm resource id of the new Arc gateway resource]
    ```
1. Update your Arc-enabled server to use Arc gateway by running the following command:

    `azcmagent config set connection.type gateway`

### [PowerShell](#tab/powershell)

1. On the machine you want to associate with a new Arc gateway, run the following command:
    
    ```powershell
    Set-AzArcGatewaySettings  
        -ResourceGroup <res-group>  
        -Subscription <subscription name>  
        -BaseProvider <RP Name>  
        -BaseResourceType <Resource Type>  
        -Name <Arc-server's resource name>  
        -SettingsResourceName "default"  
        -GatewayResourceId <Full Arm resourceid>
    ```
    
1. Update your Arc-enabled server to use Arc gateway by running the following command:

    `azcmagent config set connection.type gateway`

---

## Remove Arc gateway association (to use the direct route instead)

1. Set the connection type of the Arc-enabled Server to "direct” instead of “gateway" by running the following command:

    `azcmagent config set connection.type direct`

    > [!NOTE]
    > If you take this step, all [Azure Arc network requirements](/azure/azure-arc/network-requirements-consolidated?tabs=azure-cloud) must be met in your environment to continue leveraging Azure Arc.  
    > 

1. Detach the Arc gateway resource from the machine:

    ### [Portal](#tab/portal)
    
    1. On the Azure portal, go to the **Azure Arc - Azure Arc gateway** page.
    
    1. Select the Arc gateway Resource.
    
    1. Go to the **Associated Resources** page for your gateway resource and select the server.
    
    1. Select **Remove**.
    
    ### [CLI](#tab/cli)
    
    On a machine with access to Azure, run the following Azure CLI command:
    
    ```azurecli
    az arcgateway settings update --resource-group <Resource Group> --subscription <subscription name> --base-provider Microsoft.HybridCompute --base-resource-type machines --base-resource-name <Arc-Server’s name>  --gateway-resource-id ""
    ```
    
    > [!NOTE]
    > If you’re running this Azure CLI command within Windows PowerShell, set the `--gateway-resource-id` to null.
    > 
    
    ### [PowerShell](#tab/powershell)
    
    On a machine with access to Azure, run the following PowerShell command:
    
    ```powershell
    Set-AzArcGatewaySettings  
        -ResourceGroup <res-group>  
        -Subscription <subscription name>  
        -BaseProvider <RP Name>  
        -BaseResourceType <Resource Type>  
        -Name <Arc-server's resource name>  
        -SettingsResourceName "default"  
        -GatewayResourceId ""
    ```
    
    ---
    
### Delete an Arc gateway resource

> [!NOTE]
> This operation can take 4 to 5 minutes to complete.
> 

### [Portal](#tab/portal)

1. On the Azure portal, go to the **Azure Arc - Azure Arc gateway** page.

1. Select the Arc gateway Resource.

1. Select **Delete**.


### [CLI](#tab/cli)

On a machine with access to Azure, run the following Azure CLI command:

```azurecli-interactive
az arcgateway delete --resource-group <Resource Group> --gateway-name <Arc gateway Resource Name >
```

### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command:


```powershell
Remove-AzArcGateway  
    -ResourceGroup <res-group>  
    -GatewayName <RP Name>
```


---

## Troubleshooting

You can audit your Arc gateway’s traffic by viewing the Azure Arc proxy logs.

To view Arc proxy logs on **Windows**:
1. Run `azcmagent logs` in PowerShell.
1. In the resulting .zip file, the logs are located in the `C:\ProgramData\Microsoft\ArcProxy` folder.

To view Arc proxy logs on **Linux**:
1. Run `sudo azcmagent logs`and share the resulting file.
1. In the resulting log file, the logs are located in the `/usr/local/arcproxy/logs/` folder.

## Additional scenarios

During Public Preview, Arc gateway covers the endpoints required for onboarding a server, as well as a portion of endpoints required for additional Arc-enabled scenarios. Based on the scenario(s) you adopt, additional endpoints must be allowed in your proxy.

### Scenarios that don’t require additional endpoints

- Windows Admin Center 
- SSH 
- Extended Security Updates 
- Microsoft Defender  
- Azure Extension for SQL Server 

### Scenarios that require additional endpoints

Endpoints listed with the following scenarios must be allowed in your enterprise proxy when using Arc gateway:

- Azure Arc-enabled Data Services 

    - *.ods.opinsights.azure.com 
    
    - *.oms.opinsights.azure.com 
    
    - *.monitoring.azure.com 
    
- Azure Monitor Agent 

    - \<log-analytics-workspace-id\>.ods.opinsights.azure.com 
    
    - \<data-collection-endpoint\>.\<virtual-machine-region-name\>.ingest.monitor.azure.com 

- Azure Key Vault Certificate Sync 

    - \<vault-name\>.vault.azure.net

- Azure Automation Hybrid Runbook Worker extension 

    - *.azure-automation.net

- Windows OS Update Extension / Azure Update Manager 

    - Your environment must meet all the [prerequisites](/windows/privacy/manage-windows-11-endpoints) for Windows Update 

## Known issues

Following is a description of currently known issues for the Arc gateway.

### Refresh needed after Azure Connected Machine agent onboarding

When using the onboarding script (or the `azcmagent connect` command) to onboard a server with the gateway resource ID specified, the resource will successfully use Arc gateway. However, due to a known bug (with a fix currently underway), the Arc-enabled server won't display as an Associated Resource in Azure portal unless the resource’s settings are refreshed. Use the following procedure to perform this refresh:

   
### [Portal](#tab/portal)

1. In the Azure portal, navigate to the **Azure Arc | Arc gateway** page.

1. Select the Arc gateway resource to associate with your Arc-enabled server.

1. Navigate to the **Associated Resources** page for your gateway resource.

1. Select **Add**. 

1. Select the Arc-enabled resource to associate with your Arc gateway resource and select **Apply**.

### [CLI](#tab/cli)

On a machine with access to Azure, run the following Azure CLI command:

```azurecli
az arcgateway settings update --resource-group <Resource Group> --subscription <subscription name> --base-provider Microsoft.HybridCompute --base-resource-type machines --base-resource-name <Arc-Server’s name>  --gateway-resource-id <Full Arm resource id of the new Arc gateway resource>
```

### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command:

```powershell
Set-AzArcGatewaySettings  
    -ResourceGroup <res-group>  
    -Subscription <subscription name>  
    -BaseProvider <RP Name>  
    -BaseResourceType <Resource Type>  
    -Name <Arc-server's resource name>  
    -SettingsResourceName "default"  
    -GatewayResourceId <Full Arm resourceid>
```
---

### Arc proxy refresh needed after detaching a gateway resource from the machine

When detaching an Arc gateway resource from a machine, you must refresh the Arc proxy to clear the Arc gateway configuration. To do so, perform the following procedure:

1. Stop arc proxy. 

    - Windows: `Stop-Service arcproxy` 
    - Linux: `sudo systemctl stop arcproxyd` 

1. Delete the `cloudconfig.json` file.

    - Windows: "C:\ProgramData\AzureConnectedMachineAgent\Config\cloudconfig.json" 
    - Linux: "/var/opt/azcmagent/cloudconfig.json" 

1. Start arc proxy.

    - Windows: `Start-Service arcproxy` 
    - Linux: `sudo systemctl start arcproxyd` 

1. Restart himds (optional, but recommended).

    - Windows: `Restart-Service himds` 
    - Linux: `sudo systemctl restart himdsd` 


### Refresh needed for machines re-enabled without gateway

If an Arc-enabled machine with an Arc gateway is deleted from Azure Arc and re-Arc-enabled without an Arc gateway, a refresh is needed to update its status in the Azure portal.

> [!IMPORTANT]
> This issue occurs only when the resource is re-Arc-enabled with the same ARM ID as its initial enablement.
> 

In this scenario, the machine incorrectly displays in Azure portal as a resource associated with the Arc gateway. To prevent this, if you intend to Arc-enable a machine without an Arc gateway that was previously Arc-enabled with an Arc gateway, you must update the Arc gateway association after onboarding. To do so, use the following procedure:

### [Portal](#tab/portal)

1. In the Azure portal, navigate to the **Azure Arc | Arc gateway** page.

1. Select the Arc gateway resource.

1. Navigate to the **Associated Resources** page for your gateway resource.

1. Select the server, and then select **Remove**.

### [CLI](#tab/cli)

On a machine with access to Azure, run the following Azure CLI command:

```azurecli
az arcgateway settings update --resource-group <Resource Group> --subscription <subscription name> --base-provider Microsoft.HybridCompute --base-resource-type machines --base-resource-name <Arc-Server’s name>  --gateway-resource-id ""
```

> [!NOTE]
> If you’re running this Azure CLI command within Windows PowerShell, set the `--gateway-resource-id` to null.
> 
### [PowerShell](#tab/powershell)

On a machine with access to Azure, run the following PowerShell command:

```powershell
Set-AzArcGatewaySettings  

    -ResourceGroup <res-group>  

    -Subscription <subscription name>  

    -BaseProvider <RP Name>  

    -BaseResourceType <Resource Type>  

    -Name <Arc-server's resource name>  

    -SettingsResourceName "default"  

    -GatewayResourceId ""
```

---

### Manual gateway association required post-deletion

If an Arc gateway is deleted while a machine is still connected to it, Azure portal must be used to associate the machine with any other Arc gateway resources.

To avoid this issue, detach all Arc-enabled resources from an Arc gateway before deleting the gateway resource. If you encounter this error, use Azure portal to associate the machine with a new Arc gateway resource.
