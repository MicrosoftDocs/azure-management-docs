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

The **Arc Proxy** is a new component that runs as its own pod (called "Azure Arc Proxy"). This component acts as a forward proxy used by Azure Arc agents and extensions. There is no configuration required on your part for the Azure Arc Proxy. This pod is now part of the core Arc agents, and it runs within the context of an Arc-enabled Kubernetes Cluster.  

When the gateway is in place, traffic flows via the following hops: Arc Agentry → Azure Arc Proxy → Enterprise Proxy → Arc gateway → Target Service as shown below.

<-- Image TK -->

## Current limitations

During the public preview, the following limitations apply. Consider these factors when planning your configuration.

- TLS terminating proxies are not supported with the Arc gateway.
- You can't use ExpressRoute/site-to-site VPN or private endpoints in addition to the Arc gateway.
- There is a limit of five Arc gateway resources per Azure subscription.

## Use the Arc gateway (preview)

You can create an Arc gateway resource by using Azure CLI, Azure PowerShell, or in the Azure portal.

