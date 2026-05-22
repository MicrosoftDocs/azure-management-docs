---
title: "Quickstart: Create a Cluster with the Azure Linux Container Host for AKS"
description: Learn how to quickly create a cluster with the Azure Linux Container Host for AKS and connect to it using `kubectl`.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: quickstart
ms.date: 04/26/2026
---

# Quickstart: Create a cluster with the Azure Linux Container Host for Azure Kubernetes Service (AKS)

The Azure Linux container host for AKS delivers fast boot times, predictable updates, and strong security defaults with tight AKS lifecycle integration. Microsoft owns the full stack from kernel to CVE response, simplifying operations and reducing customer overhead. For more information, see [Azure Linux for AKS](./azure-linux-aks-overview.md).

In this quickstart, you learn how to:

> [!div class="checklist"]
>
> - Install the Kubernetes CLI, `kubectl`.
> - Create an Azure resource group.
> - Create and deploy an Azure Linux Container Host cluster.
> - Configure `kubectl` to connect to your Azure Linux Container Host cluster.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Set environment variables

Set the following environment variables to ensure consistent naming across your deployment. In this quickstart, we set the resource group name to _testAzureLinuxResourceGroup_, the cluster name to _testAzureLinuxCluster_, and the location to _EastUS2_.

```azurecli-interactive
export RESOURCE_GROUP=testAzureLinuxResourceGroup
export CLUSTER_NAME=testAzureLinuxCluster
export LOCATION=EastUS2
```

## Create a resource group

To create a cluster with the Azure Linux Container Host, you use:

- An [Azure resource group](/azure/azure-resource-manager/management/overview): A logical container into which Azure resources are deployed and managed.
- [AKS](/azure/aks/intro-kubernetes): A hosted Kubernetes service that allows you to quickly create a production ready Kubernetes cluster.

When creating a resource group, you're required to specify a location. This location is:

- The storage location of your resource group metadata.
- Where your resources run in Azure if you don't specify another region when creating a resource.

Create the resource group using the [`az group create`](/cli/azure/group#az_group_create) command with the resource group name and region environment variables you set earlier.

```azurecli-interactive
az group create --name $RESOURCE_GROUP --location $LOCATION
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxResourceGroup",
  "location": "EastUS2",
  "managedBy": null,
  "name": "testAzureLinuxResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create an Azure Linux Container Host cluster

Create an AKS cluster using the [`az aks create`](/cli/azure/aks#az_aks_create) command with the `--os-sku` parameter to provision the Azure Linux Container Host with an Azure Linux image. The following example creates an Azure Linux Container Host cluster named _testAzureLinuxCluster_ in the resource group _testAzureLinuxResourceGroup_:

```azurecli-interactive
az aks create --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --os-sku AzureLinux
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxResourceGroup/providers/Microsoft.ContainerService/managedClusters/testAzureLinuxCluster",
  "location": "EastUS2",
  "name": "testAzureLinuxCluster",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.ContainerService/managedClusters"
}
```

After a few minutes, the command completes and returns JSON-formatted information about the cluster.

## Connect to the cluster using kubectl

1. Configure `kubectl` to connect to your Kubernetes cluster using the [`az aks get-credentials`](/cli/azure/aks#az_aks_get_credentials) command. The following example gets credentials for the Azure Linux Container Host cluster using the resource group and cluster name created earlier:

    ```azurecli-interactive
    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
    ```

1. Verify the connection to your cluster using the [`kubectl get nodes`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes:

    ```bash
    kubectl get nodes
    ```

    Example output:

    <!-- expected_similarity=0.3 -->
    ```output
    NAME                           STATUS   ROLES   AGE     VERSION
    aks-nodepool1-00000000-0       Ready    agent   10m     v1.34.4
    aks-nodepool1-00000000-1       Ready    agent   10m     v1.34.4
    ```

    All nodes show a `Ready` status, confirming your Azure Linux Container Host cluster is running and reachable.

## Clean up resources

If you don't plan to continue using this cluster, delete the resource group to avoid ongoing charges using the [`az group delete`](/cli/azure/group#az_group_delete) command. Deleting the resource group removes the AKS cluster and all associated resources.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

## Related content

Now that you have a running Azure Linux Container Host cluster, you can:

- [Deploy an application to your AKS cluster](/azure/aks/learn/quick-kubernetes-deploy-cli#deploy-the-application)
- [Learn about Azure Linux Container Host concepts](./aks-core-concepts.md)
- [Explore Azure Container Linux (ACL)](./azure-container-linux-overview.md)
