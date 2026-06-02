---
title: "Tutorial: Create a Cluster with the Azure Linux Container Host for Azure Kubernetes Service (AKS)"
description: In this Azure Linux tutorial, you learn how to create a cluster with the Azure Linux Container Host for Azure Kubernetes Service (AKS).
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Create a cluster with the Azure Linux Container Host for Azure Kubernetes Service (AKS)

In this tutorial, part _one of five_, you learn how to:

> [!div class="checklist"]
>
> - Install the Kubernetes CLI, `kubectl`.
> - Create an Azure resource group.
> - Create and deploy an Azure Linux Container Host for AKS cluster.
> - Configure `kubectl` to connect to your Azure Linux Container Host cluster.

In later tutorials, you learn how to add an Azure Linux node pool to an existing cluster and migrate existing nodes to Azure Linux.

## Prerequisites

- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Set environment variables

Set the following environment variables to create unique resource names for each deployment:

```bash
export RESOURCE_GROUP="<your-resource-group-name>"
export REGION="<your-region>"
export CLUSTER_NAME="<your-cluster-name>"
```

## Create a resource group

When creating a resource group in Azure, you're required to specify a location. This location is the storage location of your resource group metadata and where your resources run in Azure if you don't specify another region when creating a resource.

Create a resource group using the [`az group create`](/cli/azure/group#az_group_create) command.

```azurecli-interactive
az group create --name $RESOURCE_GROUP --location $REGION
```

Example output:

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

## Create an Azure Linux Container Host for AKS cluster

Create an AKS cluster using the [`az aks create`](/cli/azure/aks#az_aks_create) command with the `--os-sku` parameter to provision the Azure Linux Container Host with an Azure Linux image.

```azurecli-interactive
az aks create --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --os-sku AzureLinux
```

Example output:

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

Configure `kubectl` to connect to your Kubernetes cluster using the [`az aks get-credentials`](/cli/azure/aks#az_aks_get_credentials) command.

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Verify the connection to your cluster

Verify the connection to your cluster using the [`kubectl get nodes`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command. The command returns a list of nodes in your cluster.

```bash
kubectl get nodes
```

Example output:

<!-- expected_similarity=0.3 -->
```text
NAME                           STATUS   ROLES   AGE     VERSION
aks-nodepool1-00000000-0       Ready    agent   10m     v1.20.7
aks-nodepool1-00000000-1       Ready    agent   10m     v1.20.7
```

## Next step

In this tutorial, you created and deployed an Azure Linux Container Host cluster. In the next tutorial, you learn how to add an Azure Linux node pool to an existing cluster.

> [!div class="nextstepaction"]
> [Add an Azure Linux node pool](./tutorial-add-azure-linux-node-pool.md)
