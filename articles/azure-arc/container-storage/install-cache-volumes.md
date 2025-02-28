---
title: Install Cache Volumes (preview)
description: Learn how to install the Cache Volumes offering from Azure Container Storage enabled by Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 10/29/2024

---

# Install Azure Container Storage enabled by Azure Arc Cache Volumes (preview)

This article describes the steps to install the Azure Container Storage enabled by Azure Arc extension.

## Optional: increase cache disk size

Currently, the cache disk size defaults to 8 GB. If you're satisfied with the cache disk size, see the next section, [Install the Azure Container Storage enabled by Azure Arc extension](#install-the-azure-container-storage-enabled-by-azure-arc-extension).  

If you use Edge Essentials, require a larger cache disk size, and already created a **config.json** file, append the key and value pair (`"cachedStorageSize": "20Gi"`) to your existing **config.json**. Don't erase the previous contents of **config.json**.

If you require a larger cache disk size, create **config.json** with the following contents:

```json
{
  "cachedStorageSize": "20Gi"
}
```

## Install Azure IoT Operations dependencies

Run the following command to install the Azure IoT Operations dependencies:

```azurecli 
az k8s-extension create --cluster-name "${YOUR-CLUSTER-NAME}" --name "${NAME}-certmgr" --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager
```

## Install the Azure Container Storage enabled by Azure Arc extension

Install the Azure Container Storage enabled by Azure Arc extension using the following command:

> [!NOTE]
> If you created a **config.json** file from the previous steps in [Prepare Linux](prepare-linux.md), append `--config-file "config.json"` to the following `az k8s-extension create` command. Any values set at installation time persist throughout the installation lifetime (including manual and auto-upgrades).

```bash
az k8s-extension create --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-name "${YOUR-CLUSTER-NAME}" --cluster-type connectedClusters --name hydraext --extension-type microsoft.arc.containerstorage --config previewFeaturesAllowed="cacheVolumes"
```

## Configuration operator

### Configuration CRD

The Azure Container Storage enabled by Azure Arc extension uses a Custom Resource Definition (CRD) in Kubernetes to configure the storage service. Before you publish this CRD on your Kubernetes cluster, the Azure Container Storage enabled by Azure Arc extension is dormant and uses minimal resources. Once your CRD is applied with the configuration options, the appropriate storage classes, CSI driver, and service PODs are deployed to provide services. In this way, you can customize Azure Container Storage enabled by Azure Arc to meet your needs, and it can be reconfigured without reinstalling the Arc Kubernetes Extension. Common configurations are contained here, however this CRD offers the capability to configure non-standard configurations for Kubernetes clusters with differing storage capabilities.

1. Create a file named **cacheConfig.yaml** with the following contents:

    ```yaml
    apiVersion: arccontainerstorage.azure.net/v1
    kind: CacheConfiguration
    metadata:
      name: cache-configuration
    spec:
      diskStorageClasses:
        - local-path
      cacheNodeStorageSize: "10Gi"
      metadataStorageSize: "4Gi"
      failoverCacheVolumeConsumers: true
      serviceMesh: "osm" # or "none"
    ```

1. To apply this .yaml file, run:

    ```bash
    kubectl apply -f "cacheConfig.yaml"
    ```

## Next steps

Once you complete these prerequisites, you can begin to [create a Persistent Volume (PV) with Storage Key Authentication](create-persistent-volume.md).
