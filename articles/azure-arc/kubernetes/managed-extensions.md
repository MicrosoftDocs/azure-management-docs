---
title: "Managed extensions (preview) for Arc-enabled Kubernetes"
ms.date: 04/18/2025
ms.topic: overview
description: "Managed extensions (preview) for Arc-enabled Kubernetes adds efficiency by helping ensure your extensions work well together."
---

# Managed extensions (preview) for Arc-enabled Kubernetes

Managed extensions (preview) makes it easier to build, deploy, and manage applications on Arc-enabled Kubernetes clusters. It simplifies the extension setup process, providing a consistent and validated environment that reduces the risk of errors. Using managed extensions simplifies the process of deploying extensions by automatically installing supporting files to your cluster, so that the extensions are ready to be enabled. It also ensures that supported services are updated in sync to avoid version conflicts.

> [!IMPORTANT]
> Managed extensions for Arc-enabled Kubernetes is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Managed extensions (preview) currently offers the following benefits:

- **Simplified development and deployment**: Developers can use cloud-native services across hybrid, edge, and multicloud environments. This helps applications can run consistently and securely, regardless of where they're deployed. The bundled package simplifies the setup process, providing a consistent and validated environment that reduces the risk of errors and makes it easier to get started and maintain their projects.
- **Consistent releases and updates**: The bundled package approach ensures that all services are continuously updated together, rather than individually. This means that developers benefit from regular updates and servicing, including monthly updates, major version updates, and critical security fixes. These updates are applied using a rolling update strategy, which minimizes downtime and disruption to workloads. This consistent update process ensures that the platform remains secure, reliable, and up to date.

During the preview period, enabling managed extensions on your cluster requires you to use version 1.24.4 of the [Arc-enabled Kubernetes agents](release-notes.md) and enable the `BundleFull` feature flag.

## Extension support

Currently, managed extensions (preview) provides support for the following extensions:

- [Azure Container Storage enabled by Azure Arc](/azure/azure-arc/container-storage/overview)
- [Azure Key Vault Secret Store extension for Kubernetes ("SSE")](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s)

Managed extensions (preview) for Arc-enabled Kubernetes is currently available in the following regions: East US, East US 2, West US, West US 2, West US 3, West Europe, North Europe.

## Enable managed extensions

For Kubernetes clusters that are already [connected to Azure Arc](quickstart-connect-cluster.md), enable managed extensions (preview) by running the following commands.

1. Verify that your agent version is 1.24.4:

   `az connectedk8s show  -g $RESOURCE_GROUP  -n $CLUSTER_NAME --query '{version:agentVersion}'`

   If the version isn't 1.24.4, upgrade your agents:

   `az connectedk8s upgrade -g $RESOURCE_GROUP  -n $CLUSTER_NAME --agent-version 1.24.4`

1. Enable the `BundleFull` feature flag:

   `az connectedk8s update -g $RESOURCE_GROUP -n $CLUSTER_NAME \ --auto-upgrade false --config extensionbundle.featureflag=BundleFull`

1. Verify that the feature flag is set to `BundleFull`:

   `kubectl get configmap azure-clusterconfig -n azure-arc -o jsonpath='{.data.EXTENSION_BUNDLE_ENABLED_FEATURE_FLAG}'`

Next, configure the extensions that you want to use.

### Azure Container Storage enabled by Azure Arc

After you enable managed extensions (preview), follow these steps to configure Azure Container Storage enabled by Azure Arc.

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

After you've enabled managed extensions (preview), deploy Azure Key Vault Secret Store extension for Kubernetes (SSE):

```azurecli
az k8s-extension create --cluster-name $CLUSTER_NAME \ 
  --cluster-type connectedClusters  \ 
  --extension-type microsoft.azure.secretstore \ 
  --resource-group $RESOURCE_GROUP
  --name azure_secret_store 
```

> [!IMPORTANT]
> When installing the SSE extension with managed extensions, you must use the name `azure_secret_store` rather than `ssarcextension`.

For more information, see [Use the Secret Store extension to fetch secrets for offline access in Azure Arc-enabled Kubernetes clusters](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s).