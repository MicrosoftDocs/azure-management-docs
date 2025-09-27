---
title: Map file to object path names in Azure Container Storage enabled by Azure Arc
description: Learn how to map file to object path names in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 10/31/2024
# Customer intent: As a cloud developer, I want to understand how to map file paths in Azure Container Storage to various storage destinations, so that I can configure my application to correctly save and access data in both local and cloud environments.
---

# Map file to object path names

This article describes how the paths created inside of your volume or subvolume map to a local shared file system or a cloud destination.

## Local Shared Edge Volumes

After you create your Local Shared Edge Volume PVC and deploy an application bound to it, each pod you create has a subdirectory named **/data**, or whatever you specify in your deployment YAML, [such as this](howto-configure-local-shared-edge-volumes.md#create-a-local-shared-edge-volumes-persistent-volume-claim-pvc-and-configure-a-pod-against-the-pvc). Any data written here is saved to your local storage and protected by our replication feature if you installed on a 3 node or more cluster and enabled it. This data is never sent to the cloud.

## Cloud Edge Volumes

When you create your **edgeSubvolume.yaml** file [as described here](howto-configure-cloud-ingest.md), there's a `spec.path` parameter that's set to `exampleSubDir`, but that can be changed to any desired name. After you create your Cloud Edge Volume PVC and deploy an application bound to it, each pod you create has a directory named **/data**, or whatever you specify in your deployment YAML, [such as this](howto-configure-cloud-ingest.md#attach-your-app-kubernetes-native-application). There's a subdirectory under that mount path using your subvolume `spec.path` parameter; the rest of this example uses **/data/exampleSubDir**. Any data written to this base path lands in your storage destination as configured with the `spec.container` parameter. Within **/data/exampleSubDir**, if you create a subdirectory, then the files written to that subdirectory land in a subfolder of the top-level namespace. For example, an application that writes the file **/data/exampleSubDir/2023/yearly.summary** is written to `<spec.storageaccountendpoint>/<spec.container>/2023/yearly.summary`.

> [!NOTE]
> Files can't be written directly from pod to **/data**, since the top level directory of a Cloud Edge Volume is read-only. If you get an error that the filesystem is read-only, it likely means you didn't configure your subvolume, or are not pointing your application to write to the subvolume path.

## Next steps

[Cloud Ingest Edge Volumes configuration](howto-configure-cloud-ingest.md)
