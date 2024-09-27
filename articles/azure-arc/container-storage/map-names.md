---
title: Map file to object path names in Azure Container Storage enabled by Azure Arc (preview)
description: Learn how to map file to object path names in Azure Container Storage enabled by Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 09/27/2024
---

# Map file to object path names (preview)

This article describes how the paths created inside of your volume or Subvolume map to a local shared file system or a cloud destination.

## Local Shared Edge Volumes

After you create your Local Shared Edge Volume PVC and deploy an application bound to it, each pod you create has a subdirectory named **/data**, or whatever you specify in your deployment YAML, [such as this](local-shared-edge-volumes.md#create-a-local-shared-edge-volumes-persistent-volume-claim-pvc-and-configure-a-pod-against-the-pvc). Any data written here is saved to your local storage and protected by our replication feature if you installed on a 3 node or more cluster and enabled it. This data is never sent to the cloud.

## Cloud Edge Volumes

When you create your **edgeSubvolume.yaml** file [as described here](cloud-ingest-edge-volume-configuration.md#attach-subvolume-to-edge-volume), there is a `spec.path` parameter that's set to `exampleSubDir`, but that can be changed to any desired name. After you create your Cloud Edge Volume PVC and deploy an application bound to it, each pod you create has a directory named **/data**, or whatever you specify in your deployment YAML, [such as this](cloud-ingest-edge-volume-configuration.md#attach-your-app-kubernetes-native-application). There's a subdirectory under that mount path using your Subvolume `spec.path` parameter above; the rest of this example uses `/data/exampleSubDir`. Any data written to this base path lands in your storage destination as configured with the `spec.container` parameter. Within `/data/exampleSubDir`, if you create a subdirectory, then the files written to that subdirectory land in a subfolder of the top-level namespace. For example, an application that writes the file `/data/ exampleSubDir/2023/yearly.summary` is written to `<spec.storageaccountendpoint>/<spec.container>/2023/yearly.summary`.

## Next steps


