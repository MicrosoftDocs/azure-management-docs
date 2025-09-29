---
title: Volumes and subvolumes
description: Learn about Volumes and subvolumes in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: concept-article
ms.date: 09/27/2025
# Customer intent: As a cloud architect, I want to understand the functionalities of Edge Volumes and subvolumes in Azure Container Storage, so that I can effectively manage local and cloud data synchronization for my applications.
---

# Overview of volumes and subvolumes

In Azure Container Storage enabled by Azure Arc, a *volume* refers to a [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/), the Kubernetes resource object.

## Edge Volumes

An *Edge Volume* is a mountable filesystem, consisting of the Edge Volume service, a local storage volume, and other components needed to present a filesystem to applications. 

:::image type="content" source="media/volumes-subvolumes/edge-volumes.png" alt-text="Diagram showing architecture of edge volumes.":::

Edge Volumes run in one of two **volume modes**: a **Local Shared** mode, in which the underlying local storage isn't backed by the cloud, and a **Cloud** mode in which the volume has a connection to Azure Blob Storage. 

### Local Shared mode

For Edge Volumes in Local Shared mode, there's no corresponding cloud container. Data is stored only on the edge cluster in a corresponding persistent volume. This mode is intended for applications that keep local databases or have their own logic for uploading data to the cloud. 

### Cloud mode

An Edge Volume in Cloud mode consists of a read-only pseudo-filesystem containing subdirectories that are mapped to containers in Azure storage accounts. These subdirectories are called *subvolumes*.  

When setting up a subvolume, you can choose between two different Custom Resource Definitions (CRDs).  

- **Ingest CRD:** For Ingest subvolumes, files written to it are automatically uploaded to a specified cloud container, and then the local copy of that file is evicted. This process makes this policy well-suited for applications that create file-based data at the edge, but don't need to read that data back at the edge--only in the cloud. This mode continues to allow writes when the cluster is disconnected from the cloud, resuming uploads after connectivity is reestablished. Files that are successfully uploaded to the cloud are removed from the local namespace after a user-specified time, after which the cluster can no longer read them.  

- **Mirror CRD:** In a Mirror subvolume (preview), specified cloud data is presented as a local read-only copy of that cloud file. This allows users to interact with cloud based data, without changing the original file. This is perfect for content distribution from a centralized location to many different geographic sites, or for customers to be able to interact with a cloud file even without an active cloud connection. 


## Edge Volume Details

All the subvolumes within an Edge Volume share the same local storage capacity that was specified when the Persistent Volume Claim (PVC) was created. There's also a feature to limit the concurrency of data upload, and that is done at the Edge Volume level, not at the subvolume level; therefore, all subvolumes attached to that Edge Volume are subject to this concurrency limit.

However, Ingest or Mirror behavior is set at the subvolume level, so those parameters are set for each individual subvolume. This can be helpful to specify different storage locations in the cloud for different data streams.

## Next steps

- [Configure Local Shared Volumes](howto-configure-local-shared-edge-volumes.md).
- [Configure Cloud Ingest subvolumes](howto-configure-cloud-ingest-subvolumes.md).
- [Configure Cloud Mirror subvolumes](howto-configure-cloud-mirror-subvolumes.md).
- [Using Edge Volumes together](storage-options.md).