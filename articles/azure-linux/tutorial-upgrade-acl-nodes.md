---
title: "Tutorial: Upgrade Azure Container Linux (ACL) Nodes"
description: In this Azure Linux tutorial, you learn how to upgrade Azure Container Linux (ACL) nodes in an Azure Kubernetes Service (AKS) cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Upgrade Azure Container Linux (ACL) nodes

ACL delivers updates through weekly node images that include the latest security patches and bug fixes. As part of the application and cluster lifecycle, we recommend keeping your clusters up to date and secured by enabling upgrades for your cluster. You can enable automatic node image upgrades to ensure your clusters use the latest ACL image when it scales up. You can also manually upgrade the node image on a cluster.

In this tutorial, part _five of five_, you learn how to:

> [!div class="checklist"]
>
> - Manually upgrade the node image on a cluster.
> - Automatically upgrade an ACL cluster.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).

> [!NOTE]
> Any upgrade operation, whether performed manually or automatically, upgrades the node image version if it's not already on the latest version. The latest version is contingent on a full AKS release, and you can determine it by visiting the [AKS release tracker](/azure/aks/release-tracker).

> [!IMPORTANT]
> `NodeImage` and `None` are the only supported [operating system (OS) upgrade channels](/azure/aks/auto-upgrade-node-os-image) for ACL. `Unmanaged` and `SecurityPatch` are incompatible with ACL due to the immutable `/usr` directory. Automatic A/B updates for the OS partition aren't yet available.

## Prerequisites

- In previous tutorials, you created and deployed an ACL cluster. To complete this tutorial, you need an existing cluster. If you haven't completed this step and want to follow along, see [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).
- Azure Container Linux requires Azure CLI version 2.86.0 or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

[!INCLUDE [azure container linux limitations](./includes/acl-limitations.md)]

## Manually upgrade your cluster

Manually upgrade the node image on your cluster using the [`az aks nodepool upgrade`](/cli/azure/aks/nodepool#az-aks-nodepool-upgrade) command.

```azurecli-interactive
az aks nodepool upgrade --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name <node-pool-name>
```

## Automatically upgrade your cluster

Automatic upgrades are functionally the same as manual upgrades. The selected channel determines the timing of upgrades. When making changes to auto-upgrade, allow 24 hours for the changes to take effect.

Set the auto-upgrade channel on an existing cluster using the [`az aks update`](/cli/azure/aks#az_aks_update) command with the `--auto-upgrade-channel` parameter. The following example sets the auto-upgrade channel to `stable` for an existing cluster:

```azurecli-interactive
az aks update --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --auto-upgrade-channel stable
```

Example output:

<!-- expected_similarity=0.3 -->
```json
{
  "id": "/subscriptions/xxxxx/resourceGroups/myACLResourceGroup",
  "location": "westus",
  "name": "myACLCluster",
  "properties": {
    "autoUpgradeChannel": "stable",
    "provisioningState": "Succeeded"
  }
}
```

For more information on upgrade channels, see [Using cluster auto-upgrade](/azure/aks/auto-upgrade-cluster).

## Automatically upgrade your node OS image

AKS provides multiple auto-upgrade channels dedicated to timely node-level OS security updates. This channel is different from cluster-level Kubernetes version upgrades and supersedes it.

Set the node OS upgrade channel on an existing cluster using the [`az aks update`](/cli/azure/aks#az-aks-update) command with the `--node-os-upgrade-channel` parameter. The following example sets the node OS upgrade channel to `NodeImage` for an existing cluster:

```azurecli-interactive
az aks update --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --node-os-upgrade-channel NodeImage
```

Example output:

<!-- expected_similarity=0.3 -->
```json
{
  "id": "/subscriptions/xxxxx/resourceGroups/myACLResourceGroup",
  "location": "westus",
  "name": "myACLCluster",
  "properties": {
    "nodeOsUpgradeChannel": "NodeImage",
    "provisioningState": "Succeeded"
  }
}
```

For more information on node upgrade channels, see [Using node OS auto-upgrade](/azure/aks/auto-upgrade-node-os-image).

## ACL versioning

ACL releases weekly AKS node images. Versioning follows the AKS date-based format (for example: `202506.13.0`). You can check available node images in the release notes and view the `nodeImageVersion` for a running cluster using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az-aks-nodepool-list) command.

```azurecli-interactive
az aks nodepool list \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --query '[].{name: name, nodeImageVersion: nodeImageVersion}'
```

Example output:

```json
[
  {
    "name": "nodepool1",
    "nodeImageVersion": "AKSAzureContainerLinux-202606.01.0"
  }
]
```

## Clean up resources

As this tutorial is the last part of the series, you might want to delete your ACL cluster. The Kubernetes nodes run on Azure virtual machines (VMs) and continue incurring charges even if you don't use the cluster.

Delete the Azure resource group and all related resources using the [`az group delete`](/cli/azure/group#az_group_delete) command.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

## Next steps

In this tutorial, you upgraded your ACL cluster.

For more information on Azure Container Linux, see the [ACL overview](./azure-container-linux-overview.md).
