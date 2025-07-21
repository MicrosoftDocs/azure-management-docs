---
title: Edge Volumes overview
description: Learn about the Edge Volumes offerings from Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: overview
ms.date: 05/05/2025

# Customer intent: As an IoT application developer, I want to utilize Edge Volumes for data storage and ingestion, so that I can optimize local resource utilization and manage data effectively in disconnected environments.
---

# Overview of Edge Volumes

This article describes the Edge Volumes offering from Azure Container Storage enabled by Azure Arc. 

## What are the Edge Volumes offerings?

The first Edge Volumes offering is *Local Shared Edge Volumes*, providing highly available, failover-capable storage, local to your Kubernetes cluster. This shared storage type remains independent of cloud infrastructure, making it ideal for scratch space, temporary storage, and locally persistent data unsuitable for cloud destinations. 

The second offering is *Cloud Ingest Edge Volumes*, which facilitates limitless data ingestion from edge to Blob, including Azure Data Lake Storage Gen2 and OneLake. Files written to this storage type are seamlessly transferred to Blob storage and subsequently purged from the local cache once confirmed uploaded, ensuring space availability for new data. Configurable policies allow for flexibility in upload and purge behaviors. Moreover, this storage option supports data integrity in disconnected environments, enabling local storage and synchronization upon reconnection to the network.

Tailored for IoT applications, Edge Volumes not only eliminates local storage concerns and ingest limitations, but also optimizes local resource utilization and reduces storage requirements.

## How do Edge Volumes work?

:::image type="content" source="media/edge-volumes-overview/container-storage-edge-volumes-overview.png" alt-text="Diagram of Azure Container Storage enabled by Azure Arc edge volumes overview." lightbox="media/edge-volumes-overview/container-storage-edge-volumes-overview.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

You write to Edge Volumes as if they were your local file system. For a **Local Shared Edge Volume**, your data is stored and left untouched. For a **Cloud Ingest Edge Volume**, the volume follows your specified ingest policy for delays and ingestion times, and uploads your data. It will then purge the local copy, also in accord with your specified ingest policy parameter, allowing you to keep your local volume clear of old data and continue to receive new data.

## Next steps

- [Prepare Linux](prepare-linux-edge-volumes.md)
- [Install Edge Volumes](install-edge-volumes.md)
- [Configure a Local Shared Edge Volume](local-shared-edge-volumes.md)
- [Configure a Cloud Ingest Edge Volume](cloud-ingest-edge-volume-configuration.md)
- [Monitor your deployment](monitor-deployment-edge-volumes.md)