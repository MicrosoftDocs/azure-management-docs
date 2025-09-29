---
title: Ingest data flow
description: Learn ways to control data flow to the cloud in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: concept-article
ms.date: 07/18/2025
# Customer intent: "As a cloud architect, I want to configure data ingest options for Azure Container Storage, so that I can control the data flow to the cloud and optimize upload performance based on my organization’s needs."
---

# Ingest data flow

This article describes several ways Azure Container Storage enabled by Azure Arc offers to control data flow to the cloud.

## Control options

One way to control data flow is by offering the ability to choose which data is uploaded first from the ingest subvolume to the cloud. The ingest order parameter is configurable for oldest data upload first or newest data upload first (FIFO or LIFO respectively).

Another control option is the ability to "throttle" data upload from an Ingest Edge Volume by limiting the number of active concurrent blob uploads. This throttling is accomplished by following these instructions:

1. Create an Edge Volume by following the instructions in [Cloud Ingest Edge Volumes configuration](howto-configure-cloud-ingest-subvolumes.md).
1. Run `kubectl get edgevolume` to find the name of the Edge Volume you created in the previous step.
1. Run `kubectl edit edgevolume <insert name from step 2>` to edit the Edge Volume.
1. Decide what you want your blob concurrency count to be; valid values are from 1 to 256.
1. Under spec, add a line that says:
   1. `ingestOperationConcurrency: <insert your desired blob concurrency count here>`
1. Save and close.  

When you update this information, any in-progress operations are disrupted, so we recommend that you make this change at a time when disruption is acceptable.

> [!IMPORTANT]
> This throttling behavior is controlled at the Edge Volume level, not at the subvolume level. Therefore, every subvolume associated with the Edge Volume must share the user-specified number of concurrent blob uploads.

## Next steps

- [Cloud Ingest Edge Volumes configuration](howto-configure-cloud-ingest-subvolumes.md)
