---
title: "Quickstart: Deploy Azure Container Linux (ACL) for AKS"
description: Learn how to quickly deploy Azure Container Linux (ACL) for Azure Kubernetes Service (AKS) and connect to the cluster using `kubectl`.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: quickstart
ms.date: 04/26/2026
---

# Quickstart: Deploy Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)

Azure Container Linux (ACL) for Azure Kubernetes Service (AKS) is an immutable, container-optimized operating system for AKS node pools. For more information, see [Azure Container Linux for AKS](./azure-container-linux-overview.md).

In this quickstart, you learn how to:

> [!div class="checklist"]
>
> - Install the Azure CLI and `kubectl`.
> - Create an Azure resource group.
> - Deploy an AKS cluster that uses Azure Container Linux.
> - Connect to the cluster and verify the nodes.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/).
- The latest version of Azure CLI. To install or upgrade Azure CLI, see [Install Azure CLI](/cli/azure/install-azure-cli).
- The Kubernetes CLI, `kubectl`. To install it with Azure CLI, use the [`az aks install-cli`](/cli/azure/aks#az_aks_install_cli) command:

    ```azurecli-interactive
    az aks install-cli
    ```

- Permissions to create resource groups and AKS clusters in your Azure subscription.

## Set environment variables

Set the following environment variables to define the resource group name, cluster name, and location for your deployment. You can use the same values as shown here or replace them with names that are unique in your environment.

```bash
export RESOURCE_GROUP="acl-aks-rg"
export CLUSTER_NAME="acl-aks-cluster"
export LOCATION="eastus"
```

## Create a resource group

Create an [Azure resource group](/azure/azure-resource-manager/management/overview) for the AKS cluster using the [`az group create`](/cli/azure/group#az_group_create) command.

```azurecli-interactive
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
```

## Create an AKS cluster that uses Azure Container Linux

Create an AKS cluster that uses ACL using the [`az aks create`](/cli/azure/aks#az_aks_create) command with the `--os-sku` parameter set to `AzureContainerLinux`.

```azurecli-interactive
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --node-count 3 \
  --generate-ssh-keys \
  --os-sku AzureContainerLinux
```

The deployment takes a few minutes to complete.

## Connect to the cluster

1. After the cluster is deployed, configure `kubectl` to connect to it by retrieving the cluster credentials using the [`az aks get-credentials`](/cli/azure/aks#az_aks_get_credentials) command.

    ```azurecli-interactive
    az aks get-credentials \
      --resource-group $RESOURCE_GROUP \
      --name $CLUSTER_NAME
    ```

1. Verify the nodes are ready using the [`kubectl get nodes`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes.

    ```bash
    kubectl get nodes
    ```

    Example output:

    ```output
    NAME                                STATUS   ROLES   AGE   VERSION
    aks-nodepool1-12345678-vmss000000   Ready    agent   5m    v1.34.4
    aks-nodepool1-12345678-vmss000001   Ready    agent   5m    v1.34.4
    aks-nodepool1-12345678-vmss000002   Ready    agent   5m    v1.34.4
    ```

## Clean up resources

If you don't plan to continue using this cluster, delete the resource group to avoid ongoing charges using the [`az group delete`](/cli/azure/group#az_group_delete) command. Deleting the resource group removes the AKS cluster and all associated resources.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

## Related content

To learn more about Azure Container Linux (ACL) and how it integrates with AKS, see the following resources:

- [Azure Container Linux for AKS](./azure-container-linux-overview.md)
- [Migrate existing nodes to Azure Container Linux (ACL) for AKS](./tutorial-migrate-acl-aks.md)
