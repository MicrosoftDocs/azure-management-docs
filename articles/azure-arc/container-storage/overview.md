---
title: What is Azure Container Storage enabled by Azure Arc?
description: Learn about Azure Container Storage enabled by Azure Arc, a first-party storage system designed for Arc-connected Kubernetes clusters.
author: asergaz
ms.author: sergaz
ms.topic: overview
ms.date: 05/05/2025

#customer intent: As a user, I want to understand the Azure Container Storage enabled by Azure Arc offering and its features.
# Customer intent: As a cloud architect, I want to understand how Azure Container Storage enabled by Azure Arc functions, so that I can evaluate its suitability for providing persistent storage in Arc-connected Kubernetes clusters.
---

# What is Azure Container Storage enabled by Azure Arc?

Azure Container Storage enabled by Azure Arc is a first-party storage system designed for Arc-connected Kubernetes clusters. This Arc extension can be deployed to write files to a *ReadWriteMany* persistent volume claim (PVC) where they can be stored locally, or  transferred to Azure Blob Storage destinations in the cloud. Azure Container Storage offers a range of features to support various workloads, such as Azure IoT Operations, and other Arc services. With high availability and fault-tolerance options available, this Arc extension is ready for production workloads.

:::image type="content" source="media/overview/container-storage-solution-architecture.png" alt-text="Diagram of Azure Container Storage enabled by Azure Arc solution architecture." lightbox="media/overview/container-storage-solution-architecture.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

## What does Azure Container Storage do?

Azure Container Storage serves as a native persistent storage system for Arc-connected Kubernetes clusters. Its primary role is to provide a flexible, reliable, fault-tolerant file system that allows data to be kept safely at the edge and/or to be tiered to Azure. For Azure IoT Operations and other Arc Services, Azure Container Storage is crucial in making Kubernetes clusters stateful. Key features of Arc-connected clusters running this extension include:

- **Tolerance to node failures:** When configured as a three node cluster, Azure Container Storage replicates data between nodes to ensure high availability and tolerance to single node failures.
- **Storage Local to your cluster**: With a Local Shared Edge Volume, the user can store data local to their edge deployment with a *ReadWriteMany* access model.
- **Data synchronization to Azure:** Azure Container Storage is configured with a storage target, so data written to volumes is automatically tiered to Azure Blob (block blob, Azure Data Lake Storage Gen2, or OneLake) in the cloud.
- **Simple connection:** Customers can easily connect to a configured volume using a CSI driver to start making persistent volume claims against their storage.
- **Observable:** Supports industry standard Kubernetes monitoring logs and metrics facilities, and supports Azure Monitor Agent observability.
- **Platform neutrality:** Azure Container Storage is a Kubernetes storage system that can run on any Arc Kubernetes supported platform. Validation was done for specific platforms, including Ubuntu + CNCF K3s/K8s, Windows IoT + AKS Edge Essentials, and Azure Local.

## Supported Azure regions

Azure Container Storage enabled by Azure Arc is available in the following Azure regions:

- East US
- East US 2
- West US
- West US 2
- West US 3
- North Europe
- West Europe

## Related content

- [Prepare Linux for Edge Volumes](howto-prepare-linux-edge-volumes.md)
- [Install Edge Volumes](howto-install-edge-volumes.md)
