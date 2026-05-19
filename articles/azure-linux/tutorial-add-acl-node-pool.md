---
title: "Tutorial: Add an Azure Container Linux (ACL) node pool to your existing Azure Kubernetes Service (AKS) cluster"
description: In this Azure Linux tutorial, you learn how to add an Azure Container Linux (ACL) node pool to your existing AKS cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Add an Azure Container Linux (ACL) node pool to your existing Azure Kubernetes Service (AKS) cluster

In AKS, nodes with the same configurations are grouped together into node pools. Each node pool contains the virtual machines (VMs) that run your applications. In the previous tutorial, you created an ACL cluster with a single node pool. To meet the varying compute, storage, or security requirements of your applications, you can add user node pools.

In this tutorial, part _two of five_, you learn how to:

> [!div class="checklist"]
>
> - Add an ACL node pool to an existing cluster.
> - Check the status of your node pools.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).

In later tutorials, you learn how to migrate nodes to ACL and enable telemetry to monitor your clusters.

## Prerequisites

- In the previous tutorial, you created and deployed an ACL cluster. If you haven't completed these steps and want to follow along, see [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).
- Azure Container Linux requires Azure CLI version 2.86.0 or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

[!INCLUDE [azure container linux limitations](./includes/acl-limitations.md)]

## Add an ACL node pool

Add an ACL node pool into your existing cluster using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command and specify `--os-sku AzureContainerLinux`. The following example creates a node pool named _aclpool_ that adds three nodes to the cluster:

```azurecli-interactive
az aks nodepool add \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name aclpool \
    --node-count 3 \
    --os-sku AzureContainerLinux
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "agentPoolType": "VirtualMachineScaleSets",
  "count": 3,
  "name": "aclpool",
  "osType": "Linux",
  "osSku": "AzureContainerLinux",
  "provisioningState": "Succeeded",
  "resourceGroup": "myACLResourceGroup",
  "type": "Microsoft.ContainerService/managedClusters/agentPools"
}
```

> [!NOTE]
> The name of a node pool must start with a lowercase letter and can only contain alphanumeric characters. For Linux node pools, the length must be between one and 12 characters.

## Check the node pool status

Check the status of your node pools using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az-aks-nodepool-list) command.

```azurecli-interactive
az aks nodepool list --resource-group $RESOURCE_GROUP_NAME --cluster-name $CLUSTER_NAME
```

Example output:

<!-- expected_similarity=0.3 -->
```output
[
  {
    "agentPoolType": "VirtualMachineScaleSets",
    "count": 3,
    "name": "nodepool1",
    "nodeImageVersion": "AKSAzureContainerLinux-202606.01.0",
    "osSku": "AzureContainerLinux",
    "osType": "Linux",
    "provisioningState": "Succeeded",
    "resourceGroup": "myACLResourceGroup",
    "vmSize": "Standard_DS2_v2"
  },
  {
    "agentPoolType": "VirtualMachineScaleSets",
    "count": 3,
    "name": "aclpool",
    "nodeImageVersion": "AKSAzureContainerLinux-202606.01.0",
    "osSku": "AzureContainerLinux",
    "osType": "Linux",
    "provisioningState": "Succeeded",
    "resourceGroup": "myACLResourceGroup",
    "vmSize": "Standard_DS2_v2"
  }
]
```

## Next step

In this tutorial, you added an ACL node pool to your existing cluster. In the next tutorial, you learn how to migrate existing nodes to ACL.

> [!div class="nextstepaction"]
> [Migrate to ACL](./tutorial-migrate-acl-aks.md)
