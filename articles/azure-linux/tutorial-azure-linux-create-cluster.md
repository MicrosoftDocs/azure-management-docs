---
title: Azure Linux Container Host for AKS tutorial - Create a cluster
description: In this Azure Linux Container Host for AKS tutorial, you learn how to create an AKS cluster with Azure Linux.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/06/2025
# Customer intent: "As a cloud engineer, I want to create an Azure Kubernetes Service (AKS) cluster with Azure Linux Container Host, so that I can deploy and manage containerized applications effectively in a production-ready environment."
---

# Tutorial: Create a cluster with the Azure Linux Container Host for AKS

To create a cluster with the Azure Linux Container Host, you need:
1. An Azure resource group, which is a logical container for deploying and managing Azure resources.
2. [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes), a hosted Kubernetes service that allows you to quickly create a production-ready Kubernetes cluster.

This tutorial is part one of a five-part series. You'll learn how to:

> [!div class="checklist"]
> * Install the Kubernetes CLI, `kubectl`.
> * Create an Azure resource group.
> * Create and deploy an Azure Linux Container Host cluster.
> * Configure `kubectl` to connect to your Azure Linux Container Host cluster.

Later tutorials in this series cover adding an Azure Linux node pool to an existing cluster and migrating existing nodes to Azure Linux.

[!INCLUDE [azure-linux-retirement](./includes/azure-linux-retirement.md)]

## Prerequisites

- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Create a resource group

When you create a resource group, you must specify a location. This location determines: 
- Where the resource group metadata is stored.
- Where your resources run in Azure if you don't specify another region during resource creation.

Before running the command, declare environment variables to ensure unique resource names for each deployment.

```bash
export REGION="EastUS2"
az group create --name $RESOURCE_GROUP_NAME --location $REGION
```

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxResourceGroupxxxxx",
  "location": "EastUS2",
  "managedBy": null,
  "name": "testAzureLinuxResourceGroupxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create an Azure Linux Container Host cluster

Create an AKS cluster using the `az aks create` command with the `--os-sku` parameter to provision an Azure Linux Container Host cluster. The following example creates an Azure Linux Container Host cluster. 

```bash
az aks create --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP_NAME --os-sku AzureLinux
```

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/testAzureLinuxClusterxxxxx",
  "location": "WestUS2",
  "name": "testAzureLinuxClusterxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.ContainerService/managedClusters"
}
```

After a few minutes, the command completes and returns JSON-formatted information about the cluster.

## Connect to the cluster using kubectl

To configure `kubectl` to connect to your Kubernetes cluster, use the `az aks get-credentials` command. The following example gets credentials for the Azure Linux Container Host cluster using the resource group and cluster name created earlier:

```azurecli
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

To verify the connection to your cluster, run the [kubectl get nodes](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes:

```azurecli-interactive
kubectl get nodes
```

<!-- expected_similarity=0.3 -->
```text
NAME                           STATUS   ROLES   AGE     VERSION
aks-nodepool1-00000000-0       Ready    agent   10m     v1.20.7
aks-nodepool1-00000000-1       Ready    agent   10m     v1.20.7
```

## Next steps

In this tutorial, you created and deployed an Azure Linux Container Host cluster. You learned how to: 

> [!div class="checklist"]
> * Install the Kubernetes CLI, `kubectl`.
> * Create an Azure resource group.
> * Create and deploy an Azure Linux Container Host cluster.
> * Configure `kubectl` to connect to your Azure Linux Container Host cluster.

In the next tutorial, you'll learn how to add an Azure Linux node pool to an existing cluster.

> [!div class="nextstepaction"]
> [Add an Azure Linux node pool](./tutorial-azure-linux-add-nodepool.md)
