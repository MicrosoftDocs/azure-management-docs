---
title: "Tutorial: Create a Cluster with Azure Linux with OS Guard (preview) for Azure Kubernetes Service (AKS)"
description: In this Azure Linux tutorial, you learn how to create a cluster with Azure Linux with OS Guard (preview) for Azure Kubernetes Service (AKS).
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Create a cluster with the Azure Linux with OS Guard (preview) for Azure Kubernetes Service (AKS)

[!INCLUDE [os-guard replacement](./includes/os-guard-replacement.md)]

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321737)

In this tutorial, part _one of five_, you learn how to:

> [!div class="checklist"]
>
> - Install the Kubernetes CLI, `kubectl`.
> - Install the `aks-preview` Azure CLI extension.
> - Register the `AzureLinuxOSGuardPreview` feature flag.
> - Create an Azure resource group.
> - Create and deploy an Azure Linux with OS Guard cluster.
> - Configure `kubectl` to connect to your Azure Linux with OS Guard cluster.

In later tutorials, you learn how to add an Azure Linux with OS Guard node pool to an existing cluster and migrate existing nodes to Azure Linux with OS Guard.

## Prerequisites

- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.
- [Install the `aks-preview` Azure CLI extension](#install-the-aks-preview-azure-cli-extension).
- [Register the `AzureLinuxOSGuardPreview` feature flag](#register-the-azurelinuxosguardpreview-feature-flag).

[!INCLUDE [azure linux with os guard limitations](./includes/os-guard-limitations.md)]

### Install the `aks-preview` Azure CLI extension

[!INCLUDE [preview features callout](~/reusable-content/ce-skilling/azure/includes/aks/includes/preview/preview-callout.md)]

Install the `aks-preview` extension using the [`az extension add`](/cli/azure/extension#az-extension-add) command.

```azurecli-interactive
az extension add --name aks-preview
```

Update to the latest version of the extension using the [`az extension update`](/cli/azure/extension#az-extension-update) command.

```azurecli-interactive
az extension update --name aks-preview
```

### Register the `AzureLinuxOSGuardPreview` feature flag

1. Register the `AzureLinuxOSGuardPreview` feature flag using the [`az feature register`](/cli/azure/feature#az-feature-register) command.

    ```azurecli-interactive
    az feature register --namespace "Microsoft.ContainerService" --name "AzureLinuxOSGuardPreview"
    ```

    It takes a few minutes for the status to show _Registered_.

1. Verify the registration status using the [`az feature show`](/cli/azure/feature#az-feature-show) command.

    ```azurecli-interactive
    az feature show --namespace "Microsoft.ContainerService" --name "AzureLinuxOSGuardPreview"
    ``

1. When the status reflects _Registered_, refresh the registration of the `Microsoft.ContainerService` resource provider using the [`az provider register`](/cli/azure/provider#az-provider-register) command.

    ```azurecli-interactive
    az provider register --namespace "Microsoft.ContainerService"
    ```

## Set environment variables

Set the following environment variables to create unique resource names for each deployment:

```bash
export RESOURCE_GROUP="<your-resource-group-name>"
export REGION="<your-region>"
export CLUSTER_NAME="<your-cluster-name>"
```

## Create a resource group

When creating a resource group in Azure, you're required to specify a location. This location is the storage location of your resource group metadata and where your resources run in Azure if you don't specify another region when creating a resource.

Create a resource group using the [`az group create`](/cli/azure/group#az_group_create) command.

```azurecli-interactive
az group create --name $RESOURCE_GROUP --location $REGION
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxOSGuardResourceGroupxxxxx",
  "location": "EastUS2",
  "managedBy": null,
  "name": "testAzureLinuxOSGuardResourceGroupxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create an Azure Linux with OS Guard (preview) cluster

Create an AKS cluster using the [`az aks create`](/cli/azure/aks#az-aks-create) command with the `--os-sku AzureLinuxOSGuard` parameter to provision an Azure Linux with OS Guard cluster. Enabling [FIPS](/azure/aks/enable-fips-nodes), [secure boot](/azure/aks/use-trusted-launch), and [vtpm](/azure/aks/use-trusted-launch) is required to use Azure Linux with OS Guard.

```azurecli-interactive
az aks create --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --os-sku AzureLinuxOSGuard --node-osdisk-type Managed --enable-fips-image --enable-secure-boot --enable-vtpm
```

Example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/testAzureLinuxOSGuardResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/testAzureLinuxOSGuardClusterxxxxx",
  "location": "WestUS2",
  "name": "testAzureLinuxOSGuardClusterxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.ContainerService/managedClusters"
}
```

After a few minutes, the command completes and returns JSON-formatted information about the cluster.

## Connect to the cluster using kubectl

1. Configure `kubectl` to connect to your Kubernetes cluster using the [`az aks get-credentials`](/cli/azure/aks#az-aks-get-credentials) command.

    ```azurecli-interactive
    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
    ```

1. Verify the connection to your cluster using the [`kubectl get nodes`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes.

    ```bash
    kubectl get nodes
    ```

    Example output:

    <!-- expected_similarity=0.3 -->
    ```text
    NAME                           STATUS   ROLES   AGE     VERSION
    aks-nodepool1-00000000-0       Ready    agent   10m     v1.20.7
    aks-nodepool1-00000000-1       Ready    agent   10m     v1.20.7
    ```

## Next step

In this tutorial, you created and deployed an Azure Linux with OS Guard cluster. In the next tutorial, you learn how to add an Azure Linux with OS Guard node pool to an existing cluster.

> [!div class="nextstepaction"]
> [Add an Azure Linux with OS Guard node pool](./tutorial-add-os-guard-node-pool.md)
