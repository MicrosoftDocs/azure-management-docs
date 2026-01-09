---
title: Static Retain Workflow Guide
description: Learn how to create and destroy static workflows using retain policies.
author: sethmanheim
ms.author: sethm
ms.reviewer: stevenpepin
ms.date: 01/09/2026
ms.topic: concept-article
---

# Static retain workflow guide

Azure Container Storage enabled by Azure Arc provides persistent storage solutions for Kubernetes workloads running on edge clusters. When managing storage resources, you can choose between dynamic provisioning (where resources are automatically created) and static workflows (where you manually create and manage resources). This guide focuses on *static workflows with retain policies* - a manual approach that gives you granular control over the storage resource lifecycle while providing extra data protection through retention policies.

This guide explains how to create and destroy static workflows by using retain policies. Azure Container Storage enabled by Azure Arc provides persistent storage for Kubernetes workloads running on Azure Arc-enabled clusters through EdgeVolumes and CSI drivers.

## When to use static workflows

Use static workflows when you need:

- **Precise control** over storage resource creation and lifecycle.
- **Data persistence** beyond the lifecycle of applications.
- **Manual oversight** of storage provisioning in critical environments.
- **Custom storage configurations** that dynamic provisioning doesn't easily achieve.

## Key benefits of retain policies

The retain storage classes (`unbacked-retain-sc` and `cloud-backed-retain-sc`) provide:

- **Data protection**: Prevents accidental data loss when PVCs are deleted.
- **Resource preservation**: Keeps underlying storage intact for reuse or recovery.
- **Controlled cleanup**: Requires deliberate action to remove storage resources.
- **Compliance support**: Helps meet data retention requirements.  

## Overview: What is a static workflow?

A *static workflow* involves manually creating and managing the following Kubernetes resources:

- **EdgeVolume** - Custom resource that represents storage on the edge
- **PersistentVolume (PV)** - Kubernetes storage resource
- **PersistentVolumeClaim (PVC)** - Request for storage by applications

### What is a retain policy?

The retain policy (`unbacked-retain-sc` / `cloud-backed-retain-sc` storage class) ensures that when you delete a PVC, the process preserves the underlying PV and EdgeVolume instead of automatically deleting them. This policy prevents accidental data loss and provides extra safety for critical workloads.

## Prerequisites

Before creating static workflows, make sure you have the following prerequisites:

- **Azure Arc-enabled Kubernetes cluster** with extension installed
- **kubectl** access to the cluster
- **Appropriate RBAC permissions** to create EdgeVolumes, PVs, and PVCs
- **CSI driver** (`wyvern.csi.azure.com`) running on the cluster

Verify that the Arc extension is installed by running the following commands:

```bash
kubectl get pods -n azure-arc-containerstorage
kubectl get storageclass | grep -E '*backed-retain-sc'
```

## Create static workflows with retain

Follow these steps to create a static workflow using the retain storage class.

### Step 1: Create the EdgeVolume

First, create an EdgeVolume custom resource that defines the storage requirements:

```yaml
apiVersion: arccontainerstorage.azure.net/v1
kind: EdgeVolume
metadata:
  name: my-static-volume
  namespace: default
spec:
  requestedSize: 5Gi
```

Next, apply the EdgeVolume:

```bash
kubectl apply -f edgevolume.yaml
```

### Step 2: Create the PersistentVolume

Create a persistent volume (PV) that references the EdgeVolume and uses the retain storage class:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-static-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  storageClassName: "unbacked-retain-sc" # or "cloud-backed-retain-sc" for backed volumes
  claimRef:
    name: my-static-pvc
    namespace: default
  csi:
    driver: wyvern.csi.azure.com
    readOnly: false
    volumeHandle: v2#my-static-volume#my-static-pv
    volumeAttributes:
      edgevolume: my-static-volume
      volume-mode: unbacked # or "cloud_backed" for backed volumes
```

Apply the PV:

```bash
kubectl apply -f persistentvolume.yaml
```

### Step 3: Create the PersistentVolumeClaim

Create a persistent volume claim (PVC) that binds to your static PV:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-static-pvc
  namespace: azure-arc-containerstorage
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: "unbacked-retain-sc"
  volumeName: my-static-pv
```

Apply the PVC:

```bash
kubectl apply -f persistentvolumeclaim.yaml
```

### Step 4: Verify the workflow

Check that you properly created and bound all components:

```bash
# Check EdgeVolume status
kubectl get edgevolume my-static-volume -o yaml

# Check PV status
kubectl get pv my-static-pv

# Check PVC status
kubectl get pvc my-static-pvc

# Verify binding
kubectl describe pvc -n azure-arc-containerstorage my-static-pvc
```

The EdgeVolume shows `state: deployed` and the PVC shows `Status: Bound`.

## How the retain policy works

- **Normal deletion**: When you delete a PVC with the default storage class, the process automatically deletes both the PV and the EdgeVolume.
- **Retain policy**: When you use `unbacked-retain-sc`, deleting the PVC doesn't delete the PV or the EdgeVolume.
- **Bonding protection**: The extension implements bonding between the PV and the EdgeVolume. You must delete these resources together.

### Test the retain behavior

Delete the PVC to see the retain policy in action:

```bash
kubectl delete pvc -n azure-arc-containerstorage my-static-pvc
```

Verify that the process retains the resources.

```bash
# PV should still exist
kubectl get pv my-static-pv

# EdgeVolume should still exist
kubectl get edgevolume my-static-volume
```
## Destroy static workflows

To safely destroy static workflows that use the retain policy, follow these steps.

### Safe destruction process

Because of bonding protection, you must delete the PV and EdgeVolume at the same time. If you try to delete just one, the operation hangs.

#### Method 1: Simultaneous deletion (recommended)

Delete both the PV and EdgeVolume at the same time:

```bash
# Open two terminal windows and run these commands simultaneously
kubectl delete pv my-static-pv &
kubectl delete edgevolume my-static-volume &
wait
```

#### Method 2: Use a script

Create a script for safe deletion:

```bash
#!/bin/bash
PV_NAME="my-static-pv"
EV_NAME="my-static-volume"

echo "Deleting PV and EdgeVolume simultaneously..."
kubectl delete pv "${PV_NAME}" &
kubectl delete edgevolume "${EV_NAME}" &

# Wait for both deletions to complete
wait

echo "Verifying deletion..."
if kubectl get pv "${PV_NAME}" 2>/dev/null; then
    echo "WARNING: PV still exists"
else
    echo "✓ PV successfully deleted"
fi

if kubectl get edgevolume "${EV_NAME}" 2>/dev/null; then
    echo "WARNING: EdgeVolume still exists"
else
    echo "✓ EdgeVolume successfully deleted"
fi
```

### Precautions

Avoid the following actions.

- Don't delete just the PV:

  ```bash
  kubectl delete pv my-static-pv  # This will hang due to bonding protection
  ```

- Don't delete just the EdgeVolume:

  ```bash
  kubectl delete edgevolume my-static-volume  # This also hangs due to bonding protection
  ```

### Force deletion (emergency only)

If resources get stuck, you can force the deletion as a last resort:

```bash
# Force delete PV
kubectl patch pv my-static-pv -p '{"metadata":{"finalizers":null}}'
kubectl delete pv my-static-pv --force --grace-period=0

# Force delete EdgeVolume
kubectl patch edgevolume my-static-volume -p '{"metadata":{"finalizers":null}}'
kubectl delete edgevolume my-static-volume --force --grace-period=0
```

> [!WARNING]
> Force deletion should only be used as a last resort and might leave orphaned resources.

## Troubleshoot common issues

This section covers common issues encountered during static workflow creation and destruction.

### EdgeVolume stuck in pending state

EdgeVolume shows `state: pending`.

#### Solutions

- Check operator logs: `kubectl logs -n azure-arc-acstor -l app=config-operator`
- Verify storage class exists: `kubectl get storageclass unbacked-retain-sc`
- Check cluster node capacity.

### PVC not binding to PV

PVC shows `Status: Pending`.

#### Solutions

- Verify `volumeName` in PVC matches PV name exactly.
- Check that `claimRef` in PV matches PVC name and namespace.
- Ensure storage sizes match between PV and PVC.

### Deletion hangs

`kubectl delete` command never completes.

#### Solutions

- Ensure you're deleting PV and EdgeVolume simultaneously.
- Check for finalizers: `kubectl get pv my-static-pv -o yaml | grep finalizers`.
- Use force deletion if necessary (last resort).

### Permission denied errors

The following errors are caused by insufficient permissions.

#### Symptoms

You can't create EdgeVolumes or access CSI volumes.

#### Solutions

- Verify RBAC permissions for EdgeVolume resources.
- Check that service accounts have proper permissions.
- Ensure the CSI driver is running by using `kubectl get pods -n azure-arc-acstor`.

### Diagnostic commands

Check component status:

```bash
# Check all pods
kubectl get pods -n azure-arc-containerstorage

# Check logs
kubectl logs -n azure-arc-containerstorage -l app=config-operator --tail=100

# Check CSI driver status
kubectl get csidrivers

# Check storage classes
kubectl get storageclass
```

Check resource relationships:
```bash
# Get detailed PV information
kubectl describe pv my-static-pv

# Get EdgeVolume details
kubectl describe edgevolume my-static-volume

# Check PVC events
kubectl describe pvc my-static-pvc
```

## Best practices

This section describes best practices for managing static workflows with retain policies.

### Naming conventions

- Use descriptive names that indicate the workload and purpose.
- Include environment or namespace information.
- Example: `prod-web-app-data-volume`.

### Resource management

- Always create EdgeVolume first, then PV, then PVC.
- Use consistent sizing across all three resources.
- Document the relationships between resources.

### Deletion safety

- Always verify what data is stored before deletion.
- Use simultaneous deletion method for PV and EdgeVolume.
- Test deletion procedures in non-production environments first.

### Monitoring

- Monitor EdgeVolume status regularly.
- Set up alerts for stuck or failed volumes.
- Keep logs of creation and deletion operations.

### Backup considerations

- Retain policy provides deletion protection, not data backup.
- Implement separate backup strategies for critical data.
- Document recovery procedures.

## Example complete workflow

Here's a complete example demonstrating creation and destruction:

```yaml
# complete-static-workflow.yaml
---
apiVersion: arccontainerstorage.azure.net/v1
kind: EdgeVolume
metadata:
  name: webapp-data-volume
spec:
  requestedSize: 10Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: webapp-data-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  storageClassName: "unbacked-retain-sc"
  claimRef:
    name: webapp-data-pvc
    namespace: production
  csi:
    driver: wyvern.csi.azure.com
    readOnly: false
    volumeHandle: webapp-data-volume
    volumeAttributes:
      edgevolume: webapp-data-volume
      volume-mode: unbacked
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: webapp-data-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: ""
  volumeName: webapp-data-pv
```

Deploy and verify:

```bash
# Create the workflow
kubectl apply -f complete-static-workflow.yaml

# Verify deployment
kubectl get edgevolume webapp-data-volume -n production
kubectl get pv webapp-data-pv
kubectl get pvc webapp-data-pvc -n production

# When ready to destroy
kubectl delete pvc webapp-data-pvc -n production
kubectl delete pv webapp-data-pv &
kubectl delete edgevolume webapp-data-volume -n production &
wait
```

## Summary

This guide provided comprehensive coverage of static workflows with retain functionality, ensuring safe creation and management of storage resources while avoiding common pitfalls.

## Next steps

- [Azure Container Storage enabled by Azure Arc overview](overview.md)
