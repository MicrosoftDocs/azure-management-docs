---
title: Azure Linux Container Host for AKS tutorial - Migrating to Azure Linux
description: In this Azure Linux Container Host for AKS tutorial, you learn how to migrate your nodes to Azure Linux nodes.
author: suhuruli
ms.author: suhuruli
ms.reviewer: schaffererin
ms.service: microsoft-linux
ms.custom: devx-track-azurecli, linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 04/06/2025
---

# Tutorial: Migrate nodes to Azure Linux

In this tutorial, part three of five, you migrate your existing nodes to Azure Linux. You can migrate your existing nodes to Azure Linux using one of the following methods:

* Remove existing node pools and add new Azure Linux node pools.
* In-place OS SKU migration.

If you don't have any existing nodes to migrate to Azure Linux, skip to the [next tutorial](./tutorial-azure-linux-telemetry-monitor.md). In later tutorials, you learn how to enable telemetry and monitoring in your clusters and upgrade Azure Linux nodes.

## Prerequisites

* In previous tutorials, you created and deployed an Azure Linux Container Host for AKS cluster. To complete this tutorial, you need to add an Azure Linux node pool to your existing cluster. If you haven't done this step and would like to follow along, start with [Tutorial 2: Add an Azure Linux node pool to your existing AKS cluster](./tutorial-azure-linux-add-nodepool.md).

    > [!NOTE]
    > When adding a new Azure Linux node pool, you need to add at least one as `--mode System`. Otherwise, AKS won't allow you to delete your existing node pool.

* You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Add Azure Linux node pools and remove existing node pools

1. Add a new Azure Linux node pool using the `az aks nodepool add` command. This command adds a new node pool to your cluster with the `--mode System` flag, which makes it a system node pool. System node pools are required for Azure Linux clusters.

```azurecli-interactive
# Declare environment variables with a random suffix for uniqueness
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export NODE_POOL_NAME="np$RANDOM_SUFFIX"
az aks nodepool add --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODE_POOL_NAME --mode System --os-sku AzureLinux
```

Results:

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

You can now migrate your existing Ubuntu node pools to Azure Linux by changing the OS SKU of the node pool, which rolls the cluster through the standard node image upgrade process. This new feature doesn't require the creation of new node pools.

### Limitations

There are several settings that can block the OS SKU migration request. To ensure a successful migration, review the following guidelines and limitations:

* The OS SKU migration feature isn't available through PowerShell or the Azure portal.
* The OS SKU migration feature isn't able to rename existing node pools.
* Ubuntu and Azure Linux are the only supported Linux OS SKU migration targets.
* An Ubuntu OS SKU with `UseGPUDedicatedVHD` enabled can't perform an OS SKU migration.
* An Ubuntu OS SKU with CVM 20.04 enabled can't perform an OS SKU migration.
* Node pools with Kata enabled can't perform an OS SKU migration.
* Windows OS SKU migration isn't supported.
* OS SKU migration from Mariner to Azure Linux is supported, but rolling back to Mariner is not supported.

### Prerequisites

* An existing AKS cluster with at least one Ubuntu node pool.
* We recommend that you ensure your workloads configure and run successfully on the Azure Linux container host before attempting to use the OS SKU migration feature by [deploying an Azure Linux cluster](./quickstart-azure-cli.md) in dev/prod and verifying your service remains healthy.
* Ensure the migration feature is working for you in test/dev before using the process on a production cluster.
* Ensure that your pods have enough [Pod Disruption Budget](/azure/aks/operator-best-practices-scheduler#plan-for-availability-using-pod-disruption-budgets) to allow AKS to move pods between VMs during the upgrade.
* You need Azure CLI version [2.61.0](/cli/azure/release-notes-azure-cli#may-21-2024) or higher. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).
* If you are using Terraform, you must have [v3.111.0](https://github.com/hashicorp/terraform-provider-azurerm/releases/tag/v3.111.0) or greater of the AzureRM Terraform module.

### [Azure CLI](#tab/azure-cli)

#### Migrate the OS SKU of your Ubuntu node pool

* Migrate the OS SKU of your node pool to Azure Linux using the `az aks nodepool update` command. This command updates the OS SKU for your node pool from Ubuntu to Azure Linux. The OS SKU change triggers an immediate upgrade operation, which takes several minutes to complete.

```azurecli-interactive
az aks nodepool update --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODE_POOL_NAME --os-sku AzureLinux
```

Results:

<!-- expected_similarity=0.3 -->

```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxx/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/nodePools/nodepool1",
  "name": "nodepool1",
  "osSku": "AzureLinux",
  "provisioningState": "Succeeded"
}
```

> [!NOTE]
> If you experience issues during the OS SKU migration, you can [roll back to your previous OS SKU](#rollback).

### Verify the OS SKU migration

Once the migration is complete on your test clusters, you should verify the following to ensure a successful migration:

* If your migration target is Azure Linux, run the `kubectl get nodes -o wide` command. The output should show `CBL-Mariner/Linux` as your OS image and `.cm2` at the end of your kernel version.
* Run the `kubectl get pods -o wide -A` command to verify that all of your pods and daemonsets are running on the new node pool.
* Run the `kubectl get nodes --show-labels` command to verify that all of the node labels in your upgraded node pool are what you expect.

> [!TIP]
> We recommend monitoring the health of your service for a couple weeks before migrating your production clusters.

### Run the OS SKU migration on your production clusters

1. Update your existing templates to set `OSSKU=AzureLinux`. In ARM templates, you use `"OSSKU": "AzureLinux"` in the `agentPoolProfile` section. In Bicep, you use `osSku: "AzureLinux"` in the `agentPoolProfile` section. Lastly, for Terraform, you use `os_sku = "AzureLinux"` in the `default_node_pool` section. Make sure that your `apiVersion` is set to `2023-07-01` or later.
2. Redeploy your ARM, Bicep, or Terraform template for the cluster to apply the new `OSSKU` setting. During this deploy, your cluster behaves as if it's taking a node image upgrade. Your cluster surges capacity, and then reboots your existing nodes one by one into the latest AKS image from your new OS SKU.

### Rollback

If you experience issues during the OS SKU migration, you can roll back to your previous OS SKU. To do this, you need to change the OS SKU field in your template and resubmit the deployment, which triggers another upgrade operation and restores the node pool to its previous OS SKU.

 > [!NOTE]
 > 
 > OS SKU migration does not support rolling back to OS SKU Mariner.

* Roll back to your previous OS SKU using the `az aks nodepool update` command. This command updates the OS SKU for your node pool from Azure Linux back to Ubuntu.

## Next steps

In this tutorial, you migrated existing nodes to Azure Linux using one of the following methods:

* Remove existing node pools and add new Azure Linux node pools.
* In-place OS SKU migration.

In the next tutorial, you learn how to enable telemetry to monitor your clusters.

> [!div class="nextstepaction"]
> [Enable telemetry and monitoring](./tutorial-azure-linux-telemetry-monitor.md)