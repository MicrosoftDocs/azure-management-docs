---
title: "Simplify network configuration requirements with Azure Arc gateway (preview)"
ms.custom: devx-track-azurecli
ms.date: 10/20/2024
ms.topic: how-to
description: "The Azure Arc gateway (preview) lets you onboard Kubernetes clusters to Azure Arc, requiring access to only seven endpoints."
---

# Simplify network configuration requirements with Azure Arc Gateway (preview)

If you use enterprise proxies to manage outbound traffic, the Azure Arc gateway (preview) can help simplify the process of enabling connectivity.

The Azure Arc gateway (preview) lets you:

- Connect to Azure Arc by opening public network access to only seven fully qualified domain names (FQDNs).
- View and audit all traffic that the Arc agents sends to Azure via the Arc gateway.

> [!IMPORTANT]
> Azure Arc gateway is currently in preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How the Azure Arc gateway works

The Arc gateway works by introducing two new components

The **Arc gateway resource** is an Azure Resource that serves as a common front end for Azure traffic. The gateway resource is served on a specific domain/URL. You must create this resource by following the steps outlined in this article. After you successfully create the gateway resource, this domain/URL is included in the success response.  

The **Arc Proxy** is a new component that runs as its own pod (called "Azure Arc Proxy"). This component acts as a forward proxy used by Azure Arc agents and extensions. There is no configuration required on your part for the Azure Arc Proxy. As of [version 1.21.10 of the Arc-enabled Kubernetes agents](release-notes.md), this pod is now part of the core Arc agents, and it runs within the context of an Arc-enabled Kubernetes Cluster.  

When the gateway is in place, traffic flows via the following hops: Arc Agentry → Azure Arc Proxy → Enterprise Proxy → Arc gateway → Target Service as shown below.

<-- Image TK -->

## Current limitations

During the public preview, the following limitations apply. Consider these factors when planning your configuration.

- TLS terminating proxies are not supported with the Arc gateway.
- You can't use ExpressRoute/site-to-site VPN or private endpoints in addition to the Arc gateway.
- There is a limit of five Arc gateway resources per Azure subscription.

You can create an Arc gateway resource by using Azure CLI, Azure PowerShell, or in the Azure portal.

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

1. On a machine with access to Azure, run the following Azure CLI command to create your Arc gateway resource, replacing the placeholders with your desired values:

   ```azurepowershell
   New-AzArcgateway 
   -name <gateway's name> 
   -resource-group <resource group> 
   -location <region> 
   -subscription <subscription name or id> 
   -gateway-type public  
   -allowed-features *
   ```

### [Azure portal](#tab/azure-portal)

1. In the Azure portal, navigate to  **Azure Arc > Azure Arc gateway (preview)**.
1. Select **Create**.
1. Select a subscription and resource group.
1. For **Name**, enter a name for the Arc gateway resource.
1. For **Location**, specify a region.
1. Select **Next: Tags**, then enter one or more custom tags if desired.
1. Select **Review + Create**.
1. After reviewing the details, select **Create.** 
---

It generally takes about ten minutes to finish creating the Arc gateway resource.

## Confirm ...

## Step 3

### [Azure CLI](#tab/azure-cli)

content

### [Azure PowerShell](#tab/azure-powershell)

more content

---

## Step 4

TK