---
title: "Tutorial: Upgrade Azure Linux Container Host Nodes"
description: In this Azure Linux tutorial, you learn how to upgrade Azure Linux Container Host nodes in an Azure Kubernetes Service (AKS) cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Upgrade Azure Linux Container Host nodes

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321846)

The Azure Linux Container Host ships updates through two mechanisms: updated Azure Linux node images and automatic package updates.

As part of the application and cluster lifecycle, we recommend keeping your clusters up to date and secured by enabling upgrades for your cluster. You can enable automatic node image upgrades to ensure your clusters use the latest Azure Linux Container Host image when it scales up. You can also manually upgrade the node image on a cluster.

In this tutorial, part _five of five_, you learn how to:

> [!div class="checklist"]
>
> - Manually upgrade the node image on a cluster.
> - Automatically upgrade an Azure Linux Container Host cluster.
> - Deploy Kured in an Azure Linux Container Host cluster.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md) and [Tutorial 2: Add an Azure Linux node pool to your existing AKS cluster](./tutorial-add-azure-linux-node-pool.md).

> [!NOTE]
> Any upgrade operation, whether performed manually or automatically, upgrades the node image version if not already on the latest. The latest version is contingent on a full AKS release, and can be determined by visiting the [AKS release tracker](/azure/aks/release-tracker).

## Prerequisites

- In previous tutorials, you created and deployed an Azure Linux Container Host cluster. To complete this tutorial, you need an existing cluster. If you haven't done this step and would like to follow along, start with [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).
- You need the latest version of Azure CLI. Find the version using the `az --version` command. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Manually upgrade your cluster

Manually upgrade the node image on a cluster using the [`az aks nodepool upgrade`](/cli/azure/aks/nodepool#az-aks-nodepool-upgrade) command.

```azurecli-interactive
az aks nodepool upgrade --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --name $NODE_POOL_NAME
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
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxResourceGroup",
  "location": "WestUS2",
  "name": "testAzureLinuxCluster",
  "properties": {
    "autoUpgradeChannel": "stable",
    "provisioningState": "Succeeded"
  }
}
```

For more information on upgrade channels, see [Using cluster auto-upgrade](/azure/aks/auto-upgrade-cluster).

## Enable automatic package upgrades

You can also enable automatic package upgrades. If automatic package upgrades are enabled, the dnf-automatic systemd service runs daily and installs any updated packages that have been published.

Set the node OS upgrade channel on an existing cluster using the [`az aks update`](/cli/azure/aks#az_aks_update) command with the `--node-os-upgrade-channel` parameter. The following example sets the node OS upgrade channel to `Unmanaged` for an existing cluster:

```azurecli-interactive
az aks update --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --node-os-upgrade-channel Unmanaged
```

Example output:

<!-- expected_similarity=0.3 -->
```json
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxResourceGroup",
  "location": "WestUS2",
  "name": "testAzureLinuxCluster",
  "properties": {
    "nodeOsUpgradeChannel": "Unmanaged",
    "provisioningState": "Succeeded"
  }
}
```

## Enable an automatic reboot daemon

To protect your clusters, security updates are automatically applied to Azure Linux nodes. These updates include OS security fixes, kernel updates, and package upgrades. Some of these updates require a node reboot to complete the process. AKS doesn't automatically reboot these nodes to complete the update process.

We recommend enabling an automatic reboot daemon, such as [Kured](https://kured.dev/docs/), so that your cluster can reboot nodes that have taken kernel updates. To deploy the Kured DaemonSet in an Azure Linux Container Host cluster, see [Deploy Kured in an AKS cluster](/azure/aks/node-updates-kured#deploy-kured-in-an-aks-cluster).

## Clean up resources

As this tutorial is the last part of the series, you might want to delete your Azure Linux Container Host cluster. The Kubernetes nodes run on Azure virtual machines (VMs) and continue incurring charges even if you don't use the cluster.

Delete the Azure resource group and all related resources using the [`az group delete`](/cli/azure/group#az_group_delete) command.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

## Next steps

In this tutorial, you upgraded your Azure Linux Container Host cluster.

For more information on the Azure Linux Container Host, see the [Azure Linux Container Host overview](./azure-linux-aks-overview.md).
