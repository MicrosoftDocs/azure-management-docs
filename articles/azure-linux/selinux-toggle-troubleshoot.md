---
title: Toggle SELinux Mode on Azure Container Linux (ACL) Node Pools for Troubleshooting
description: Guidance on how to switch SELinux (Security-Enhanced Linux) from enforcing mode to permissive mode on Azure Container Linux (ACL) nodes in Azure Kubernetes Service (AKS) for troubleshooting purposes, including identifying symptoms, understanding causes, and applying solutions.
author: flora-taagen
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 05/12/2026
---

# Toggle SELinux mode on Azure Container Linux (ACL) node pools for troubleshooting

This article shows you how to switch SELinux (Security-Enhanced Linux) from _enforcing mode_ to _permissive mode_ on Azure Container Linux (ACL) nodes in Azure Kubernetes Service (AKS) for troubleshooting purposes. This can be useful when you need to collect diagnostics or validate workload behavior that is being blocked by SELinux enforcement on ACL nodes.

## Symptom

You need to collect diagnostics or validate workload behavior, but SELinux enforcement blocks the action on ACL nodes.

## Cause

ACL nodes run SELinux in enforcing mode by default. To change behavior, you must apply the ACL security profile tag on the node pool.

## Prerequisites

- Azure CLI installed and signed in.
- You know your AKS cluster name, resource group, and node pool name.
- You can access a node for verification. For details, see [Connect to AKS nodes with kubectl debug](/azure/aks/node-access#connect-with-kubectl-debug).

## Resolution

To switch SELinux from enforcing mode to permissive mode on ACL nodes, you can either [**create a new ACL node pool**](#option-1-create-a-new-acl-node-pool-in-permissive-mode) or [**update an existing ACL node pool**](#option-2-update-an-existing-acl-node-pool) with the appropriate tag. The tag to set is `acl-node-security-profile=selinux=permissive`.

### Option 1: Create a new ACL node pool in permissive mode

1. Create a new ACL node pool in _permissive mode_ using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command with the `--tags` parameter to set the ACL security profile tag.

    ```azurecli-interactive
    az aks nodepool add \
      --resource-group <resource-group-name> \
      --cluster-name <cluster-name> \
      --name <node-pool-name> \
      --node-count 1 \
      --tags acl-node-security-profile="selinux=permissive" \
      --no-wait
    ```

1. Verify the tag was applied using the [`az aks show`](/cli/azure/aks#az-aks-show) command to check the node pool profiles and their tags.

    ```azurecli-interactive
    az aks show \
      --resource-group <resource-group-name> \
      --name <cluster-name> \
      --query "agentPoolProfiles[].{nodepoolName:name,tags:tags}"
    ```

    Example output:

    ```output
    [
      {
        "nodepoolName": "<node-pool-name>",
        "tags": {
          "acl-node-security-profile": "selinux=permissive"
        }
      }
    ]
    ```

### Option 2: Update an existing ACL node pool

1. Update an existing ACL node pool to _permissive mode_ using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command with the `--tags` parameter to set the ACL security profile tag.

    ```azurecli-interactive
    az aks nodepool update \
      --resource-group <resource-group-name> \
      --cluster-name <cluster-name> \
      --name <node-pool-name> \
      --tags acl-node-security-profile="selinux=permissive" \
      --no-wait
    ```

1. Verify the tag was applied using the [`az aks show`](/cli/azure/aks#az-aks-show) command to check the node pool profiles and their tags.

    ```azurecli-interactive
    az aks show \
      --resource-group <resource-group-name> \
      --name <cluster-name> \
      --query "agentPoolProfiles[].{nodepoolName:name,tags:tags}"
    ```

    Example output:

    ```output
    [
      {
        "nodepoolName": "<node-pool-name>",
        "tags": {
          "acl-node-security-profile": "selinux=permissive"
        }
      }
    ]
    ```

1. Restart the nodes in the node pool so the new SELinux mode takes effect.

## Verification

1. Connect to a node in the target pool using `kubectl debug` and chroot into the host filesystem.
1. Run the `sestatus` command to check the current SELinux mode:

    ```bash
    sestatus
    ```

    In the output, confirm `Current mode` shows `permissive`. For example:

    ```output
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   permissive
    Mode from config file:          permissive
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Memory protection checking:     actual (secure)
    Max kernel policy version:      33
    ```

## Related content

- [What is Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)?](./azure-container-linux-overview.md)
- [Tutorial: Add an ACL node pool to an existing cluster](./tutorial-add-acl-node-pool.md)
- [Tutorial: Migrate existing nodes to ACL](./tutorial-migrate-acl-aks.md)
