---
title: Azure Linux with OS Guard for Azure Kubernetes Service (AKS) Tutorial - Migrate to Azure Linux with OS Guard
description: In this Azure Linux with OS Guard for AKS tutorial, you learn how to migrate your nodes to Azure Linux with OS Guard nodes.
author: florataagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: devx-track-azurecli, linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 09/24/2025
# Customer intent: As a cloud administrator, I want to migrate existing AKS node pools to Azure Linux with OS Guard, so that I can take advantage of advanced security features and ensure optimal performance for my containerized applications.
---

# Tutorial: Migrate nodes to Azure Linux with OS Guard

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321934)

In this tutorial, part three of five, you migrate your existing nodes to Azure Linux with OS Guard. You can migrate your existing nodes using one of the following methods:

- Remove existing node pools and add new Azure Linux with OS Guard node pools.
- In-place OS SKU migration.

If you don't have any existing nodes to migrate, skip to the [next tutorial](./tutorial-azure-linux-os-guard-telemetry-monitor.md). In later tutorials, you learn how to enable telemetry and monitoring in your clusters and upgrade Azure Linux with OS Guard nodes.

## Prerequisites

- In previous tutorials, you created and deployed an Azure Linux with OS Guard for AKS cluster. To complete this tutorial, you need an AKS cluster with an Azure Linux with OS Guard node pool. If you haven't completed this step and want to follow along, see [Tutorial 2: Add an Azure Linux with OS Guard node pool to your existing AKS cluster](./tutorial-azure-linux-os-guard-add-nodepool.md).

    > [!NOTE]
    > When adding a new Azure Linux with OS Guard node pool, you need to add at least one as `--mode System`. Otherwise, AKS won't allow you to delete your existing node pool.

- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

## Add Azure Linux with OS Guard node pools and remove existing node pools

Add a new Azure Linux with OS Guard node pool using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command. This command adds a new node pool to your cluster with the `--mode System` flag, which makes it a system node pool. System node pools are required for Azure Linux with OS Guard clusters.

```azurecli-interactive
# Declare environment variables with a random suffix for uniqueness
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export NODE_POOL_NAME="np$RANDOM_SUFFIX"
az aks nodepool add --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODE_POOL_NAME --mode System --os-sku AzureLinuxOSGuard
```

Example output:

<!-- expected_similarity=0.3 -->

```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxx/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/nodePools/systempool",
  "name": "systempool",
  "provisioningState": "Succeeded"
}
```

2. Remove your existing nodes using the `az aks nodepool delete` command.

## In-place OS SKU migration

You can migrate your existing Ubuntu or Azure Linux node pools to Azure Linux with OS Guard by changing the OS SKU of the node pool, which rolls the cluster through the standard node image upgrade process. This new feature doesn't require the creation of new node pools.

### Limitations

There are several settings that can block the OS SKU migration request. To ensure a successful migration, review the following guidelines and limitations:

- The OS SKU migration feature isn't available through PowerShell or the Azure portal.
- The OS SKU migration feature doesn't support renaming existing node pools.
- Ubuntu, Azure Linux, and Azure Linux with OS Guard are the only supported Linux OS SKU migration targets.
- Trusted launch is required by default for Azure Linux with OS Guard, customers need to have trusted launch enabled to be able to migrate to Azure Linux with OS Guard. Since Trusted Launch cannot be enabled on existing node pools, this may require new node pool creation. 
- Customers using Gen 1 only vm sizes will not be able to migrate to Azure Linux with OS Guard since there is no supported Gen 1 image. They will need to create new node pools with a vm size that supports gen 2.
- An Ubuntu OS SKU with `UseGPUDedicatedVHD` enabled can't perform an OS SKU migration.
- An Ubuntu OS SKU with CVM 20.04 enabled can't perform an OS SKU migration.
- Node pools with Kata enabled can't perform an OS SKU migration.
- Windows OS SKU migration isn't supported.
- OS SKU migration from Mariner to Azure Linux is supported, but rolling back to Mariner isn't supported.

### Prerequisites

- An existing AKS cluster with at least one Azure Linux node pool.
- We recommend that you ensure your workloads configure and run successfully on the Azure Linux with OS Guard container host before attempting to use the OS SKU migration feature by [deploying an Azure Linux with OS Guard cluster](./quickstart-osguard-azure-cli.md) in dev/prod and verifying your service remains healthy.
- Ensure the migration feature is working for you in test/dev before using the process on a production cluster.
- Ensure that your pods have enough [Pod Disruption Budget](/azure/aks/operator-best-practices-scheduler#plan-for-availability-using-pod-disruption-budgets) to allow AKS to move pods between VMs during the upgrade.
- You need Azure CLI version [2.61.0](/cli/azure/release-notes-azure-cli#may-21-2024) or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

### [Azure CLI](#tab/azure-cli)

#### Migrate the OS SKU of your Azure Linux Container Host node pool

- Migrate the OS SKU of your node pool to Azure Linux with OS Guard using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command. This command updates the OS SKU for your node pool from Azure Linux to Azure Linux with OS Guard. The OS SKU change triggers an immediate upgrade operation, which takes several minutes to complete.

```azurecli-interactive
az aks nodepool update --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODE_POOL_NAME --os-sku AzureLinuxOSGuard
```

Example output:

<!-- expected_similarity=0.3 -->

```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxx/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/nodePools/nodepool1",
  "name": "nodepool1",
  "osSku": "AzureLinuxOSGuard",
  "provisioningState": "Succeeded"
}
```

> [!NOTE]
> If you experience issues during the OS SKU migration, you can [roll back to your previous OS SKU](#rollback).

### [ARM template](#tab/arm-template)

#### Example ARM templates

##### 0base.json

```json
 {
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerService/managedClusters",
      "apiVersion": "2023-07-01",
      "name": "akstestcluster",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayname": "Demo of AKS Nodepool Migration"
      },
      "identity": {
        "type": "SystemAssigned"
      },
      "properties": {
        "enableRBAC": true,
        "dnsPrefix": "testcluster",
        "agentPoolProfiles": [
          {
            "name": "testnp",
            "count": 3,
            "vmSize": "Standard_D4a_v4",
            "osType": "Linux",
            "osSku": "AzureLinux",
            "mode": "System"
          }
        ]
      }
    }
  ]
}
```

##### 1mcupdate.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerService/managedClusters",
      "apiVersion": "2023-07-01",
      "name": "akstestcluster",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayname": "Demo of AKS Nodepool Migration"
      },
      "identity": {
        "type": "SystemAssigned"
      },
      "properties": {
        "enableRBAC": true,
        "dnsPrefix": "testcluster",
        "agentPoolProfiles": [
          {
            "name": "testnp",
            "osType": "Linux",
            "osSku": "AzureLinuxOSGuard",
            "mode": "System"
          }
        ]
      }
    }
  ]
} 
```

##### 2apupdate.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "apiVersion": "2023-07-01",
      "type": "Microsoft.ContainerService/managedClusters/agentPools",
      "name": "akstestcluster/testnp",
      "location": "[resourceGroup().location]",
      "properties": {
        "osType": "Linux",
        "osSku": "AzureLinux",
        "mode": "System"
      }
    }
  ]
}
```

#### Deploy a test cluster

1. Create a resource group for the test cluster using the [`az group create`](/cli/azure/group#az-group-create) command.

    ```azurecli-interactive
    az group create --name testRG --location eastus
    ```

1. Deploy a baseline Azure Linux OS SKU cluster with three nodes using the [`az deployment group create`](/cli/azure/deployment/group#az-deployment-group-create) command and the [0base.json example ARM template](#0basejson).

    ```azurecli-interactive
    az deployment group create --resource-group testRG --template-file 0base.json
    ```

1. Migrate the OS SKU of your system node pool to Azure Linux with OS Guard using the [`az deployment group create`](/cli/azure/deployment/group#az-deployment-group-create) command.

    ```azurecli-interactive
    az deployment group create --resource-group testRG --template-file 1mcupdate.json
    ```

1. Migrate the OS SKU of your system node pool back to Azure Linux using the [`az deployment group create`](/cli/azure/deployment/group#az-deployment-group-create) command.

    ```azurecli-interactive
    az deployment group create --resource-group testRG --template-file 2apupdate.json
    ```

---

### Verify the OS SKU migration

Once the migration is complete on your test clusters, you should verify the following to ensure a successful migration:

- If your migration target is Azure Linux with OS Guard, run the `kubectl get nodes -o wide` command. The output should show `Microsoft Azure Linux 3.0` as your OS image and `.azl3` at the end of your kernel version.
- Run the `kubectl get pods -o wide -A` command to verify that all of your pods and daemonsets are running on the new node pool.
- Run the `kubectl get nodes --show-labels` command to verify that all of the node labels in your upgraded node pool are what you expect.

> [!TIP]
> We recommend monitoring the health of your service for a couple weeks before migrating your production clusters.

### Rollback

If you experience issues during the OS SKU migration, you can roll back to your previous OS SKU. To do this, you need to change the OS SKU field in your template and resubmit the deployment, which triggers another upgrade operation and restores the node pool to its previous OS SKU.

 > [!NOTE]
 > OS SKU migration doesn't support rolling back to OS SKU Mariner.

- Roll back to your previous OS SKU using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command. This command updates the OS SKU for your node pool from Azure Linux with OS Guard back to Azure Linux.

## Next steps

In this tutorial, you migrated existing nodes to Azure Linux with OS Guard. In the next tutorial, you learn how to enable telemetry to monitor your clusters.

> [!div class="nextstepaction"]
> [Enable telemetry and monitoring](./tutorial-azure-linux-os-guard-telemetry-monitor.md)