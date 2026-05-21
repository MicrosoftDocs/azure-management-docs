---
title: "Tutorial: Add an Azure Linux node pool to your existing Azure Kubernetes Service (AKS) cluster"
description: In this Azure Linux tutorial, you learn how to add an Azure Linux node pool to your existing Azure Kubernetes Service (AKS) cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Add an Azure Linux node pool to your existing Azure Kubernetes Service (AKS) cluster

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321935)

In AKS, nodes with the same configurations are grouped together into node pools. Each pool contains the VMs that run your applications. In the previous tutorial, you created an Azure Linux Container Host cluster with a single node pool. To meet the varying compute or storage requirements of your applications, you can create extra user node pools.

In this tutorial, part _two of five_, you learn how to:

> [!div class="checklist"]
>
> - Add an Azure Linux node pool.
> - Check the status of your node pools.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).

In later tutorials, you learn how to migrate nodes to Azure Linux and enable telemetry to monitor your clusters.

## Prerequisites

- In the previous tutorial, you created and deployed an Azure Linux Container Host cluster. If you haven't done these steps and would like to follow along, start with [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).
- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Set environment variables

Set the following environment variables to create unique resource names for each deployment. Replace the placeholder `<your-node-pool-name>` with a name of your choice. You can optionally append a random suffix to ensure uniqueness. The name of a node pool must start with a lowercase letter and can only contain alphanumeric characters. For Linux node pools the length must be between one and 12 characters.

```bash
# Set random suffix for uniqueness
export RANDOM_SUFFIX=$(openssl rand -hex 3)

# Set node pool name
export NODE_POOL_NAME="<your-node-pool-name>$RANDOM_SUFFIX"
```

## Add an Azure Linux node pool

Add an Azure Linux node pool to your existing cluster using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az_aks_nodepool_add) command and specify `--os-sku AzureLinux`. The following example creates a node pool that runs three nodes in the cluster from [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).

```azurecli-interactive
az aks nodepool add \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name $NODE_POOL_NAME \
    --node-count 3 \
    --os-sku AzureLinux
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "agentPoolType": "VirtualMachineScaleSets",
  "count": 3,
  "name": "alnodepool",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "testAzureLinuxResourceGroupxxxxx",
  "type": "Microsoft.ContainerService/managedClusters/agentPools"
}
```

## Check the node pool status

Check the status of your node pools using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az_aks_nodepool_list) command.

```azurecli-interactive
az aks nodepool list --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME
```

Example output:

<!-- expected_similarity=0.3 -->
```output
[
  {
    "agentPoolType": "VirtualMachineScaleSets",
    "availabilityZones": null,
    "count": 1,
    "enableAutoScaling": false,
    "enableEncryptionAtHost": false,
    "enableFips": false,
    "enableNodePublicIp": false,
    "id": "/subscriptions/REDACTED/resourcegroups/myAKSResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/myAKSClusterxxxxx/agentPools/nodepoolx",
    "maxPods": 110,
    "mode": "System",
    "name": "nodepoolx",
    "nodeImageVersion": "AKSUbuntu-1804gen2containerd-2023.06.06",
    "orchestratorVersion": "1.25.6",
    "osDiskSizeGb": 128,
    "osDiskType": "Managed",
    "osSku": "Ubuntu",
    "osType": "Linux",
    "powerState": {
      "code": "Running"
    },
    "provisioningState": "Succeeded",
    "resourceGroup": "myAKSResourceGroupxxxxx",
    "type": "Microsoft.ContainerService/managedClusters/agentPools",
    "vmSize": "Standard_DS2_v2"
  },
  {
    "agentPoolType": "VirtualMachineScaleSets",
    "availabilityZones": null,
    "count": 3,
    "enableAutoScaling": false,
    "enableEncryptionAtHost": false,
    "enableFips": false,
    "enableNodePublicIp": false,
    "id": "/subscriptions/REDACTED/resourcegroups/myAKSResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/myAKSClusterxxxxx/agentPools/npxxxxxx",
    "maxPods": 110,
    "mode": "User",
    "name": "npxxxxxx",
    "nodeImageVersion": "AzureLinuxContainerHost-2023.06.06",
    "orchestratorVersion": "1.25.6",
    "osDiskSizeGb": 128,
    "osDiskType": "Managed",
    "osSku": "AzureLinux",
    "osType": "Linux",
    "powerState": {
      "code": "Running"
    },
    "provisioningState": "Succeeded",
    "resourceGroup": "myAKSResourceGroupxxxxx",
    "type": "Microsoft.ContainerService/managedClusters/agentPools",
    "vmSize": "Standard_DS2_v2"
  }
]
```

## Next step

In this tutorial, you added an Azure Linux node pool to your existing cluster. In the next tutorial, you learn how to migrate existing nodes to Azure Linux.

> [!div class="nextstepaction"]
> [Migrate nodes to Azure Linux](./tutorial-migrate-azure-linux-aks.md)
