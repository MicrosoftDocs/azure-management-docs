---
title: Prepare Linux for Edge Volumes using a multi-node cluster (preview)
description: Learn how to prepare Linux for Edge Volumes with a multi-node cluster using AKS enabled by Azure Arc, Edge Essentials, or Ubuntu.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 10/29/2024
zone_pivot_groups: platform-select-with-other
---

# Prepare Linux for Edge Volumes using a multi-node cluster (preview)

This article describes how to prepare Linux using a multi-node cluster, and assumes you [fulfilled the prerequisites](prepare-linux.md#prerequisites).

::: zone pivot="aks"
## Prepare Linux with AKS enabled by Azure Arc

Install and configure Open Service Mesh (OSM) using the following commands:

```azurecli
az k8s-extension create --resource-group "YOUR_RESOURCE_GROUP_NAME" --cluster-name "YOUR_CLUSTER_NAME" --cluster-type connectedClusters --extension-type Microsoft.openservicemesh --scope cluster --name osm \
--config osm.osm.featureFlags.enableWASMStats=false" \
--config osm.osm.enablePermissiveTrafficPolicy=false" \
--config osm.osm.configResyncInterval=10s" \
--config osm.osm.osmController.resource.requests.cpu=100m" \
--config osm.osm.osmBootstrap.resource.requests.cpu=100m" \
--config osm.osm.injector.resource.requests.cpu=100m
```

::: zone-end

::: zone pivot="aks-ee"
[!INCLUDE [multi-node](includes/multi-node-edge-essentials.md)]

::: zone-end

::: zone pivot="ubuntu"
## Prepare Linux with Ubuntu

This section describes how to prepare Linux with Ubuntu if you run a multi-node cluster.

First, install and configure Open Service Mesh (OSM) using the following command:

```azurecli
az k8s-extension create --resource-group "YOUR_RESOURCE_GROUP_NAME" --cluster-name "YOUR_CLUSTER_NAME" --cluster-type connectedClusters --extension-type Microsoft.openservicemesh --scope cluster --name osm \
--config osm.osm.featureFlags.enableWASMStats=false" \
--config osm.osm.enablePermissiveTrafficPolicy=false" \
--config osm.osm.configResyncInterval=10s" \
--config osm.osm.osmController.resource.requests.cpu=100m" \
--config osm.osm.osmBootstrap.resource.requests.cpu=100m" \
--config osm.osm.injector.resource.requests.cpu=100m
```

[!INCLUDE [multi-node-ubuntu](includes/multi-node-ubuntu.md)]
::: zone-end

::: zone pivot="other"
The available platform options are production-like environments that Microsoft has validated. They are not necessarily the only environments on which Azure Container Storage enabled by Azure Arc can run. Azure Container Storage enabled by Azure Arc can run on any Arc-enabled Kubernetes cluster that meets the Azure Arc-enabled Kubernetes system requirements. If you're running on an environment not listed, here are a few suggestions to increase the likelihood of a successful installation:

1. Run the following command to increase the user watch and instance limits:

   ```bash
   echo fs.inotify.max_user_instances=8192 | sudo tee -a /etc/sysctl.conf
   echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

1. For better performance, increase the file descriptor limit:

   ```bash
   echo fs.file-max = 100000 | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

1. Install the required **NVME over TCP** module for your kernel using the following command:

   ```bash
   sudo apt install linux-modules-extra-`uname -r`
   ```

1. Set the number of HugePages to 512 using the following command:

   ```bash
   HUGEPAGES_NR=512
   echo $HUGEPAGES_NR | sudo tee /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
   echo "vm.nr_hugepages=$HUGEPAGES_NR" | sudo tee /etc/sysctl.d/99-hugepages.conf
   ```

::: zone-end

## Next steps

[Install Extension](install-edge-volumes.md)
