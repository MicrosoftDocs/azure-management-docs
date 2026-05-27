---
title: Supported Azure Services in Azure Linux
description: Learn about the Azure services that are supported on the Azure Linux platform, including agents, extensions, and add-ons for AKS.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 05/04/2026
---

# Supported Azure services in Azure Linux

Azure Linux supports a range of Azure agents, extensions, and add-ons. These services are available as RPMs on [packages.microsoft.com](https://packages.microsoft.com), container images on Azure Container Registry (ACR), virtual machine (VM) extensions, and add-ons for Azure Kubernetes Service (AKS). This article provides an overview of the supported services and their compatibility with the Azure Linux platform.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Supported agents and extensions

The following table highlights commonly used agents and extensions in Azure Linux. For the complete list of supported agents and their compatibility status, see the respective documentation for each agent or extension.

| Agent / Extension | Description |
| ----------------- | ----------- |
| **Azure Monitor** | End-to-end observability for applications and infrastructure |
| **Azure Linux Agent (WALA)** | VM provisioning, networking, and extension management |
| **Azure CLI** | Command-line interface for managing Azure resources |
| **Key Vault** | Secure retrieval of secrets, keys, and certificates from Azure Key Vault |
| **Microsoft Defender for Cloud** | Threat detection, vulnerability management, and security monitoring for cloud workloads |
| **cloud-init** | Industry-standard multi-distribution method for cross-platform cloud instance initialization |
| **Azure Arc** | Management and governance of servers and Kubernetes clusters across on-premises, multicloud, and edge environments |
| **.NET** | Cross-platform runtime for building cloud, web, and microservice applications |
| **Microsoft Build of OpenJDK** | Microsoft's distribution of the Java Development Kit for running Java-based workloads |
| **Azure Resource Manager (ARM) templates** | JSON-based templates for defining and deploying Azure infrastructure as code |

## Add-ons and extensions for AKS

All existing and future AKS extensions and add-ons support Azure Linux. For more information, see the following resources:

- [Available add-ons](/azure/aks/integrations)
- [Currently available extensions](/azure/aks/cluster-extensions#currently-available-extensions)

## Related content

To learn more about Azure Linux, see the following resources:

- [What is Azure Linux?](./azure-linux-overview.md)
- [Azure Linux architecture overview](./whats-new-azure-linux-4.md)
