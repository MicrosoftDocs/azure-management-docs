---
title: Blobs not uploading or mirroring when using Managed Identity
description: Learn how to troubleshoot an issue in which blobs aren't uploaded or mirrored when using Managed Identity.
author: sethmanheim
ms.author: sethm
ms.date: 10/23/2025
ms.topic: troubleshooting
ms.reviewer: stevenpepin
ms.lastreviewed: 10/23/2025
---

# Blobs not uploading or mirroring when using Managed Identity

When using Managed Identity for authentication, you might encounter issues where blobs fail to upload or mirror as expected. This article helps you identify and resolve common causes of blob upload and mirroring failures when using Managed Identity authentication.

## Symptoms

When you enable **MANAGED_IDENTITY** for an Azure Container Storage sub-volume, an MSI adapter sidecar is created within the **ACSA** pod. This sidecar service can sometimes become unhealthy, which stops the ACSA pod from uploading or downloading any data to Azure.

## Mitigation steps

To resolve issues with blob uploads or mirroring when using Managed Identity, follow these steps:

### Step 1: check the logs

Using the name of the edge volume:

```bash
$ kubectl get edgevolumes.arccontainerstorage.azure.net 
NAME               STATUS     STATUSDETAILS   SPACEAVAILABLE
myedgevolume       deployed                   93394M
```

Check the ACSA logs and look for the phrase `MSI adapter is not healthy` by running the following command:

```bash
$ kubectl logs -n azure-arc-containerstorage -l name=w-myedgevolume -c datamover | grep "MSI adapter is not healthy"
```

If the MSI adapter is unhealthy, the problem might involve the Arc Extension that provides the identity.

### Step 2: fix if you see "In clusterIdentityCRDInteraction status not populated"

If the MSI adapter container logs contain the message "In clusterIdentityCRDInteraction status not populated," one possible root cause is that an identity certificate was not deleted when re-installing the extension.

```bash
kubectl logs -n azure-arc-containerstorage -l name=w-myedgevolume -c msi-adapter
```

A common cause of the MSI adapter issue is that an old identity certificate was left behind when the Arc extension was re-installed or was otherwise initialized incorrectly. You can delete that secret and restart the operator to clear the problem.

> [!IMPORTANT]
> These steps assume you have kubectl access to the cluster and that the Arc extension is installed in the `azure-arc` namespace.

#### Find your extension name

Run the following command to find the name of your Azure Container Storage extension:

```bash
helm list -A | grep arccontainerstorage
```
You should see a line that looks like this:

```output
NAME               NAMESPACE                    REVISION   UPDATED                                 STATUS   CHART                       APP VERSION
myextensionname    azure-arc-containerstorage   1          2025-08-26 18:37:42.699657275 +0000 UTC deployed arccontainerstorage-0.1.0-4 0.1.0-4
```

Replace `myextensionname` with the name that appears for your extension.

#### Delete the stale certificate secret

Run the following command to delete the stale identity certificate secret:

```bash
kubectl delete secret -n azure-arc extension-identity-certificate-<extension-name>
```

For example:

```bash
kubectl delete secret -n azure-arc extension-identity-certificate-myextensionname
```

This command removes the leftover identity certificate.

#### Restart the ClusterIdentityOperator

Run the following command to delete the ClusterIdentityOperator pod, which will force it to restart and recreate the necessary resources:

```bash
kubectl delete pod -n azure-arc clusteridentityoperator-<pod-name>
```

You can find the current pod name using the following command:

```bash
kubectl get pods -n azure-arc | grep clusteridentityoperator
```

After deletion, the operator automatically restarts.

#### Verify that everything is back to normal

Run the following commands to verify that the operator pod is running and that the certificate secret has been recreated:

```bash
# Operator pod should be running
kubectl get pods -n azure-arc | grep clusteridentityoperator

# Certificate secret should be recreated
kubectl get secrets -n azure-arc extension-identity-certificate-<extension-name>
```

When the pod shows `Running` and the secret exists again, the MSI adapter should become healthy.

## Need more help?

If the issue persists after performing these steps, [follow the instructions here](support-feedback.md) to create a support ticket.