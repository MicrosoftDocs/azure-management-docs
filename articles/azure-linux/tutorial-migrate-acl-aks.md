---
title: "Tutorial: Migrate Nodes to Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)"
description: In this Azure Linux tutorial, you learn how to migrate nodes to Azure Container Linux (ACL) for AKS.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Migrate nodes to Azure Container Linux (ACL)

In this tutorial, part _three of five_, you migrate your existing nodes to ACL. You can migrate your existing nodes using one of the following methods:

- Remove existing node pools and add new ACL node pools.
- Perform an in-place operating system (OS) SKU migration.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).

If you don't have any existing nodes to migrate, skip to the [next tutorial](./tutorial-monitor-acl-aks.md). In later tutorials, you learn how to enable telemetry and monitoring in your clusters and upgrade ACL nodes.

## Prerequisites

- In previous tutorials, you created and deployed an ACL cluster. If you haven't completed these steps and want to follow along, see [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).
- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

[!INCLUDE [azure container linux limitations](./includes/acl-limitations.md)]

## Add ACL node pools and remove existing node pools

1. Add a new ACL node pool using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command. Use `--mode System` so the new pool can serve as the system agent pool, which allows you to delete the original node pool in the next step. The following example creates a node pool named _aclsystem_ that adds three nodes to the cluster:

    ```azurecli-interactive
    az aks nodepool add \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name aclsystem \
        --mode System \
        --os-sku AzureContainerLinux \
        --node-count 3
    ```

    Example output:

    <!-- expected_similarity=0.3 -->

    ```JSON
    {
      "id": "/subscriptions/xxxxx/resourceGroups/myACLResourceGroup/providers/Microsoft.ContainerService/managedClusters/myACLCluster/nodePools/aclsystem",
      "name": "aclsystem",
      "osSku": "AzureContainerLinux",
      "provisioningState": "Succeeded"
    }
    ```

1. Remove your existing node pool using the [`az aks nodepool delete`](/cli/azure/aks/nodepool#az-aks-nodepool-delete) command.

    ```azurecli-interactive
    az aks nodepool delete \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name <existing-node-pool-name>
    ```

## In-place OS SKU migration

### Limitations for in-place OS SKU migration

There are several settings that can block the OS SKU migration request. To ensure a successful migration, review the following guidelines and limitations:

- The OS SKU migration feature isn't available through PowerShell or the Azure portal.
- The OS SKU migration feature doesn't support renaming existing node pools.
- Ubuntu, Azure Linux, and AzureContainerLinux are the only supported Linux OS SKU migration targets.
- ACL requires Trusted Launch. If not already enabled on your node pool, you must include `--enable-secure-boot` and `--enable-vtpm` when migrating to the `AzureContainerLinux` OS SKU. Your node pool's VM size must also support Trusted Launch. If your current VM size doesn't support it, you need to resize or recreate the node pool with a supported VM size before migrating.
- [Generation 1 VMs](/azure/aks/aks-virtual-machine-sizes#vm-support-on-aks) aren't supported.
- An Ubuntu OS SKU with `UseGPUDedicatedVHD` enabled can't perform an OS SKU migration.
- [Confidential Virtual Machines (CVMs)](/azure/aks/confidential-containers-overview) aren't supported.
- [Pod Sandboxing](/azure/aks/use-pod-sandboxing) isn't supported.
- Windows OS SKU migration isn't supported.

### Prerequisites for in-place OS SKU migration

- An existing AKS cluster with at least one Linux node pool.
- We recommend that you verify your workloads run successfully on ACL by [deploying an ACL cluster](./quick-deploy-acl-aks-cli.md) in a development or staging environment before migrating production clusters.
- Ensure the migration feature is working for you in test/dev before using the process on a production cluster.
- Ensure that your pods have enough [Pod Disruption Budget (PDB)](/azure/aks/operator-best-practices-scheduler#plan-for-availability-using-pod-disruption-budgets) to allow AKS to move pods between VMs during the upgrade.
- You need Azure CLI version [2.61.0](/cli/azure/release-notes-azure-cli#may-21-2024) or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

### Migrate to ACL using in-place OS SKU migration

You can migrate your existing Ubuntu or Azure Linux node pools to ACL by changing the OS SKU of the node pool, which rolls the cluster through the standard node image upgrade process. This method doesn't require creating new node pools; instead, your existing node pools are automatically reimaged.

> [!IMPORTANT]
> ACL requires Trusted Launch. You must include `--enable-secure-boot` and `--enable-vtpm` when migrating to the `AzureContainerLinux` OS SKU. Your node pool's VM size must also support Trusted Launch.

Migrate the OS SKU of your node pool to ACL using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command. This command triggers a reimage of your node pool. The OS SKU change triggers an immediate upgrade operation, which takes several minutes to complete.

```azurecli-interactive
az aks nodepool update \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name <existing-node-pool-name> \
    --os-sku AzureContainerLinux \
    --enable-secure-boot \
    --enable-vtpm
```

Example output:

<!-- expected_similarity=0.3 -->

```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myACLResourceGroup/providers/Microsoft.ContainerService/managedClusters/myACLCluster/nodePools/nodepool1",
  "name": "nodepool1",
  "osSku": "AzureContainerLinux",
  "provisioningState": "Succeeded"
}
```

> [!NOTE]
> If you experience issues during the OS SKU migration, you can [roll back to your previous OS SKU](#roll-back-to-your-previous-os-sku).

### Verify the OS SKU migration

Once the migration is complete on your test clusters, verify the following to ensure a successful migration:

1. Confirm the new nodes are running ACL using the following command:

    ```bash
    kubectl get nodes -o wide
    ```

1. Verify all of your pods and daemonsets are running on the new node pool using the following command:

    ```bash
    kubectl get pods -o wide -A
    ```

1. Verify all of the node labels in your upgraded node pool are what you expect using the following command:

    ```bash
    kubectl get nodes --show-labels
    ```

1. Check the node image version using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az-aks-nodepool-list) command.

    ```azurecli-interactive
    az aks nodepool list \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --query '[].{name: name, osSku: osSku, nodeImageVersion: nodeImageVersion}'
    ```

    Example output:

    ```json
    [
      {
        "name": "nodepool1",
        "nodeImageVersion": "AKSAzureContainerLinux-202606.01.0",
        "osSku": "AzureContainerLinux"
      }
    ]
    ```

> [!TIP]
> We recommend monitoring the health of your service for a couple of weeks before migrating your production clusters.

### Roll back to your previous OS SKU

If you experience issues during the OS SKU migration, you can roll back to your previous OS SKU. To do this, change the OS SKU field back to your previous value and resubmit the deployment, which triggers another upgrade operation and reimages the node pool to its previous OS SKU.

Roll back to your previous OS SKU using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command. This example rolls back from ACL to Azure Linux:

```azurecli-interactive
az aks nodepool update \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name <existing-node-pool-name> \
    --os-sku AzureLinux
```

## Next step

In this tutorial, you migrated existing nodes to ACL. In the next tutorial, you learn how to enable telemetry and monitoring for your ACL cluster.

> [!div class="nextstepaction"]
> [Enable telemetry and monitoring](./tutorial-monitor-acl-aks.md)
