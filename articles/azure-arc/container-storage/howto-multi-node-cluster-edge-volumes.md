---
title: Prepare Linux for Edge Volumes using a multi-node cluster
description: Learn how to prepare Linux for Edge Volumes with a multi-node cluster using AKS enabled by Azure Arc, Edge Essentials, or Ubuntu.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.custom:
  - linux-related-content
  - build-2025
ms.date: 03/12/2025
zone_pivot_groups: platform-select-with-other
# Customer intent: As a system administrator, I want to prepare a Linux environment for Edge Volumes using a multi-node cluster, so that I can ensure optimal performance and compatibility with Azure Container Storage.
---

# Prepare Linux for Edge Volumes using a multi-node cluster

This article describes how to prepare Linux using a multi-node cluster, and assumes you [fulfilled the prerequisites](howto-prepare-linux-edge-volumes.md#prerequisites).

::: zone pivot="aks-other"
## Prepare Linux with AKS enabled by Azure Arc

If you run a multi-node cluster on Linux with AKS enabled by Azure Arc, you don't need to perform any additional steps.

::: zone-end

::: zone pivot="aks-ee-other"
[!INCLUDE [multi-node](includes/multi-node-edge-essentials.md)]

::: zone-end

::: zone pivot="ubuntu-other"
## Prepare Linux with Ubuntu

This section describes how to prepare Linux with Ubuntu if you run a multi-node cluster.

[!INCLUDE [multi-node-ubuntu](includes/multi-node-ubuntu.md)]
::: zone-end

::: zone pivot="other"
## Prepare Linux with other platforms

The available platform options are production-like environments that Microsoft validated. These platforms aren't necessarily the only environments on which Azure Container Storage enabled by Azure Arc can run. Azure Container Storage can run on any Arc-enabled Kubernetes cluster that meets the Azure Arc-enabled Kubernetes system requirements. If you're running on an environment not listed, here are a few suggestions to increase the likelihood of a successful installation:

1. Run the following commands to increase the user watch and instance limits:

   ```bash
   echo fs.inotify.max_user_instances=8192 | sudo tee -a /etc/sysctl.conf
   echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

1. Run the following commands to increase the file descriptor limit for better performance:

   ```bash
   echo fs.file-max = 100000 | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

1. Run the following command to install the required **NVME over TCP** module for your kernel:

   ```bash
   sudo apt install linux-modules-extra-`uname -r`
   ```

1. Run the following command to set the number of `HugePages` to 512:

   ```bash
   HUGEPAGES_NR=512
   echo $HUGEPAGES_NR | sudo tee /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
   echo "vm.nr_hugepages=$HUGEPAGES_NR" | sudo tee /etc/sysctl.d/99-hugepages.conf
   ```
::: zone-end

## Next steps

[Install Extension](howto-install-edge-volumes.md)
