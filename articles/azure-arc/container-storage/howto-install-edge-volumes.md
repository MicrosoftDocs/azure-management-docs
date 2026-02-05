---
title: Install Edge Volumes
description: Learn how to install the Edge Volumes offering from Azure Container Storage enabled by Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 02/05/2026
# Customer intent: "As a cloud administrator, I want to install and configure the Azure Container Storage extension with Edge Volumes in Kubernetes, so that I can efficiently manage storage solutions across my containerized applications in a hybrid cloud environment."
---

# Install Azure Container Storage enabled by Azure Arc Edge Volumes

This article describes the steps to install the Azure Container Storage extension.

## Install certificate and trust managers

Azure Container Storage is dependent upon a `cert-manager` and `trust-manager`. You can bring your own, or these are available as a platform extension that can be installed using the following command:

```azurecli 
az k8s-extension create --cluster-name "${YOUR-CLUSTER-NAME}" --name "${NAME}-certmgr" --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager --release-train preview
```

> [!NOTE]
> This platform extension is supplied by Azure IoT Operations, but installing the platform extension does not install Azure IoT Operations on your device. 

## Install the Azure Container Storage enabled by Azure Arc extension

Install the Azure Container Storage extension using the following command:

```azurecli
az k8s-extension create --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-name "${YOUR-CLUSTER-NAME}" --cluster-type connectedClusters --name azure-arc-containerstorage --extension-type microsoft.arc.containerstorage
```

> [!NOTE]
> By default, the `--release-namespace` parameter is set to `azure-arc-containerstorage`. If you want to override this setting, add the `--release-namespace` flag to the following command and populate it with your details. Any values set at installation time persist throughout the installation lifetime (including manual and autoupgrades).

> [!IMPORTANT]
> If you use OneLake, you must use a unique extension name for the `--name` variable in the `az k8s-extension create` command.

## Configuration operator

### Configuration CRD

The Azure Container Storage extension uses a Custom Resource Definition (CRD) in Kubernetes to configure the storage service. Before you publish this CRD on your Kubernetes cluster, the Azure Container Storage extension is dormant and uses minimal resources. Once your CRD is applied with the configuration options, the appropriate storage classes, CSI driver, and service PODs are deployed to provide services. In this way, you can customize Azure Container Storage to meet your needs, and it can be reconfigured without reinstalling the Arc Kubernetes Extension. Common configurations are contained here, however this CRD offers the capability to configure nonstandard configurations for Kubernetes clusters with differing storage capabilities.

#### [Single-node or two-node cluster](#tab/single)

#### Single-node or two-node cluster with Ubuntu or Edge Essentials

If you run a single-node or two-node cluster with **Ubuntu** or **Edge Essentials**, follow these instructions:

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
   ```

1. To apply this .yaml file, run:

   ```bash
   kubectl apply -f "edgeConfig.yaml"
   ```

#### [Arc-enabled AKS/AKS Arc](#tab/arc)

#### Arc-enabled AKS or AKS Arc

If you run a single-node or multi-node cluster with **Arc-enabled AKS** or **AKS enabled by Arc**, follow these instructions:

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
   ```

1. To apply this .yaml file, run:

   ```bash
   kubectl apply -f "edgeConfig.yaml"
   ```

---

## Next steps

- [Configure your Local Shared Edge volumes](howto-configure-local-shared-edge-volumes.md)
- [Configure your Cloud Ingest Edge Volumes](howto-configure-cloud-ingest-subvolumes.md)
