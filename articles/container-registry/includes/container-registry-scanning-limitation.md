---
title: include file
description: include file
services: container-registry
author: rayoef
ms.service: azure-container-registry
ms.topic: include
ms.date: 02/11/2026
ms.author: rayoflores
ms.custom: include file
# Customer intent: "As a DevOps engineer, I want to understand the implications of disabling public network access in a container registry, so that I can configure my environment properly while ensuring that critical services can still access it."
---

> [!IMPORTANT]
> If your container registry restricts access to private endpoints, selected subnets, or IP addresses, some functionality might be unavailable or require more configuration.
>
> * When you disable public network access to a registry, certain [trusted services](../allow-access-trusted-services.md), including Microsoft Defender for Cloud, can access the registry only if you enable a network setting to bypass the network rules.
> * Once you disable the public network access, instances of certain Azure services, including Azure DevOps Services, can't access the container registry.
> * Private endpoints aren't currently supported with agents managed by Azure DevOps. You need to use a self-hosted agent with network line of sight to the private endpoint.
> * If the registry has an approved private endpoint and you disable public network access, you can't list repositories and tags outside the virtual network by using the Azure portal, Azure CLI, or other tools.
