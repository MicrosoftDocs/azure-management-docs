---
title: "Tutorial: Create a Cluster with Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)"
description: In this Azure Linux tutorial, you learn how to create a cluster with Azure Container Linux (ACL) for Azure Kubernetes Service (AKS).
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Create a cluster with Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)

In this tutorial, part _one of five_, you learn how to:

> [!div class="checklist"]
>
> - Install the Kubernetes CLI, `kubectl`.
> - Create an Azure resource group.
> - Create and deploy an ACL cluster.
> - Configure `kubectl` to connect to your ACL cluster.

In later tutorials, you learn how to add an ACL node pool to an existing cluster and migrate existing nodes to ACL.

[!INCLUDE [azure container linux limitations](./includes/acl-limitations.md)]

## Prerequisites

- Azure Container Linux requires Azure CLI version 2.86.0 or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

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
  "id": "/subscriptions/xxxxx/resourceGroups/myACLResourceGroup",
  "location": "westus",
  "managedBy": null,
  "name": "myACLResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create an ACL cluster

Create an AKS cluster using the [`az aks create`](/cli/azure/aks#az-aks-create) command with the `--os-sku AzureContainerLinux` parameter to provision an ACL cluster. The following example creates an ACL cluster with three nodes:

```azurecli-interactive
az aks create \
  --name $CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --os-sku AzureContainerLinux \
  --node-count 3 \
  --generate-ssh-keys
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myACLResourceGroup/providers/Microsoft.ContainerService/managedClusters/myACLCluster",
  "location": "westus",
  "name": "myACLCluster",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.ContainerService/managedClusters"
}
```

After a few minutes, the command completes and returns JSON-formatted information about the cluster.

## Connect to the cluster using kubectl

Configure `kubectl` to connect to your Kubernetes cluster using the [`az aks get-credentials`](/cli/azure/aks#az-aks-get-credentials) command. The following example gets credentials for the ACL cluster:

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Verify the connection to your cluster

Verify the connection to your cluster using the [`kubectl get nodes`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes.

```bash
kubectl get nodes
```

Example output:

<!-- expected_similarity=0.3 -->
```text
NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-00000000-0            Ready    agent   10m     v1.34.0
aks-nodepool1-00000000-1            Ready    agent   10m     v1.34.0
aks-nodepool1-00000000-2            Ready    agent   10m     v1.34.0
```

## Next step

In this tutorial, you created and deployed an ACL cluster. In the next tutorial, you learn how to add an ACL node pool to an existing cluster.

> [!div class="nextstepaction"]
> [Add an ACL node pool](./tutorial-add-acl-node-pool.md)
