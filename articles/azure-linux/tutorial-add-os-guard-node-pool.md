---
title: "Tutorial: Add an Azure Linux with OS Guard (preview) Node Pool to your Existing AKS Cluster"
description: In this Azure Linux tutorial, you learn how to add an Azure Linux with OS Guard (preview) node pool to your existing Azure Kubernetes Service (AKS) cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Add an Azure Linux with OS Guard (preview) node pool to an existing Azure Kubernetes Service (AKS) cluster

[!INCLUDE [os-guard replacement](./includes/os-guard-replacement.md)]

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321935)

In AKS, nodes with the same configurations are grouped together into node pools. Each node pool contains the virtual machines (VMs) that run your applications. In the previous tutorial, you created an Azure Linux with OS Guard cluster with a single node pool. To meet the varying compute, storage, or security requirements of your applications, you can add user node pools.

In this tutorial, part _two of five_, you learn how to:

> [!div class="checklist"]
>
> - Add an Azure Linux with OS Guard node pool to an existing cluster.
> - Check the status of your node pools.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with Azure Linux with OS Guard for AKS](./tutorial-create-cluster-os-guard-aks.md).

In later tutorials, you learn how to migrate nodes to Azure Linux with OS Guard and enable telemetry to monitor your clusters.

## Prerequisites

- In the previous tutorial, you created and deployed an Azure Linux with OS Guard cluster. If you haven't completed these steps and want to follow along, see [Tutorial 1: Create a cluster with Azure Linux with OS Guard for AKS](./tutorial-create-cluster-os-guard-aks.md).
- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

[!INCLUDE [azure linux with os guard limitations](./includes/os-guard-limitations.md)]

## Add an Azure Linux with OS Guard node pool

Add an Azure Linux with OS Guard node pool into your existing cluster using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command and specify `--os-sku AzureLinuxOSGuard`. Enabling [FIPS](/azure/aks/enable-fips-nodes), [secure boot](/azure/aks/use-trusted-launch), and [vtpm](/azure/aks/use-trusted-launch) is also required to use Azure Linux with OS Guard. The following example creates a node pool named _osgNodePool_ that adds three nodes to the cluster:

```azurecli-interactive

az aks nodepool add \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name osgNodePool \
    --node-count 3 \
    --os-sku AzureLinuxOSGuard
    --node-osdisk-type Managed 
    --enable-fips-image 
    --enable-secure-boot 
    --enable-vtpm
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "agentPoolType": "VirtualMachineScaleSets",
  "count": 3,
  "name": "osgNodePool",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "testAzureLinuxOSGuardResourceGroupxxxxx",
  "type": "Microsoft.ContainerService/managedClusters/agentPools"
}
```

> [!NOTE]
> The name of a node pool must start with a lowercase letter and can only contain alphanumeric characters. For Linux node pools, the length must be between one and 12 characters.

## Check the node pool status

Check the status of your node pools using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az-aks-nodepool-list) command.

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
    "count": 3,
    "enableAutoScaling": false,
    "enableEncryptionAtHost": false,
    "enableFips": false,
    "enableNodePublicIp": false,
    "id": "/subscriptions/REDACTED/resourcegroups/myAKSResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/myAKSClusterxxxxx/agentPools/npxxxxxx",
    "maxPods": 110,
    "mode": "User",
    "name": "npxxxxxx",
    "nodeImageVersion": "AzureLinuxContainerHost-2025.10.03",
    "orchestratorVersion": "1.32.6",
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
    "nodeImageVersion": "AzureLinuxOSGuard-2025.10.03",
    "orchestratorVersion": "1.32.6",
    "osDiskSizeGb": 128,
    "osDiskType": "Managed",
    "osSku": "AzureLinuxOSGuard",
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

In this tutorial, you added an Azure Linux with OS Guard node pool to your existing cluster. In the next tutorial, you learn how to migrate existing nodes to Azure Linux with OS Guard.

> [!div class="nextstepaction"]
> [Migrate to Azure Linux with OS Guard](./tutorial-migrate-os-guard-aks.md)
