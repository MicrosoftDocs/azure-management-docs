---
title: "Version-managed extensions (preview) for Arc-enabled Kubernetes"
ms.date: 06/17/2025
ms.topic: overview
description: "Version-managed extensions (preview) for Arc-enabled Kubernetes adds efficiency by helping your extensions work better together."
ms.custom:
  - references_regions
  - build-2025
---

# Version-managed extensions (preview) for Arc-enabled Kubernetes

Version-managed extensions (preview) makes it easier to build, deploy, and manage applications on Arc-enabled Kubernetes clusters. The version-managed extensions are validated to reduce version incompatibility errors across the extensions installed in the cluster. Supported extensions are updated in sync to avoid version conflicts.

> [!IMPORTANT]
> Version-managed extensions for Arc-enabled Kubernetes is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Version-managed extensions (preview) currently offers the following benefits:

- **Consistent releases and updates**: Using version-managed extensions ensures that supported services are continuously updated together, rather than individually. This means that developers benefit from regular updates and servicing, including monthly updates, major version updates, and critical security fixes. These updates are applied using a rolling update strategy, which minimizes downtime and disruption to workloads. This consistent update process ensures that the platform remains secure, reliable, and up to date.
- **Improved extension compatibility**: Because supported extensions are updated together, there's less risk of version incompatibility issues between extensions. This lets you focus on building and deploying applications without worrying about whether your extensions will work together seamlessly or whether particular versions are compatible with each other.

Version-managed extensions (preview) for Arc-enabled Kubernetes is currently available in the following regions: East US, East US 2, West US, West US 2, West US 3, West Europe, North Europe.

## Prerequisites

To use version-managed extensions, an Arc-enabled Kubernetes cluster needs version 1.24.4 or later of the [Azure Arc-enabled Kubernetes agents](conceptual-agent-overview.md), To verify that your agent version is 1.24.4 or later, run the following command:

`az connectedk8s show  -g $RESOURCE_GROUP  -n $CLUSTER_NAME --query '{version:agentVersion}'`

If the agent version is lower, [upgrade your agents manually](agent-upgrade.md):

`az connectedk8s upgrade -g $RESOURCE_GROUP  -n $CLUSTER_NAME --agent-version 1.24.4`

> [!TIP]
> We recommend upgrading to [the latest version of the Azure Arc-enabled Kubernetes agents](/azure/azure-arc/kubernetes/release-notes).

You must have version 2.70.0 or later of the Azure CLI to deploy version-managed extensions to your clusters.

## Deploy version-managed extensions (preview)

Currently, version-managed extensions (preview) provides support for the following extensions:

- [Azure Container Storage enabled by Azure Arc](/azure/azure-arc/container-storage/overview)
- [Azure Key Vault Secret Store extension for Kubernetes ("SSE")](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s)

To deploy and configure version-managed extensions (preview) for Arc-enabled Kubernetes, use the [az vme](/cli/azure/vme) command.

### Configure all available version-managed extensions

To deploy all currently supported version-managed extensions, you can use the `az vme install` command with the `--include all` option. This command installs all extensions that are part of the version-managed extensions (preview) program.

Currently, before running the command to deploy all currently supported extension, you must first install Azure IoT Operations dependencies (`cert-manager` and `trust-manager`), which are required for the Azure Container Storage enabled by Azure Arc extension:

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

To simultaneously deploy all extensions currently supported by version-managed extensions, run the following command:

```azurecli
az vme install --resource-group my-resource-group --cluster-name my-cluster --include all
```

Currently, this command installs the Azure Container Storage enabled by Azure Arc and the Azure Key Vault Secret Store extension for Kubernetes ("SSE") extensions.

### Configure Azure Container Storage enabled by Azure Arc

Follow these steps to configure Azure Container Storage enabled by Azure Arc.

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
   az vme install --resource-group my-resource-group --cluster-name my-cluster --include microsoft.arc.containerstorage
   ```

For more information, see [What is Azure Container Storage enabled by Azure Arc?](/azure/azure-arc/container-storage/overview)

### Configure Azure Key Vault Secret Store extension for Kubernetes

To deploy the Azure Key Vault Secret Store extension for Kubernetes ("SSE"), run the following command

```azurecli
az vme install --resource-group my-resource-group --cluster-name my-cluster --include microsoft.azure.secretstore
```

For more information, see [Use the Secret Store extension to fetch secrets for offline access in Azure Arc-enabled Kubernetes clusters](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s).
