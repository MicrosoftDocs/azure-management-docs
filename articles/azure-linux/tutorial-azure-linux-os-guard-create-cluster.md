---
title: Azure Linux with OS Guard for Azure Kubernetes Service (AKS) Tutorial - Create a Cluster
description: In this tutorial, you learn how to create an AKS cluster with Azure Linux with OS Guard.
author: florataagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 09/24/2025
# Customer intent: "As a cloud engineer, I want to create an Azure Kubernetes Service (AKS) cluster with Azure Linux with OS Guard, so that I can deploy and manage containerized applications effectively in a production-ready environment."
---

# Tutorial: Create a cluster with the Azure Linux with OS Guard for Azure Kubernetes Service (AKS)

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321737)


In this tutorial, part one of five, you learn how to:

> [!div class="checklist"]
> - Install the Kubernetes CLI, `kubectl`.
> - Install the `aks-preview` Azure CLI extension.
> - Register the `AzureLinuxOSGuardPreview` feature flag.
> - Create an Azure resource group.
> - Create and deploy an Azure Linux with OS Guard cluster.
> - Configure `kubectl` to connect to your Azure Linux with OS Guard cluster.

In later tutorials, you learn how to add an Azure Linux with OS Guard node pool to an existing cluster and migrate existing nodes to Azure Linux with OS Guard.

## Considerations and limitations
	
Before you begin, review the following considerations and limitations for Azure Linux with OS Guard (preview):
	
- Kubernetes version 1.32.0 or higher is required for Azure Linux with OS Guard.
- All Azure Linux with OS Guard images have [Federal Information Process Standard (FIPS)](/azure/aks/enable-fips-nodes) and [Trusted Launch](/azure/aks/use-trusted-launch) enabled.
- Azure CLI and ARM templates are the only supported deployment methods for Azure Linux with OS Guard on AKS in preview. PowerShell and Terraform aren't supported.
- Arm64 images aren't supported with Azure Linux with OS Guard on AKS in preview.
- `NodeImage` and `None` are the only supported OS Upgrade channels for Azure Linux with OS Guard on AKS. `Unmanaged` and `SecurityPatch` are incompatible with Azure Linux with OS Guard due to the immutable /usr directory.
- [Artifact Streaming](/azure/aks/artifact-streaming) isn't supported.
- Kata Containers aren't supported.
- [Confidential Virtual Machines (CVMs)](/azure/aks/confidential-containers-overview) aren't supported.
- [Gen 1 virtual machines (VMs)](/azure/aks/aks-virtual-machine-sizes#vm-support-on-aks) aren't supported.

## Prerequisites

- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.
- [Install the `aks-preview` Azure CLI extension](#install-the-aks-preview-azure-cli-extension).
- [Register the `AzureLinuxOSGuardPreview` feature flag](#register-the-azure-linux-os-guard-preview-feature-flag).

### Install the aks-preview Azure CLI extension
	
[!INCLUDE [preview features callout](~/reusable-content/ce-skilling/azure/includes/aks/includes/preview/preview-callout.md)]

- Install the `aks-preview` extension using the [`az extension add`](/cli/azure/extension#az-extension-add) command.

    ```azurecli-interactive
    az extension add --name aks-preview
    ```
	
- Update to the latest version of the extension using the [`az extension update`](/cli/azure/extension#az-extension-update) command.
	
    ```azurecli-interactive
    az extension update --name aks-preview
    ```
	
### Register the Azure Linux OS Guard Preview feature flag
	
1. Register the `AzureLinuxOSGuardPreview` feature flag using the [`az feature register`](/cli/azure/feature#az-feature-register) command.
	
    ```azurecli-interactive
    az feature register --namespace "Microsoft.ContainerService" --name "AzureLinuxOSGuardPreview"
    ```
	
    It takes a few minutes for the status to show _Registered_.
	
1. Verify the registration status using the [`az feature show`](/cli/azure/feature#az-feature-show) command.
	
    ```azurecli-interactive
    az feature show --namespace "Microsoft.ContainerService" --name "AzureLinuxOSGuardPreview"
    ```
	
1. When the status reflects _Registered_, refresh the registration of the `Microsoft.ContainerService` resource provider using the [`az provider register`](/cli/azure/provider#az-provider-register) command.
	
    ```azurecli-interactive
    az provider register --namespace "Microsoft.ContainerService"
    ```

## Create a resource group

An Azure resource group is a logical group in which Azure resources are deployed and managed. When creating a resource group, it's required to specify a location. This location is:

- The storage location of your resource group metadata.
- Where your resources run in Azure if you don't specify another region when creating a resource.

Create a resource group using the [`az group create`](/cli/azure/group#az-group-create) command. Before running the command, environment variables are declared to ensure unique resource names for each deployment.

```azurecli-interactive
export REGION="EastUS2"
az group create --name $RESOURCE_GROUP_NAME --location $REGION
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

## Create an Azure Linux with OS Guard cluster

Create an AKS cluster using the [`az aks create`](/cli/azure/aks#az-aks-create) command with the `--os-sku` parameter to provision the Azure Linux Container Host with an Azure Linux image. The following example creates an Azure Linux with OS Guard cluster:

```azurecli-interactive
az aks create --name $MY_AZ_CLUSTER_NAME --resource-group $MY_RESOURCE_GROUP_NAME --os-sku AzureLinuxOSGuard --node-osdisk-type Managed --enable-fips-image --enable-secure-boot --enable-vtpm
```

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

To configure `kubectl` to connect to your Kubernetes cluster, use the `az aks get-credentials` command. The following example gets credentials for the Azure Linux Container Host cluster using the resource group and cluster name created earlier:

```azurecli
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

To verify the connection to your cluster, run the [kubectl get nodes](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes:

```azurecli-interactive
kubectl get nodes
```

<!-- expected_similarity=0.3 -->
```text
NAME                           STATUS   ROLES   AGE     VERSION
aks-nodepool1-00000000-0       Ready    agent   10m     v1.20.7
aks-nodepool1-00000000-1       Ready    agent   10m     v1.20.7
```

## Next steps

In this tutorial, you created and deployed an Azure Linux with OS Guard cluster. You learned how to: 

> [!div class="checklist"]
> * Install the Kubernetes CLI, `kubectl`.
> * Install the aks-preview Azure CLI extension
> * Register the `AzureLinuxOSGuardPreview` feature flag
> * Create an Azure resource group.
> * Create and deploy an Azure Linux Container Host cluster.
> * Configure `kubectl` to connect to your Azure Linux Container Host cluster.

In the next tutorial, you'll learn how to add an Azure Linux with OS Guard node pool to an existing cluster.

> [!div class="nextstepaction"]
> [Add an Azure Linux with OS Guard node pool](./tutorial-azure-linux-os-guard-add-nodepool.md)
