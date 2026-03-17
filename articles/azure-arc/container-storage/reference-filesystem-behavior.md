---
title: Filesystem Behavior
description: Learn about the filesystem interface and behavior around shared filesystems and filesystems undergoing mirror syncs 
author: adamschwab
ms.author: adschwab
ms.reviewer: stevenpepin
ms.date: 03/10/2026
ms.topic: concept-article
ms.custom:
  - linux-related-content
---

# Filesystem behavior

Ultimately, Azure Container Storage enabled by Azure Arc provides a filesystem interface to applications. Filesystems have specific behaviors that applications must be designed to handle. This article explains how shared filesystem changes propagate and what happens to a mirror filesystem during mirror synchronization.

## What do we mean by a filesystem interface?

An application can access many filesystems in its folder structure via [mounts](https://man7.org/linux/man-pages/man8/mount.8.html). In Kubernetes, applications (pods) configure mounts using PVCs. When a pod mounts with an EdgeVolume PVC it gets access to a shared filesystem via a mounted folder.

See [this guide](howto-configure-cloud-ingest-subvolumes.md), for example, to configure a PVC for out extension's cloud-backed volume. A filesystem mounted with an EdgeVolume PVC has a local and remote component. The local filesystem receives application filesystem operations. It forwards filesystem operations on to the remote component that has access to the underlying storage.

## Change propagation and local caching

Many pods from any node in the cluster can mount the same shared EdgeVolume folder. Within this folder, writes from one pod are readable from other pods. If there are pods on separate nodes that are mounting the same shared folder changes can take some time to propagate. New or removed files and folders can take up to 60 seconds to propagate. File writes can take up to 30 seconds to propagate.

The reason for these delays is because of client-side cache invalidation times. When the pod does a read operation into the local filesystem, it might use a local cache and never make a network call. The cache entries it uses eventually expire, so once the relevant cache times out the pod's filesystem re-reads the information over the network. At this point, the local filesystem pulls the latest file metadata and contents.

File operation sequences that involve opening the file, reading, and then closing the file (as opposed to long-lived open files) always read the latest contents of the file. When the local pod's filesystem processes an open call, it goes directly to the server to check if the file changed.

## Mirror filesystem behavior

When a mirror sync occurs, an internal application runs to detect and perform filesystem operations to synchronize the edge filesystem to match a blob target. Mirror subvolume configuration is described in [this article](howto-configure-cloud-mirror-subvolumes.md).

The operations that might be performed to complete a mirror sync fall into the following categories:

- Creating new files or folders to match newly created blobs.
- Removing existing files or folders to match removed blobs.
- Modifying existing files to match modified blobs, by downloading the new blob and renaming it over top of the old file.
- Modifying file or folder metadata: owner, group, permissions, and xattrs based on metadata changes in the blob.

When the internal application performs these operations, it does so directly on the underlying filesystem. The changes eventually make their way to customer pods reading the filesystem. Once a change is made in the internal filesystem, it can take up to 30 seconds for file changes or 60 seconds for created or removed files to propagate to customer pods.

Since blob modification syncs are implemented using renames with full file contents, applications never see partially synced files. They either see the old file's contents or the new file's contents.

However, metadata changes are applied directly to the customer visible file or folder. This means it's possible to see partially synced metadata (owner, group, permissions, or xattrs).

