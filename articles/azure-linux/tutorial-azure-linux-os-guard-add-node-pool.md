---
title: Azure Linux with OS Guard (preview) for AKS tutorial - Add an Azure Linux with OS Guard (preview) node pool to an existing Azure Kubernetes Service (AKS) cluster
description: In this Azure Linux with OS Guard for AKS tutorial, you learn how to add an Azure Linux with OS Guard node pool to your existing AKS cluster.
author: florataagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 09/24/2025
# Customer intent: As a DevOps engineer, I want to add an Azure Linux with OS Guard node pool to my existing AKS cluster, so that I can accommodate varying compute and security needs for my applications.
---

# Tutorial: Add an Azure Linux with OS Guard (preview) node pool to an existing Azure Kubernetes Service (AKS) cluster

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321935)

In Azure Kubernetes Service (AKS), nodes with the same configurations are grouped together into node pools. Each node pool contains the virtual machines (VMs) that run your applications. In the previous tutorial, you created an Azure Linux with OS Guard cluster with a single node pool. To meet the varying compute, storage, or security requirements of your applications, you can add user node pools.

In this tutorial, part two of five, you learn how to:

> [!div class="checklist"]
>
> - Add an Azure Linux with OS Guard node pool to an existing cluster.
> - Check the status of your node pools.

In later tutorials, you learn how to migrate nodes to Azure Linux with OS Guard and enable telemetry to monitor your clusters.

## Considerations and limitations
	
Before you begin, review the following considerations and limitations for Azure Linux with OS Guard (preview):
	
- Kubernetes version 1.32.0 or higher is required for Azure Linux with OS Guard.
- All Azure Linux with OS Guard images have [Federal Information Process Standard (FIPS)](/azure/aks/enable-fips-nodes) and [Trusted Launch](/azure/aks/use-trusted-launch) enabled.
- Azure CLI and ARM templates are the only supported deployment methods for Azure Linux with OS Guard on AKS in preview. PowerShell and Terraform aren't supported.
- [Arm64](/azure/aks/use-arm64-vms) images aren't supported with Azure Linux with OS Guard on AKS in preview.
- `NodeImage` and `None` are the only supported [OS Upgrade channels](/azure/aks/auto-upgrade-node-os-image) for Azure Linux with OS Guard on AKS. `Unmanaged` and `SecurityPatch` are incompatible with Azure Linux with OS Guard due to the immutable /usr directory.
- [Artifact Streaming](/azure/aks/artifact-streaming) isn't supported.
- [Pod Sandboxing](/azure/aks/use-pod-sandboxing) isn't supported.
- [Confidential Virtual Machines (CVMs)](/azure/aks/confidential-containers-overview) aren't supported.
- [Gen 1 virtual machines (VMs)](/azure/aks/aks-virtual-machine-sizes#vm-support-on-aks) aren't supported.

## Prerequisites

- In the previous tutorial, you created and deployed an Azure Linux with OS Guard cluster. If you haven't completed these steps and want to follow along, see [Tutorial 1: Create a cluster with Azure Linux with OS Guard for AKS](./tutorial-azure-linux-os-guard-create-cluster.md).
- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

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

## Add an Azure Linux with OS Guard node pool

Add an Azure Linux with OS Guard node pool into your existing cluster using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command and specify `--os-sku AzureLinuxOSGuard`. Enabling [FIPS](/azure/aks/enable-fips-nodes), [secure boot](/azure/aks/use-trusted-launch), and [vtpm](/azure/aks/use-trusted-launch) are also required to use Azure Linux with OS Guard. The following example creates a node pool named _osgNodepool_ that adds three nodes in the _testAzureLinuxOSGuardCluster_ cluster in the _testAzureLinuxOSGuardResourceGroup_ resource group. Environment variables are declared and a random suffix is appended to the resource group and cluster names to ensure uniqueness.

```azurecli-interactive
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export NODEPOOL_NAME="np$RANDOM_SUFFIX"

az aks nodepool add \
    --resource-group $RESOURCE_GROUP \
    --cluster-name $CLUSTER_NAME \
    --name $NODEPOOL_NAME \
    --node-count 3 \
    --os-sku AzureLinuxOSGuard
    --node-osdisk-type Managed 
    --enable-fips-image 
    --enable-secure-boot 
    --enable-vtpm
```

Example output:
<!-- expected_similarity=0.3 -->
```JSON
{
  "agentPoolType": "VirtualMachineScaleSets",
  "count": 3,
  "name": "osgNodepool",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "testAzureLinuxOSGuardResourceGroupxxxxx",
  "type": "Microsoft.ContainerService/managedClusters/agentPools"
}
```

> [!NOTE]
> The name of a node pool must start with a lowercase letter and can only contain alphanumeric characters. For Linux node pools, the length must be between one and 12 characters.

## Check the node pool status

Check the status of your node pools using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az-aks-nodepool-list) command and specify your resource group and cluster name.

```azurecli-interactive
az aks nodepool list --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME
```

Example output:
<!-- expected_similarity=0.3 -->
```output
[
  {
    "agentPoolType": "VirtualMachineScaleSets",
    "availabilityZones": null,
    "count": 3,
    "enableAutoScaling": false,
    "enableEncryptionAtHost": false,
    "enableFips": false,
    "enableNodePublicIp": false,
    "id": "/subscriptions/REDACTED/resourcegroups/myAKSResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/myAKSClusterxxxxx/agentPools/npxxxxxx",
    "maxPods": 110,
    "mode": "User",
    "name": "npxxxxxx",
    "nodeImageVersion": "AzureLinuxContainerHost-2025.10.03",
    "orchestratorVersion": "1.32.6",
    "osDiskSizeGb": 128,
    "osDiskType": "Managed",
    "osSku": "AzureLinux",
    "osType": "Linux",
    "powerState": {
      "code": "Running"
    },
    "provisioningState": "Succeeded",
    "resourceGroup": "myAKSResourceGroupxxxxx",
    "type": "Microsoft.ContainerService/managedClusters/agentPools",
    "vmSize": "Standard_DS2_v2"
  },
  {
    "agentPoolType": "VirtualMachineScaleSets",
    "availabilityZones": null,
    "count": 3,
    "enableAutoScaling": false,
    "enableEncryptionAtHost": false,
    "enableFips": false,
    "enableNodePublicIp": false,
    "id": "/subscriptions/REDACTED/resourcegroups/myAKSResourceGroupxxxxx/providers/Microsoft.ContainerService/managedClusters/myAKSClusterxxxxx/agentPools/npxxxxxx",
    "maxPods": 110,
    "mode": "User",
    "name": "npxxxxxx",
    "nodeImageVersion": "AzureLinuxOSGuard-2025.10.03",
    "orchestratorVersion": "1.32.6",
    "osDiskSizeGb": 128,
    "osDiskType": "Managed",
    "osSku": "AzureLinuxOSGuard",
    "osType": "Linux",
    "powerState": {
      "code": "Running"
    },
    "provisioningState": "Succeeded",
    "resourceGroup": "myAKSResourceGroupxxxxx",
    "type": "Microsoft.ContainerService/managedClusters/agentPools",
    "vmSize": "Standard_DS2_v2"
  }
]
```

## Next steps

In this tutorial, you added an Azure Linux with OS Guard node pool to your existing cluster. In the next tutorial, you learn how to migrate existing nodes to Azure Linux with OS Guard.

> [!div class="nextstepaction"]
> [Migrate to Azure Linux with OS Guard](./tutorial-azure-linux-os-guard-migration.md)