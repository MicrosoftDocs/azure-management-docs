---
title: 'Quickstart: Enable Azure Linux 3.0 for Azure Kubernetes Service (AKS) clusters and node pools'
description: Learn how to enable Azure Linux 3.0 for AKS clusters and node pools.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: references_regions, devx-track-azurecli, linux-related-content
ms.topic: quickstart
ms.date: 10/10/2024

# Customer intent: As a Kubernetes administrator, I want to enable Azure Linux 3.0 for my AKS clusters and node pools, so that I can take advantage of the latest features and improvements in the node operating system.
---
# Quickstart: Enable Azure Linux 3.0 for Azure Kubernetes Service (AKS) clusters and node pools

Starting with AKS version 1.32, Azure Linux 3.0 is the default Azure Linux node OS for AKS clusters and node pools. 

## Create new Azure Linux 3.0 clusters and node pools

Any new AKS clusters or node pools created using the `--os-sku=AzureLinux` flag and that run AKS version 1.32 default to Azure Linux 3.0. You can deploy clusters or node pools using the method of your choice to use Azure Linux 3.0 as the node OS:

* [Quickstart with CLI](./quickstart-azure-cli.md)
* [Quickstart with PowerShell](./quickstart-azure-powershell.md)
* [Quickstart with Terraform](./quickstart-terraform.md)
* [Quickstart with Azure Resource Manager (ARM)](./quickstart-azure-resource-manager-template.md)

## Upgrade existing Azure Linux 2.0 clusters and node pools to Azure Linux 3.0

To upgrade existing Azure Linux 2.0 clusters and node pools to Azure Linux 3.0, you can upgrade them to AKS version 1.32. For more information about AKS cluster upgrades, see [Upgrade an AKS cluster](/azure/aks/upgrade-aks-cluster). 

Alternatively, starting with Kubernetes version 1.28+, you can perform an in-place node pool update from `--os-sku AzureLinux` to `--os-sku AzureLinux3`, enabling you to migrate existing node pools from Azure Linux 2.0 to 3.0 while remaining on the same Kubernetes version. Your node pool will automatically reimage to the equivalent Azure Linux 3.0 node image. See documentation [here](/azure/aks/upgrade-os-version).    

## Next steps
For more information about Azure Linux 3.0, see [What's new with Azure Linux 3.0?](./intro-azure-linux.md).
