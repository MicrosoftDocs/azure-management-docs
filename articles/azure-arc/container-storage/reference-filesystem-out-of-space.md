---
title: Filesystem Out of Space Behavior
description: Learn about the filesystem interface and behavior around what happens when the underlying filesystem becomes full
author: adamschwab
ms.author: adschwab
ms.reviewer: stevenpepin
ms.date: 03/10/2026
ms.topic: concept-article
ms.custom:
  - linux-related-content
---

# Filesystem out of space behavior

Ultimately, Azure Container Storage enabled by Azure Arc provides a filesystem interface to applications. Filesystems have specific behaviors that applications must be designed to handle. This article explains what happens when the filesystem becomes full and how EdgeVolume filesystems behave in this condition.

## What happens when an EdgeVolume is full?

There are several ways that an EdgeVolume can become full. Upon creation, EdgeVolumes allocate storage with a Kubernetes storage provider. The storage provider is configured with the setting `defaultDiskStorageClasses` in the [EdgeStorageConfiguration CRD](howto-install-edge-volumes.md?#configuration-crd). The exact amount of space available depends on both the storage class configured for the EdgeVolume and the capacity requested for that EdgeVolume. But regardless of how the EdgeVolume was configured, it can run out of space.

### Out of space error

EdgeVolume filesystems store files in an internal filesystem. If this filesystem runs out of available space, attempts to write into the mounted folder result in IO errors such as ENOSPC. Most applications aren't equipped to gracefully handle such errors and on receiving an ENOSPC exit with an error message such as:

`<application error>: No Space Left on Device`

There are two different ways this underlying filesystem can run out of space:

- The available bytes allocated for the filesystem can run out.
- The available inodes allocated for the filesystem can run out.

Which of these cases is more likely to happen depends on the workload. Generally running out of space is more likely. Running out of inodes would only happen if the average file size is small (typically less than 16 KiB).

### Ingest out of space

Even though ingest subvolumes remove (evict) local files after a successful upload to blob storage, it's possible for an ingest subvolume to cause its underlying filesystem to run out of space:

- If the rate of data creation outpaces the available network bandwidth to write the data out to Azure Blob.
- If the cluster becomes disconnected from Azure Blob for a long enough time for the local filesystem to fill up.

If either case occurs, eventually an ingest subvolume can become full and application filesystem operations will fail with error ENOSPC.

### Out of space mitigation

In all of the previously mentioned out of space conditions, deletion should be an option to clear up space. Some Kubernetes storage providers also allow for increasing the size of the backing storage volume.

If the out of space condition is for an ingest subvolume and it's due to insufficient bandwidth, look into raising the EdgeVolume `ingestOperationConcurrency` setting to allow more bandwidth for ingest operations. It's also possible the local data write throughput is higher than the available bandwidth to Azure. If so, you might need to minimize what data is written into the shared ingest folder from applications or work to improve the available network bandwidth to Azure.

Note about `rancher.io/local-path` (the default storage provider for most k3s Kubernetes clusters): the underlying storage mechanism used here doesn't create a separate filesystem from the rest of the system. It simply mounts into a particular folder on the host (the default location is `/var/lib/rancher/k3s/storage`, though this location is configurable). This provider can be particularly dangerous when running out of space if the local-path folder is in the same filesystem as the host OS. If so, any local-path running out of space means the whole OS partition is out of space. Then the whole node could become unresponsive and difficult to recover.

If you use the local-path provisioner, make sure the target folder configured is a separate filesystem from your host OS so that if it runs out of space the OS won't be impacted.

### Mirror out of space

Mirror subvolumes are a bit different because their folders are read-only. No file creates or writes can happen from application pods into these folders.

Unfortunately, mirror subvolumes aren't exempt from running out of space.

- Mirror subvolumes and ingest subvolumes are subvolumes, which means they can share the same storage (represented by the EdgeVolume). They exist as top-level folders in this volume and share the space.
- Mirror subvolumes mirror blob containers based on the configuration of the MirrorSubvolume CRD. The backing blob container can have effectively infinite capacity, but local storage certainly doesn't.

If an EdgeVolume filesystem runs out of space, some of the mirrored blobs won't be transferred into the local mirror folder location. The mirror sync might complete with file errors.

When a mirror sync is kicked off, files to download are listed out from the target blob container. For each file to download, the edge application running the sync allocates some storage space in a hidden folder with a command called [`fallocate`](https://man7.org/linux/man-pages/man2/fallocate.2.html). If the EdgeVolume filesystem has insufficient unallocated storage for the full file, this `fallocate` call will fail. Error messages from `fallocate` failures end up in the status of the MirrorSubvolume Kubernetes resource like this:

```yaml
status:
  fileErrors:
  - error: 'fallocate failed for staging file <staging location>:
      no space left on device'
    export: 2
    fileName: <blob name>
```

The overall mirror sync might complete but individual files that don't fit remain as file error entries. The order that the files attempt to download isn't guaranteed, and a failed `fallocate` won't stop the sync from continuing.

The following are some mitigation strategies when dealing with mirror subvolumes running out of space:

- Rework the backing container so that the blobs to be mirrored don't overrun the storage capacity of the EdgeVolume. This mitigation can be managed without changing the MirrorSubvolume configuration. Once the tweaks to the blob are complete, resync the mirror. If the configured Azure Storage container's blobs all fit in the available space, the file errors will disappear.
- Configure a prefix in the MirrorSubvolume custom resource to filter the blobs to be mirrored to just include blobs matching the prefix. In combination with the first strategy, this mitigation can allow for rearranging the blobs without having to remove any blobs from the Azure Storage container.
- Some storage providers allow for increasing the volume size. If you're using such a storage provider and you have extra space available, you might be able to increase your allocated storage to the EdgeVolume to make all the mirrored blobs fit in the EdgeVolume filesystem.

Note that blob deletions result in corresponding deletions on the edge. Additionally, blob changes result in new `fallocate` calls. If the `fallocate` succeeds, the full blob downloads to a temporary file and then gets renamed over the old file's location. Because of this temporary file, blob changes require the full space for that blob to be available on the edge to complete the change. Otherwise, the old file will remain in place and a file error will appear.
