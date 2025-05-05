---
title: Cache Volumes (preview) overview
description: Learn about the Cache Volumes offering from Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: overview
ms.date: 08/26/2024

---

# Overview of Cache Volumes (preview)

This article describes the Cache Volumes (preview) offering from Azure Container Storage enabled by Azure Arc.

## How does Cache Volumes (preview) work?

:::image type="content" source="media/container-storage-cache-volumes-overview.png" alt-text="Diagram of Cache Volumes (preview) in Azure Container Storage enabled by Azure Arc." lightbox="media/container-storage-cache-volumes-overview.png":::

Cache Volumes (preview) works by performing the following operations:

- **Write** - Your file is processed locally and saved in the cache. If the file doesn't change within 3 seconds, Cache Volumes automatically uploads it to your chosen blob destination.
- **Read** - If the file is already in the cache, the file is served from the cache memory. If it isn't available in the cache, the file is pulled from your chosen blob storage target.

## Next steps

- [Prepare Linux](prepare-linux.md)
- [Install Cache Volumes](install-cache-volumes.md)
- [Create a persistent volume](create-persistent-volume.md)
- [Monitor your deployment](azure-monitor-kubernetes.md)
