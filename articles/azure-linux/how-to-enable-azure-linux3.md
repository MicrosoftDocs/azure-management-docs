---
title: 'Quickstart: Enable Azure Linux 3.0 for AKS clusters and node pools (Preview) '
description: Learn how to enable Azure Linux 3.0 for AKS clusters and node pools.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: references_regions, devx-track-azurecli, linux-related-content
ms.topic: quickstart
ms.date: 10/10/2024

---
# Quickstart: Enable Azure Linux 3.0 for AKS clusters and node pools (Preview)

This article shows you how to enable Azure Linux 3.0 as the default Azure Linux node OS for new AKS clusters and node pools running AKS version 1.31 or later. Once enabled, any new AKS clusters or node pools created with the `--os-sku=AzureLinux` option will default to using Azure Linux 3.0.

## Limitations

* Not supported on Kubernetes version 1.30 and below.

## Enable Azure Linux 3.0  

1. Register the `AzureLinuxV3Preview` feature flag using the [`az feature register`](/cli/azure/feature#az-feature-register) command.  

    ```azurecli-interactive  
     az feature register --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"  
    ```  

    It takes a few minutes for the status to show *Registered*.  

1. Verify the registration status using the [`az feature show`](/cli/azure/feature#az-feature-show) command.  

Once these steps are completed, you can deploy the cluster using the method of your choice to use Azure Linux 3.0 as the node OS: 

- [Quickstart with CLI](./quickstart-azure-cli.md)
- [Quickstart with PowerShell](./quickstart-azure-powershell.md)
- [Quickstart with Terraform](./quickstart-terraform.md)
- [Quickstart with ARM](./quickstart-azure-resource-manager-template.md)

## Unregister the Azure Linux 3.0 Feature Flag

1. Unregister the `AzureLinuxV3Preview` feature flag by using the [az feature unregister][az-feature-unregister] command. 

```azurecli-interactive
az feature unregister --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"
```

2. Refresh the registration of the *Microsoft.ContainerService* resource provider by using the [az provider register][az-provider-register] command:

```azurecli-interactive
    az provider register --namespace "Microsoft.ContainerService"
```