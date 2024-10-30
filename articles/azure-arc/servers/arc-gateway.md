---
title: How to simplify network configuration requirements through Azure Arc gateway (Public Preview)
description: Learn how to simplify network configuration requirements through Azure Arc gateway (Public Preview).
ms.date: 10/30/2024
ms.topic: how-to
---

# Simplify network configuration requirements through Azure Arc gateway (Public Preview)

If you use enterprise firewalls or proxies to manage outbound traffic, the Azure Arc gateway lets you onboard infrastructure to Azure Arc using only seven (7) endpoints. With Azure Arc gateway, you can:

- Connect to Azure Arc by opening public network access to only seven (7) Fully Qualified Domains (FQDNs).
- View and audit all traffic an Azure Connected Machine agent sends to Azure via the Arc gateway.

This article explains how to set up and use Arc gateway (Public Preview).

> [!IMPORTANT]
> The Arc gateway feature for Azure Arc-enabled servers is currently in Public Preview in all regions where Azure Arc-enabled servers is present. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, Public Preview, or otherwise not yet released into general availability
>  

## How it works

Azure Arc gateway consists of two main components:

- **The Arc gateway resource:** An Azure resource that serves as a common front-end for Azure traffic. This gateway resource is served on a specific domain. Once the Arc gateway resource is created, the domain is returned to you in the success response.

- **The Arc Proxy:** A new component added to Arc agentry. This component runs as a service called "Azure Arc Proxy" and acts as a forward proxy used by the Azure Arc agents and extensions. No configuration is required on your part for the gateway router. This router is part of Arc core agentry and runs within the context of an Arc-enabled resource.

When the gateway is in place, traffic flows via the following hops: **Arc agentry → Arc Proxy → Enterprise proxy → Arc gateway  → Target service**

:::image type="content" source="media/arc-gateway/arc-gateway-overview.png" alt-text="Diagram showing the route of traffic flow for Azure Arc gateway.":::

## Restrictions and limitations

The Arc gateway object has limits you should consider when planning your setup. These limitations apply only to the public preview. These limitations might not apply when the Arc gateway feature is generally available.

- TLS Terminating Proxies aren't supported (Public Preview)
- ExpressRoute/Site-to-Site VPN or private endpoints used with the Arc gateway (Public Preview) isn't supported.
- There's a limit of five (5) Arc gateway (Public Preview) resources per Azure subscription.

## How to use the Arc gateway (Public Preview)

There are five steps to use the feature:

1. Create an Arc gateway resource.
1. Ensure the required URLs are allowed in your environment.
1. Onboard Azure Arc resources with your Arc gateway resource.
1. Configure existing Azure Arc resources to use Arc gateway.
1. Verify that the setup succeeded.

### Step 1: Create an Arc gateway resource

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/).

1. On the **Azure Arc - Azure Arc gateway** page, select **Create**.

1. Select the subscription and resource group where you want the Arc gateway resource to be managed within Azure. An Arc gateway resource can be used by any Arc-enabled resource in the same Azure tenant.

1. For **Name**, input the name that for the Arc gateway resource. 

1. For **Location**, input the region where the Arc gateway resource should live. An Arc gateway resource can be used by any Arc-enabled Resource in the same Azure tenant.  

1. Select **Next**.

1. On the **Tags** page, specify one or more custom tags to support your standards.  

1. Select **Review & Create**.

1. Review your input details, and then select **Create**.

1. The gateway creation process takes 9-10 minutes to complete.


### [CLI](#tab/cli)

1. Add the arc gateway extension to your Azure CLI:

    `az extension add -n arcgateway`

1. On a machine with access to Azure, run the following commands to create your Arc gateway resource:

    ```azurecli
    az login --use-device-code
    az account set --subscription [subscription name or id]
    az connectedmachine gateway create --name [Your gateway’s Name] --resource-group [Your Resource Group] --location [Location] --gateway-type public --allowed-features * --subscription [subscription name or id]
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

When the resource is created, the success response includes the Arc gateway URL. Ensure your Arc gateway URL and all URLs in the following table are allowed in the environment where your Arc resources live:

|URL  |Purpose  |
|---------|---------|
|[Your URL Prefix].gw.arc.azure.com  |Your gateway URL (This URL can be obtained by running `az connectedmachine gateway list` after you create your gateway Resource)  |
|management.azure.com  |Azure Resource Manager Endpoint, required for Azure Resource Manager control channel  |
|login.microsoftonline.com  |Microsoft Entra ID’s endpoint, for acquiring Identity access tokens  |
|gbl.his.arc.azure.com  |The cloud service endpoint for communicating with Azure Arc agents  |
|\<region\>.his.arc.azure.com  |Used for Arc’s core control channel  |
|packages.microsoft.com  |Required to acquire Linux based Arc agentry payload, only needed to connect Linux servers to Arc  |
|download.microsoft.com   |Used to download the Windows installation package  |

### Step 3: Onboard Azure Arc resources with your Arc gateway resource.

1. Generate the installation script.
 
    Follow the instructions at [Quickstart: Connect hybrid machines with Azure Arc-enabled servers](learn/quick-enable-hybrid-vm.md) to create a script that automates the downloading and installation of the Azure Connected Machine agent and establishes the connection with Azure Arc.
     
    > [!IMPORTANT]
    >   When generating the onboarding script, select **Proxy Server** under **Connectivity method** to reveal the dropdown for **Gateway resource**.
    >       

1. Run the installation script to onboard your servers to Azure Arc.
    
    In the script, the Arc gateway resource's ARM ID is shown as `--gateway-id`.

### Step 4: Configure existing Azure Arc resources to use Arc gateway

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

### Step 5: Verify that the setup succeeded
On the onboarded server, run the following command: `azcmagent show`
The result should indicate the following values:

- **Agent Status** should show as **Connected**.
- **Using HTTPS Proxy** should show as **http://localhost:40343**
- **Upstream Proxy** should show as your enterprise proxy (if you set one)

Additionally, to verify successful set-up, you can run the following command: `azcmagent check`
The result should indicate that the `connection.type` is set to gateway, and the **Reachable** column should indicate **true** for all URLs.

## Cleanup instructions

To clean up your gateway, detach the gateway resource from the applicable server(s); the resource can then be deleted safely:

1. Set the connection type of the Azure Arc-enabled server to "direct" instead of "gateway":  

    `azcmagent config set connection.type direct` 

1. Run the following command to delete the resource: 

    `az connectedmachine gateway delete   --resource group [resource group name] --gateway-name [gateway resource name]` 

    This operation may take a few minutes.  

## Troubleshooting

You can audit your Arc gateway’s traffic by viewing the Azure Arc proxy logs.

To view proxy logs on **Windows**:
1. Run `azcmagent logs` in PowerShell.
1. In the resulting .zip file, the logs are located in the `C:\ProgramData\Microsoft\ArcProxy` folder.

To view proxy logs on **Linux**:
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

### Scenarios that do require additional endpoints

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
