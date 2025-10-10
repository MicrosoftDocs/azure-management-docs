---
title: Azure Linux with OS Guard for AKS tutorial - Upgrade Azure Linux with OS Guard nodes
description: In this Azure Linux with OS Guard for AKS tutorial, you learn how to upgrade Azure Linux with OS Guard nodes.
author: florataagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 10/06/2025
# Customer intent: "As a cloud administrator, I want to upgrade the Azure Linux with OS Guard nodes in my AKS cluster, so that I can ensure the environment is secure and up-to-date with the latest features and security patches."
---

# Tutorial: Upgrade Azure Linux with OS Guard nodes

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321846)

Azure Linux with OS Guard ships updates through two mechanisms: updated node images and automatic package updates.

As part of the application and cluster lifecycle, we recommend keeping your clusters up to date and secured by enabling upgrades for your cluster. You can enable automatic node-image upgrades to ensure your clusters use the latest Azure Linux with OS Guard image when it scales up. You can also manually upgrade the node-image on a cluster.

In this tutorial, part five of five, you learn how to:

> [!div class="checklist"]
>
> * Manually upgrade the node-image on a cluster.
> * Automatically upgrade an Azure Linux with OS Guard cluster.
> * Deploy Kured in an Azure Linux with OS Guard cluster. 

> [!NOTE]
> Any upgrade operation, whether performed manually or automatically, upgrades the node image version if not already on the latest. The latest version is contingent on a full AKS release, and can be determined by visiting the [AKS release tracker](/azure/aks/release-tracker).

## Prerequisites

* In previous tutorials, you created and deployed an Azure Linux with OS Guard cluster. To complete this tutorial, you need an existing cluster. If you haven't done this step and would like to follow along, start with [Tutorial 1: Create a cluster with Azure Linux with OS Guard for AKS](./tutorial-azure-linux-os-guard-create-cluster.md).
* You need the latest version of Azure CLI. Find the version using the `az --version` command. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Manually upgrade your cluster

To manually upgrade the node-image on a cluster, run the following command for your OS Guard node pool:

```azurecli
az aks nodepool upgrade --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODEPOOL_NAME
```

## Automatically upgrade your cluster

Auto-upgrade provides a set once and forget mechanism that yields tangible time and operational cost benefits. By enabling auto-upgrade, you can ensure your clusters are up to date and don't miss the latest Azure Linux with OS Guard features or patches from AKS and upstream Kubernetes.

Automatically completed upgrades are functionally the same as manual upgrades. The selected channel determines the timing of upgrades. When making changes to auto-upgrade, allow 24 hours for the changes to take effect.

To set the auto-upgrade channel on an existing cluster, update the --auto-upgrade-channel parameter:

```bash
az aks update --resource-group $AZ_LINUX_RG --name $AZ_LINUX_CLUSTER --auto-upgrade-channel stable
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

You can also configure automatic upgrades for package upgrades by enabling the node OS upgrade channel. If automatic package upgrades are enabled, the `dnf-automatic systemd` service runs daily and installs any updated packages that have been published.

Set the node OS upgrade channel on an existing cluster using the [`az aks update`](/cli/azure/aks#az-aks-update) command with the `--node-os-upgrade-channel` parameter.

```azurecli-interactive
az aks update --resource-group $AZ_LINUX_RG --name $AZ_LINUX_CLUSTER --node-os-upgrade-channel Unmanaged
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

## Clean up resources

As this tutorial is the last part of the series, you may want to delete your Azure Linux Container Host cluster. The Kubernetes nodes run on Azure virtual machines and continue incurring charges even if you don't use the cluster. 

## Next steps

In this tutorial, you upgraded your Azure Linux Container Host cluster.

For more information on Azure Linux with OS Guard, see the [Azure Linux with OS Guard overview](./intro-azure-linux-os-guard.md).