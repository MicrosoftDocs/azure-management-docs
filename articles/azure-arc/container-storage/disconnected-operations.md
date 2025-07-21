---
title: Disconnected operations
description: Learn about disconnected operations in Azure Container Storage enabled by Azure Arc, including their benefits and use cases.
author: asergaz
ms.author: sergaz
ms.topic: concept-article
ms.date: 06/25/2025

#CustomerIntent: As a systems administrator, I want to understand disconnected operations so that I can manage my Azure Container Storage more effectively.
---

# Disconnected operations

Azure Container Storage enabled by Azure Arc is architected to support your disconnected operational needs. In this context, disconnection refers to your local Kubernetes cluster losing network connection to Azure. 

This feature set is intended for use cases where the cluster is intended to be connected to the network always, but could experience intentional or accidental outages lasting up to **72 hours**. 

In this case, Local Shared Volumes and Ingest Subvolumes continue to receive writes from your applications when the network connection is down. This allows continuity of work at the edge.  

With a network connection, Ingest Subvolumes continuously upload; in the disconnected scenario, they continue to attempt to upload, and once the connection is restored, the changes in files are synchronized to Azure. This ensures data consistency and continuity. 

This ability to continue operation while disconnected is perfect for many customer applications, including manufacturing operations that don’t require real time cloud data.  


## Next steps

- [Configure your Local Shared Edge volumes](local-shared-edge-volumes.md)
- [Configure your Cloud Ingest Edge Volumes](cloud-ingest-edge-volume-configuration.md)
