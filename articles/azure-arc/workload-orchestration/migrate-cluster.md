---
title: Migrate targets to a new cluster
description: Learn how to migrate workload orchestration targets and deployed solutions from an old cluster to a new Arc-enabled cluster.
author: nathmanish
ms.author: nathmanish
ms.topic: how-to
ms.date: 05/26/2026
# Customer intent: As a platform engineer, I want to migrate my workload orchestration targets and deployed solutions to a new cluster, so that I can recover from cluster failures or expand capacity without redeploying from scratch.
---

# Migrate targets to a new cluster

When you need to replace an existing cluster—whether due to a cluster failure, capacity expansion, or infrastructure upgrade—you can sync your workload orchestration targets and deployed solutions to a new Azure Arc-enabled cluster. This article walks you through the process of re-creating the target resources and solution deployments on the new cluster without requiring to redeploy each solution manually.

## Prerequisites

- Azure CLI with the `workload-orchestration` extension (v5.2.0 or later). To update, run:

  ```azurecli
  az extension add --upgrade --name workload-orchestration
  ```

- A new Arc-connected cluster with the workload orchestration extension installed. For setup instructions, see [Set up workload orchestration using CLI](set-up-workload-orchestration.md).
- The custom location on the new cluster must have the same Azure Resource Manager (ARM) ID as the original custom location.

## Verify the state of the old cluster

Before replacing the cluster, you can connect to the cluster and verify the existing targets and deployed solutions.

```azurecli
kubectl get targets -A
kubectl get solutions -A
kubectl get instances -A
```

Note the targets and solutions listed. These are the resources that will be synced to the new cluster.

## Set up the new cluster

1. Delete the existing cluster, Arc connection, and custom location.
1. Re-create the cluster, Arc connection, and custom location. Make sure that the Azure Resource Manager (ARM) ID for the custom location on the new cluster remains the same as the original. Use the same resource group, subscription, and custom location name when re-creating it. Follow the steps in [Prepare your Arc cluster](set-up-workload-orchestration.md#prepare-your-arc-cluster).

## Sync targets and solutions to the new cluster

Run the following command with the ARM ID of the custom location on the new cluster:

```azurecli
az workload-orchestration sync --custom-location "<custom-location-ARM-id>"
```

The command presents a list of all targets that are associated with the specified custom location. You can choose which targets to sync by either:

- Entering a comma-separated list of target names to sync specific targets.
- Pressing **Enter** without any input to sync all targets.

For staged solutions, the sync process reconfigures and generates a new revision before deploying.

## Verify the migration

After the sync completes, confirm that all targets and solutions were migrated successfully:

```azurecli
kubectl get targets -A
kubectl get solutions -A
kubectl get instances -A
```

You should see the same targets and solution revisions that existed on the old cluster, along with their corresponding instances.

## Related content

- [Set up workload orchestration using CLI](set-up-workload-orchestration.md)
- [Delete resources](delete-resources.md)
- [Troubleshooting guide](troubleshooting.md)
