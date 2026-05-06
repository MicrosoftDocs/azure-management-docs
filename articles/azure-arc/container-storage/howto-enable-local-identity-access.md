---
title: Enable reduced-privilege identity access
description: Learn how to enable the localhost token endpoint for Azure Container Storage enabled by Azure Arc to reduce container privileges.
author: jswoodward
ms.author: jawoodwa
ms.topic: how-to
ms.date: 05/06/2026
ms.reviewer: sethm

# CustomerIntent: As a Kubernetes administrator, I want to reduce the security privileges required by the identity adapter sidecar in Azure Container Storage enabled by Azure Arc.
---

# Enable reduced-privilege identity access for Azure Container Storage enabled by Azure Arc

This article describes how to enable the localhost token endpoint for Azure Container Storage enabled by Azure Arc. When you use Managed Identity authentication for cloud-backed Edge Volumes, an identity adapter sidecar runs alongside the data mover in each volume pod. By default, this sidecar requires `NET_ADMIN` and `NET_RAW` Linux capabilities.

When you enable the localhost token endpoint, the identity adapter no longer requires these elevated network privileges. This change improves alignment with the Kubernetes [restricted Pod Security Standard](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted).

> [!IMPORTANT]
> Azure Container Storage enabled by Azure Arc is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites

- An Arc-connected Kubernetes cluster with Azure Container Storage enabled by Azure Arc, version 2.12.0 or later.
- Azure CLI with the **k8s-extension** extension installed.
- **kubectl** configured to access your cluster.

## Enable the localhost token endpoint at install time

Pass the `ENABLE_LOCALHOST_TOKEN_ENDPOINT` configuration setting when you create the extension:

```azurecli
az k8s-extension create --resource-group "<RESOURCE_GROUP>" \
    --cluster-name "<CLUSTER_NAME>" \
    --cluster-type connectedClusters \
    --name azure-arc-containerstorage \
    --extension-type Microsoft.Arc.ContainerStorage \
    --scope cluster \
    --release-namespace azure-arc-containerstorage \
    --configuration-settings ENABLE_LOCALHOST_TOKEN_ENDPOINT=True
```

> [!NOTE]
> Any values you set at installation time persist throughout the installation lifetime, including manual and auto upgrades.

## Enable the localhost token endpoint on an existing installation

If you already have Azure Container Storage installed, update the extension:

```azurecli
az k8s-extension update --resource-group "<RESOURCE_GROUP>" \
    --cluster-name "<CLUSTER_NAME>" \
    --cluster-type connectedClusters \
    --name azure-arc-containerstorage \
    --configuration-settings ENABLE_LOCALHOST_TOKEN_ENDPOINT=True
```

After the update, the operator automatically redeploys volume pods with the new configuration.

## Verify the configuration

After you enable the feature, confirm that the data mover container in your Edge Volume pods has the identity endpoint environment variables:

```bash
kubectl get pods -l app=wyvern -n azure-arc-containerstorage \
    -o json | jq '.items[].spec.containers[] | select(.name=="datamover") | .env[] | select(.name=="IDENTITY_ENDPOINT" or .name=="IDENTITY_HEADER")'
```

You should see output similar to the following, indicating that the identity adapter is configured to use the localhost token endpoint:

```output
{
  "name": "IDENTITY_ENDPOINT",
  "value": "http://127.0.0.1:8421/metadata/identity/oauth2/token"
}
{
  "name": "IDENTITY_HEADER",
  "value": "ArcK8s"
}
```

If these environment variables are present, the identity adapter is using the localhost endpoint and no longer requires elevated network privileges.

## Need more help?

If the identity adapter sidecar shows errors after you enable this feature, verify that your extension version is 2.12.0 or later:

```azurecli
az k8s-extension show --resource-group "<RESOURCE_GROUP>" \
    --cluster-name "<CLUSTER_NAME>" \
    --cluster-type connectedClusters \
    --name azure-arc-containerstorage \
    --query version
```

If the issue persists, [follow the instructions here](support-feedback.md) to create a support ticket.

## Next steps

- [Azure Container Storage enabled by Azure Arc overview](overview.md)
- [Configure Cloud Ingest subvolumes](howto-configure-cloud-ingest-subvolumes.md)
- [Configure Cloud Mirror subvolumes](howto-configure-cloud-mirror-subvolumes.md)
