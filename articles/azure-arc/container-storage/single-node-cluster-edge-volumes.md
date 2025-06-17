---
title: Prepare Linux for Edge Volumes using a single-node or two-node cluster
description: Learn how to prepare Linux for Edge Volumes with a single-node or two-node cluster in Azure Container Storage enabled by Azure Arc using AKS enabled by Azure Arc, Edge Essentials, or Ubuntu.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 03/12/2025
zone_pivot_groups: platform-select-with-other
# Customer intent: As a DevOps engineer, I want to configure Linux for Edge Volumes on single-node or two-node clusters, so that I can ensure optimal performance and compatibility with Azure Container Storage enabled by Azure Arc.
---

# Prepare Linux for Edge Volumes using a single-node or two-node cluster

This article describes how to prepare Linux using a single-node or two-node cluster, and assumes you [fulfilled the prerequisites](prepare-linux-edge-volumes.md#prerequisites).

::: zone pivot="aks-other"
## Prepare Linux with AKS enabled by Azure Arc

If you run a single-node or two-node cluster on Linux with AKS enabled by Azure Arc, you don't need to perform any additional steps.

::: zone-end

::: zone pivot="aks-ee-other"
[!INCLUDE [single-node-edge-essentials](includes/single-node-edge-essentials.md)]
::: zone-end

::: zone pivot="ubuntu-other"
[!INCLUDE [single-node-ubuntu](includes/single-node-ubuntu.md)]
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

1. Run the following command to install the local path provisioner:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/Azure/AKS-Edge/main/samples/storage/local-path-provisioner/local-path-storage.yaml
   ```
::: zone-end

## Next steps

[Install Azure Container Storage enabled by Azure Arc Edge Volumes](install-edge-volumes.md)
