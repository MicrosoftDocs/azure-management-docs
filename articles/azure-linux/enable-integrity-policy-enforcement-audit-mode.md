---
title: Enable IPE in audit mode on Azure Container Linux (ACL) node pools
description: Learn how to enable Integrity Policy Enforcement (IPE) in audit mode on Azure Container Linux (ACL) node pools in Azure Kubernetes Service (AKS).
author: flora-taagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 10/09/2026
---

# Enable IPE in audit mode on Azure Container Linux (ACL) node pools

This article shows you how to enable Integrity Policy Enforcement (IPE) in audit mode on Azure Container Linux (ACL) nodes in Azure Kubernetes Service (AKS). In audit mode, IPE records policy violations without blocking the affected operations. 

## Symptom: IPE diagnostics are unavailable

You need to collect IPE diagnostics, but IPE is currently disabled.

## Cause: IPE is disabled by default on ACL nodes

IPE is disabled by default on ACL nodes. To change this behavior, you must set the `acl-node-security-profile` tag (the ACL security profile tag) on the node pool to `ipe=audit`.

## Prerequisites

- Azure CLI installed and signed in.
- You know your AKS cluster name, resource group, and node pool name.
- You can access a node for verification. For details, see [Connect to AKS nodes with kubectl debug](/azure/aks/node-access#connect-with-kubectl-debug).
- Your cluster isn't a network isolated cluster. IPE isn't currently available on network isolated AKS clusters.

## Resolution: create or update an ACL node pool

To enable IPE in audit mode on ACL nodes, you can either [**create a new ACL node pool**](#create-a-node-pool-with-ipe-in-audit-mode) or [**update an existing ACL node pool**](#enable-ipe-in-audit-mode-on-an-existing-node-pool) with the appropriate tag. The tag to set is `acl-node-security-profile=ipe=audit`. *Note: IPE isn't currently available when using Network Isolated Clusters on AKS.*

## Set the `acl-node-security-profile` tag on a node pool

Enable IPE in audit mode when you create a node pool or update an existing node pool. Set the ACL security profile tag to `acl-node-security-profile=ipe=audit`.

The `acl-node-security-profile` tag accepts the following IPE values:

| Value | Effect |
|---|---|
| `ipe=audit` | IPE records policy violations without blocking operations. |
| `ipe=disabled` | IPE is turned off. |
| `ipe=off` | Alias for `ipe=disabled`. |

Tag behavior:

- New nodes apply the IPE mode on their first boot.
- Nodes added by scaling out an existing node pool inherit the tag from that pool.
- New node pools don't inherit the tag from other node pools.
- Setting `--tags` at the cluster level doesn't apply the ACL security profile to node pools.
- The `--tags` parameter replaces all existing tags on the node pool.

### Create a node pool with IPE in audit mode

> [!NOTE]
> IPE audit mode isn't available on network isolated AKS clusters.

1. Create an ACL node pool by using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az-aks-nodepool-add) command. Set the `acl-node-security-profile` tag with the `--tags` parameter.

    ```azurecli-interactive
    az aks nodepool add \
      --resource-group <resource-group-name> \
      --cluster-name <cluster-name> \
      --name <node-pool-name> \
      --node-count 1 \
      --tags acl-node-security-profile="ipe=audit" \
      --no-wait
    ```

1. Verify that the tag was applied by using the [`az aks show`](/cli/azure/aks#az-aks-show) command.

    ```azurecli-interactive
    az aks show \
      --resource-group <resource-group-name> \
      --name <cluster-name> \
      --query "agentPoolProfiles[].{nodepoolName:name,tags:tags}"
    ```


### Enable IPE in audit mode on an existing node pool

1. Select the pool and inspect its tags.

   ```azurecli
   az aks show \
     --resource-group <resource-group-name> \
     --name <cluster-name> \
     --query "agentPoolProfiles[].{nodepoolName:name,tags:tags}"
   ```

1. Update an existing ACL node pool by using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command. Set the `acl-node-security-profile` tag with the `--tags` parameter.

    > [!IMPORTANT]
    > The `--tags` parameter replaces all existing Azure resource tags on the node pool. Include any other tags that you want to keep. To set more than one profile setting, separate them with commas inside the tag value, for example `acl-node-security-profile="selinux=enforcing,ipe=audit"`.

    ```azurecli-interactive
    az aks nodepool update \
      --resource-group <resource-group-name> \
      --cluster-name <cluster-name> \
      --name <node-pool-name> \
      --tags acl-node-security-profile="ipe=audit" \
      --no-wait
    ```

1. Verify that the tag was applied by using the [`az aks show`](/cli/azure/aks#az-aks-show) command.

    ```azurecli-interactive
    az aks show \
      --resource-group <resource-group-name> \
      --name <cluster-name> \
      --query "agentPoolProfiles[].{nodepoolName:name,tags:tags}"
    ```

1. Restart the nodes in the node pool so that the new IPE mode takes effect.

### Turn off IPE audit mode on an ACL node pool

To turn off IPE audit mode, set the `acl-node-security-profile` tag on the node pool to `ipe=disabled`. The value `ipe=off` is an equivalent alias.

1. Update the node pool by using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az-aks-nodepool-update) command.

    ```azurecli-interactive
    az aks nodepool update \
      --resource-group <resource-group-name> \
      --cluster-name <cluster-name> \
      --name <node-pool-name> \
      --tags acl-node-security-profile="ipe=disabled" \
      --no-wait
    ```

1. Restart the nodes in the node pool so that the new IPE mode takes effect.
1. Verify the change. Follow the steps in [Verify IPE audit events](#verify-ipe-audit-events) and confirm the values for the disabled mode.

## Verify IPE audit events

1. Connect to a node in the target pool by using `kubectl debug`, and then chroot into the host file system.

1. Check the active policy and enforcement setting:

   ```bash
   cat /sys/kernel/security/ipe/policies/acl_ipe_boot_policy/active; echo
   cat /sys/kernel/security/ipe/enforce; echo
   ```

   | Mode | `active` | `enforce` |
   |---|---|---|
   | Audit | `1` | `0` |
   | Disabled | `0` | `1` |

   Repeat for every node. Check both values; `enforce` alone doesn't prove
   activation. 

1. Review the kernel audit log for IPE events.

    ```bash
    journalctl -b --no-pager | grep 'ipe_op='
    ```

In audit mode, IPE logs policy violations but doesn't block the operations that caused them. You might see denials from AKS or Microsoft-managed components that are outside your control. Work is underway to address these violations.

## Related content

- [Azure Container Linux security features](./azure-container-linux-security-features.md)
- [Toggle SELinux mode for troubleshooting](./selinux-toggle-troubleshoot.md)
