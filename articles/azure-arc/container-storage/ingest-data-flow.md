---
title: Ingest data flow
description: Learn ways to control data flow to the cloud in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: concept-article
ms.date: 10/02/2024
---

# Ingest data flow

This article describes several ways Azure Container Storage enabled by Azure Arc offers to control data flow to the cloud.

## Control options

One way to control data flow is by offering the ability to choose which data is uploaded first from the ingest subvolume to the cloud. As described in [Ingest policies](ingest-policies.md), the ingest order parameter is configurable for oldest data upload first or newest data upload first (FIFO or LIFO respectively).

Another control option is the ability to "throttle" data upload from an Ingest Edge Volume by limiting the number of active concurrent blob uploads. This throttling is accomplished by following these instructions:

1. Create an Edge Volume by following the instructions in [Create a Cloud Ingest Persistent Volume Claim (PVC)](/azure/azure-arc/container-storage/cloud-ingest-edge-volume-configuration?tabs=portal).
1. Run `kubectl get edgevolume` to find the name of the Edge Volume you created in the previous step.
1. Run `kubectl edit edgevolume <insert name from step 2>` to edit the Edge Volume.
1. Decide what you want your blob concurrency count to be; valid values are from 1 to 256.
1. Under spec, add a line that says:
   1. `ingestOperationConcurrency: <insert your desired blob concurrency count here>`
1. Save and close.  

When you update this information, any in-progress operations are disrupted, so it's recommended that you make this change at a time when disruption is acceptable.

> [!IMPORTANT]
> This throttling behavior is controlled at the Edge Volume level, not at the subvolume level. Therefore, every subvolume associated with the Edge Volume must share the user-specified number of concurrent blob uploads.

## Next steps

- [Ingest policies](ingest-policies.md)
- [Create a Cloud Ingest Persistent Volume Claim (PVC)](cloud-ingest-edge-volume-configuration.md)
