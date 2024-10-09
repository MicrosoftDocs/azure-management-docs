---
title: 'Quickstart: Deploy an Azure Linux Container Host for AKS cluster by using the Azure CLI'
description: Learn how to quickly create an Azure Linux Container Host for AKS cluster using the Azure CLI.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: references_regions, devx-track-azurecli, linux-related-content
ms.topic: quickstart
ms.date: 04/18/2023
---

# How To: Enable Azure Linux 3.0 Preview 

This document goes over how to enable Azure Linux 3.0 to be used as the default Azure Linux node OS for new AKS clusters and nodepools. Once enabled, new AKS clusters or nodepools created with the `--os-sku=AzureLinux` option will default to using Azure Linux 3.0 on AKS versions v1.31 and greater. 

[!INCLUDE [preview features callout](~/reusable-content/ce-skilling/azure/includes/aks/includes/preview/preview-callout.md)]

## Enable the feature flag

1. Register the `AzureLinuxV3Preview` feature flag by using the [az feature register][az-feature-register] command, as shown in the following example:

  ```azurecli-interactive
  az feature register --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"
  ```

2. It takes a few minutes for the status to show *Registered*. Verify the registration status by using the [az feature show][az-feature-show] command:

  ```azurecli-interactive
  az feature show --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"
  ```

When the status reflects *Registered*, refresh the registration of the *Microsoft.ContainerService* resource provider by using the [az provider register][az-provider-register] command:

   ```azurecli-interactive
      az provider register --namespace "Microsoft.ContainerService"
   ```

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