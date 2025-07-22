---
title: include file
description: include file
services: container-registry
author: chasedmicrosoft

ms.service: azure-container-registry
ms.topic: include
ms.date: 05/19/2021
ms.author: doveychase
ms.custom: include file
# Customer intent: "As a DevOps engineer, I want to understand the implications of disabling public network access in a container registry, so that I can configure my environment properly while ensuring that critical services can still access it."
---

> [!IMPORTANT]
> Some functionality may be unavailable or require more configuration in a container registry that restricts access to private endpoints, selected subnets, or IP addresses.
>
> * When public network access to a registry is disabled, registry access by certain [trusted services](../allow-access-trusted-services.md) including Azure Security Center requires enabling a network setting to bypass the network rules.
> * Once the public network access is disabled, instances of certain Azure services including Azure DevOps Services are currently unable to access the container registry. 
> * Private endpoints are not currently supported with Azure DevOps managed agents. You will need to use a self-hosted agent with network line of sight to the private endpoint. 
> * If the registry has an approved private endpoint and public network access is disabled, repositories and tags can't be listed outside the virtual network using the Azure portal, Azure CLI, or other tools.
