---
title: Set ingest policy
description: Learn how to set ingest policies in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 10/02/2024
# Customer intent: As a cloud administrator, I want to configure ingest policies for Azure Container Storage, so that I can manage the file ingestion and eviction processes tailored to my storage requirements.
---

# Set ingest policy

This article describes how to set ingest policies in Azure Container Storage enabled by Azure Arc. The ingest policy that you set for that subvolume determine the ingest characteristics of your subvolume.

## Ingest policy parameters

You can configure the following parameters. The following table also lists the default values if you don't edit the policy:

| Parameter                  | Description                                                                                                                                                                                          | Available values                                                                                                                              | Default            |
|----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|
| `spec.ingest.order`          | The order in which files written to the subvolume are ingested. This order is a best effort, not a guarantee.                                                                                            | `oldest-first`: the oldest files yet to be ingested are prioritized.<br/><br/> `newest-first`: the newest files yet to be ingested are prioritized.  | `oldest-first`             |
| `spec.ingest.minDelaySec`    | The minimum number of seconds after the last file handle is closed before the file is eligible for ingest.                                                                                            | Any integer value of seconds between 0 and 31536000 (one year).                                                                                | 60 seconds               |
| `spec.eviction.order`        | Once a file is ingested successfully, how the system evicts the local copy of that file.                                                                                                    | `unordered`: ingested files are evicted at some point after their `minDelaySec` elapses.<br/><br/>  `never`: ingested files are never evicted.      | `unordered`                |
| `spec.eviction.minDelaySec`  | The number of seconds after a file is ingested successfully before the system deletes the local copy of that file. This parameter has no effect if `spec.eviction.order` is set to `never`.  | Any integer value of seconds between 0 and 31536000 (one year).                                                                                | 300 seconds (5 minutes)  |

## Change ingest policy

If you want to change the ingest policy from the default **edgeingestpolicy-default**, create a file named **myedgeingest-policy.yaml** with the following contents:

```yaml
apiVersion: arccontainerstorage.azure.net/v1 
kind: EdgeIngestPolicy 
metadata: 
  name: <create-a-policy-name-here> # This must be updated and referenced in the spec.ingestPolicy section of the edgeSubvolume.yaml 
spec: 
  ingest: 
    order: <your-ingest-order> 
    minDelaySec: <your-min-delay-sec> 
  eviction: 
    order: <your-eviction-order> 
    minDelaySec: <your-min-delay-sec>
```

To apply **myedgeingest-policy.yaml**, run the following command:

```bash
kubectl apply -f "myedgeingest-policy.yaml"
```

You can then use this new ingest policy for new ingest subvolumes you create by putting its name in the `spec.ingestPolicy` field. You can also update the ingest policy of an existing subvolume by putting your newly created policy name in the `spec.ingestPolicy` field of that subvolume, and once you reapply the configuration for that subvolume, the policy updates.

## Next steps

[Cloud Ingest Edge Volumes configuration](cloud-ingest-edge-volume-configuration.md)
