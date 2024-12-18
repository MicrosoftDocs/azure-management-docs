---
title: 'Quickstart: Enable Azure Linux 3.0 for Azure Kubernetes Service (AKS) clusters and node pools (Generally Available)'
description: Learn how to enable Azure Linux 3.0 for AKS clusters and node pools.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: references_regions, devx-track-azurecli, linux-related-content
ms.topic: quickstart
ms.date: 10/10/2024

---
# Quickstart: Enable Azure Linux 3.0 for Azure Kubernetes Service (AKS) clusters and node pools (Generally Available)

Starting AKS version 1.32 Azure Linux 3.0 is the default Azure Linux node OS for AKS clusters and node pools. 

## Limitations

* Not supported on Kubernetes version 1.31 and below.

## Create new Azure Linux 3.0 clusters and node pools

Any new AKS clusters or node pools created using the `--os-sku=AzureLinux` flag and that run AKS version 1.32 default to Azure Linux 3.0. You can deploy clusters or node pools using the method of your choice to use Azure Linux 3.0 as the node OS:

    - [Quickstart with CLI](./quickstart-azure-cli.md)
    - [Quickstart with PowerShell](./quickstart-azure-powershell.md)
    - [Quickstart with Terraform](./quickstart-terraform.md)
    - [Quickstart with Azure Resource Manager (ARM)](./quickstart-azure-resource-manager-template.md)

# Upgrading Existing Azure Linux 2.0 clusters and node pools to Azure Linux 3.0

To upgrade existing Azure Linux 2.0 clusters and node pools to Azure Linux 3.0 you can upgrade to AKS version 1.32. No additional action is required. For more details on upgrading your AKS cluster see [Upgrade an AKS cluster] (https://learn.microsoft.com/en-us/azure/aks/upgrade-aks-cluster?tabs=azure-cli).  

## Next steps
For more information about Azure Linux 3.0, see [What's new with Azure Linux 3.0?](./intro-azure-linux.md).