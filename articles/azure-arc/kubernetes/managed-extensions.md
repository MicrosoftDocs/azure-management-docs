---
title: "Version-managed extensions (preview) for Arc-enabled Kubernetes"
ms.date: 05/05/2025
ms.topic: overview
description: "Version-managed extensions (preview) for Arc-enabled Kubernetes adds efficiency by helping your extensions work better together."
ms.custom: references_regions
---

# Version-managed extensions (preview) for Arc-enabled Kubernetes

Version-managed extensions (preview) makes it easier to build, deploy, and manage applications on Arc-enabled Kubernetes clusters. The version-managed extensions are validated to reduce version incompatibility errors across the extensions installed in the cluster. Supported extensions are updated in sync to avoid version conflicts.

> [!IMPORTANT]
> Version-managed extensions for Arc-enabled Kubernetes is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Version-managed extensions (preview) currently offers the following benefits:

- **Consistent releases and updates**: Using version-managed extensions ensures that supported services are continuously updated together, rather than individually. This means that developers benefit from regular updates and servicing, including monthly updates, major version updates, and critical security fixes. These updates are applied using a rolling update strategy, which minimizes downtime and disruption to workloads. This consistent update process ensures that the platform remains secure, reliable, and up to date.
- **Improved extension compatibility**: Because supported extensions are updated together, there's less risk of version incompatibility issues between extensions. This lets you focus on building and deploying applications without worrying about whether your extensions will work together seamlessly or whether particular versions are compatible with each other.

## Extension support

Currently, version-managed extensions (preview) provides support for the following extensions:

- [Azure Container Storage enabled by Azure Arc](/azure/azure-arc/container-storage/overview)
- [Azure Key Vault Secret Store extension for Kubernetes ("SSE")](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s)

Version-managed extensions (preview) for Arc-enabled Kubernetes is currently available in the following regions: East US, East US 2, West US, West US 2, West US 3, West Europe, North Europe.

## Enable version-managed extensions

During the preview period, enabling version-managed extensions on your cluster requires you to use version 1.24.4 or later of the [Arc-enabled Kubernetes agents](release-notes.md) and enable the `BundleFull` feature flag.

For Kubernetes clusters that are already [connected to Azure Arc](quickstart-connect-cluster.md), enable version-managed extensions (preview) by running the following commands.

1. Verify that your agent version is 1.24.4 or later:

   `az connectedk8s show  -g $RESOURCE_GROUP  -n $CLUSTER_NAME --query '{version:agentVersion}'`

   If the agent version is lower, upgrade your agents:

   `az connectedk8s upgrade -g $RESOURCE_GROUP  -n $CLUSTER_NAME --agent-version 1.24.4`

1. Enable the `BundleFull` feature flag:

   `az connectedk8s update -g $RESOURCE_GROUP -n $CLUSTER_NAME \ --auto-upgrade false --config extensionbundle.featureflag=BundleFull`

1. Verify that the feature flag is set to `BundleFull`:

   `kubectl get configmap azure-clusterconfig -n azure-arc -o jsonpath='{.data.EXTENSION_BUNDLE_ENABLED_FEATURE_FLAG}'`

Next, configure the extensions that you want to use.

### Azure Container Storage enabled by Azure Arc

After you enable version-managed extensions (preview), follow these steps to configure Azure Container Storage enabled by Azure Arc.

1. Install the Azure IoT Operations dependencies (`cert-manager` and `trust-manager`):

   ```azurecli
   az k8s-extension create -g $RESOURCE_GROUP -c $CLUSTER_NAME \
    --cluster-type connectedClusters \
    --extension-type "microsoft.iotoperations.platform" \
    --name azure-iot-operations-platform \
    --configuration-settings installCertManager=true \
    --release-train preview \
    --version 0.7.6 \
    --auto-upgrade false \
    --release-namespace cert-manager \
    --scope cluster
   ```

1. Deploy the Azure Container Storage enabled by Azure Arc extension:

   ```azurecli
   az k8s-extension create -g $RESOURCE_GROUP -c $CLUSTER_NAME \
    --cluster-type connectedClusters \
    --cluster-name $CLUSTER_NAME --cluster-type connectedClusters \
    --name azure-arc-containerstorage \
    --extension-type microsoft.arc.containerstorage \
    --release-train stable \
    --auto-upgrade false
   ```

For more information, see [What is Azure Container Storage enabled by Azure Arc?](/azure/azure-arc/container-storage/overview)

### Azure Key Vault Secret Store extension for Kubernetes

After you enable version-managed extensions (preview), deploy Azure Key Vault Secret Store extension for Kubernetes (SSE):

```azurecli
az k8s-extension create --cluster-name $CLUSTER_NAME \ 
  --cluster-type connectedClusters  \ 
  --extension-type microsoft.azure.secretstore \ 
  --resource-group $RESOURCE_GROUP
  --name azure-secret-store 
```

> [!IMPORTANT]
> Currently, during the preview period, you must specify the exact value `azure-secret-store` to install the SSE extension with version-managed extensions.

For more information, see [Use the Secret Store extension to fetch secrets for offline access in Azure Arc-enabled Kubernetes clusters](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s).