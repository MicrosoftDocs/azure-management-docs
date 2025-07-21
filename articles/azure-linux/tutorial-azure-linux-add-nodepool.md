---
title: Azure Linux Container Host for AKS tutorial - Add an Azure Linux node pool to your existing AKS cluster
description: In this Azure Linux Container Host for AKS tutorial, you learn how to add an Azure Linux node pool to your existing cluster.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 04/06/2025
# Customer intent: As a DevOps engineer, I want to add an Azure Linux node pool to my existing AKS cluster, so that I can accommodate varying compute and storage needs for my applications.
---

# Tutorial: Add an Azure Linux node pool to your existing AKS cluster

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321935)

In AKS, nodes with the same configurations are grouped together into node pools. Each pool contains the VMs that run your applications. In the previous tutorial, you created an Azure Linux Container Host cluster with a single node pool. To meet the varying compute or storage requirements of your applications, you can create additional user node pools.

In this tutorial, part two of five, you learn how to:

> [!div class="checklist"]
>
> * Add an Azure Linux node pool.
> * Check the status of your node pools.

In later tutorials, you learn how to migrate nodes to Azure Linux and enable telemetry to monitor your clusters.

## Prerequisites

* In the previous tutorial, you created and deployed an Azure Linux Container Host cluster. If you haven't done these steps and would like to follow along, start with [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-azure-linux-create-cluster.md).
* You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Add an Azure Linux node pool

To add an Azure Linux node pool into your existing cluster, use the `az aks nodepool add` command and specify `--os-sku AzureLinux`. The following example creates a node pool named *ALnodepool* that runs three nodes in the *testAzureLinuxCluster* cluster in the *testAzureLinuxResourceGroup* resource group. Environment variables are declared below and a random suffix is appended to the resource group and cluster names to ensure uniqueness.

```azurecli-interactive
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export NODEPOOL_NAME="np$RANDOM_SUFFIX"

az aks nodepool add \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name $NODEPOOL_NAME \
    --node-count 3 \
    --os-sku AzureLinux
```

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

> [!NOTE]
> The name of a node pool must start with a lowercase letter and can only contain alphanumeric characters. For Linux node pools the length must be between one and 12 characters.

## Check the node pool status

To see the status of your node pools, use the `az aks nodepool list` command and specify your resource group and cluster name. The same environment variable values declared earlier are used here.

```azurecli-interactive
az aks nodepool list --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME
```

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

## Next steps

In this tutorial, you added an Azure Linux node pool to your existing cluster. You learned how to:

> [!div class="checklist"]
>
> * Add an Azure Linux node pool.
> * Check the status of your node pools.

In the next tutorial, you learn how to migrate existing nodes to Azure Linux.

> [!div class="nextstepaction"]
> [Migrating to Azure Linux](./tutorial-azure-linux-migration.md)