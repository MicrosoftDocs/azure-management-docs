---
title: Install Edge Volumes
description: Learn how to install the Edge Volumes offering from Azure Container Storage enabled by Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 10/28/2024
---

# Install Azure Container Storage enabled by Azure Arc Edge Volumes

This article describes the steps to install the Azure Container Storage enabled by Azure Arc extension.

## Install Azure IoT Operations dependencies

First, run the following command to install the Azure IoT Operations dependencies:

```azurecli 
az k8s-extension create --cluster-name "${YOUR-CLUSTER-NAME}" --name "${NAME}-certmgr" --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager
```

## Install the Azure Container Storage enabled by Azure Arc extension

Install the Azure Container Storage enabled by Azure Arc extension using the following command:

```azurecli
az k8s-extension create --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-name "${YOUR-CLUSTER-NAME}" --cluster-type connectedClusters --name azure-arc-containerstorage --extension-type microsoft.arc.containerstorage
```

> [!NOTE]
> By default, the `--release-namespace` parameter is set to `azure-arc-containerstorage`. If you want to override this setting, add the `--release-namespace` flag to the following command and populate it with your details. Any values set at installation time persist throughout the installation lifetime (including manual and auto-upgrades).

> [!IMPORTANT]
> If you use OneLake, you must use a unique extension name for the `--name` variable in the `az k8s-extension create` command.

## Configuration operator

### Configuration CRD

The Azure Container Storage enabled by Azure Arc extension uses a Custom Resource Definition (CRD) in Kubernetes to configure the storage service. Before you publish this CRD on your Kubernetes cluster, the Azure Container Storage enabled by Azure Arc extension is dormant and uses minimal resources. Once your CRD is applied with the configuration options, the appropriate storage classes, CSI driver, and service PODs are deployed to provide services. In this way, you can customize Azure Container Storage enabled by Azure Arc to meet your needs, and it can be reconfigured without reinstalling the Arc Kubernetes Extension. Common configurations are contained here, however this CRD offers the capability to configure non-standard configurations for Kubernetes clusters with differing storage capabilities.

#### [Single node or 2-node cluster](#tab/single)

#### Single node or 2-node cluster with Ubuntu or Edge Essentials

If you run a single node or 2-node cluster with **Ubuntu** or **Edge Essentials**, follow these instructions:

1. Create a file named **edgeConfig.yaml** with the following contents:

   ```yaml
   apiVersion: arccontainerstorage.azure.net/v1
   kind: EdgeStorageConfiguration
   metadata:
     name: edge-storage-configuration
   spec:
     defaultDiskStorageClasses:
       - "default"
       - "local-path"
     serviceMesh: "osm" 
   ```

1. To apply this .yaml file, run:

   ```bash
   kubectl apply -f "edgeConfig.yaml"
   ```

#### [Multi-node cluster](#tab/multi)

#### Multi-node cluster with Ubuntu or Edge Essentials

If you run a 3 or more node Kubernetes cluster with **Ubuntu** or **Edge Essentials**, follow these instructions. This configuration installs the ACStor storage subsystem to provide fault-tolerant, replicated storage for Kubernetes clusters with 3 or more nodes:

1. Create a file named **edgeConfig.yaml** with the following contents:

   > [!NOTE]
   > To relocate storage to a different location on disk, update `diskMountPoint` with your desired path.

   ```yaml
   apiVersion: arccontainerstorage.azure.net/v1
   kind: EdgeStorageConfiguration
   metadata:
     name: edge-storage-configuration
   spec:
     defaultDiskStorageClasses:
       - acstor-arccontainerstorage-storage-pool
     serviceMesh: "osm"
   ---
   apiVersion: arccontainerstorage.azure.net/v1
   kind: ACStorConfiguration
   metadata:
     name: acstor-configuration
   spec:
       diskMountPoint: /mnt
       diskCapacity: 10Gi
       createStoragePool:
           enabled: true
           replicas: 3
   ```

1. To apply this .yaml file, run:

   ```bash
   kubectl apply -f "edgeConfig.yaml"
   ```

#### [Arc-connected AKS/AKS Arc](#tab/arc)

#### Arc-connected AKS or AKS Arc

If you run a single-node or multi-node cluster with **Arc-connected AKS** or **AKS enabled by Arc**, follow these instructions:

1. Create a file named **edgeConfig.yaml** with the following contents:

   ```yaml
   apiVersion: arccontainerstorage.azure.net/v1
   kind: EdgeStorageConfiguration
   metadata:
     name: edge-storage-configuration
   spec:
     defaultDiskStorageClasses:
       - "default"
       - "local-path"
     serviceMesh: "osm" 
   ```

1. To apply this .yaml file, run:

   ```bash
   kubectl apply -f "edgeConfig.yaml"
   ```

---

## Next steps

- [Configure your Local Shared Edge volumes](local-shared-edge-volumes.md)
- [Configure your Cloud Ingest Edge Volumes](cloud-ingest-edge-volume-configuration.md)
