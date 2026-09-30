---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title:       "Manage the Argo CD extension on AKS and Azure Arc-enabled Kubernetes"
description: How to article for managing the Argo CD extension on AKS and Azure Arc-enabled Kubernetes
author:      ponatara
ms.author:   ponatara
ms.topic:    how-to
ms.date:     09/25/2026
---

# Manage the Argo CD extension on AKS and Azure Arc-enabled Kubernetes

This article shows how to manage an existing Argo CD extension deployment. For installation, workload identity, private Azure Container Registry access, monitoring setup, migration, and first application deployment, see [Tutorial: Deploy applications using GitOps with Argo CD](tutorial-use-gitops-argocd.md).

## Prerequisites

Before you begin, ensure:

- Argo CD extension is installed on an AKS or Azure Arc-enabled Kubernetes cluster.

- Read and write permissions are configured on the cluster resource and `Microsoft.KubernetesConfiguration/extensions`.
- The latest Azure CLI and `k8s-extension` Azure CLI extensions are installed.

- `kubectl` configured to access the target cluster.

Set the following Bash variables. Use `managedClusters` for AKS or `connectedClusters` for Azure Arc-enabled Kubernetes.

```azurecli-interactive
RESOURCE_GROUP="my-resource-group"
CLUSTER_NAME="my-cluster"
CLUSTER_TYPE="managedClusters"
EXTENSION_NAME="argocd"
ARGOCD_NAMESPACE="argocd"
```

## Inspect extension status

View the extension provisioning state, installed version, and configuration.

```azurecli-interactive
az k8s-extension show \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --output yaml
```

Return only the provisioning state.

```azurecli-interactive
az k8s-extension show \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --query provisioningState \
  --output tsv
```

Verify that the Argo CD workloads are available in the cluster.

```bash
kubectl get deployments,pods --namespace "$ARGOCD_NAMESPACE"
```

## Configure Application namespaces

Use the `configs.params.application\.namespaces` setting to specify the namespaces in which Argo CD Application resources can be created.

The following example allows Application resources in the `default`, `argocd`, and `team-a` namespaces.

```azurecli-interactive
az k8s-extension update \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --config "configs.params.application\.namespaces=default,argocd,team-a"
```

| Scenario | Configuration value |
|---|---|
| Application resources only in the Argo CD namespace | `argocd` |
| Application resources in two namespaces | `argocd,team-a` |
| Application resources in multiple team namespaces | `argocd,team-a,team-b` |

After the update completes, verify the provisioning state.

```azurecli-interactive
az k8s-extension show \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --query provisioningState \
  --output tsv
```

> [!NOTE]
> The configured namespaces control where Argo CD Application resources can be defined. The destination namespace in an Application specification controls where that application's Kubernetes resources are deployed.

## Update extension settings

Use `az k8s-extension update` to change extension-managed Argo CD settings. The following example updates the URL used by Argo CD.

```azurecli-interactive
az k8s-extension update \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --config "configs.cm.url=https://argocd.contoso.com"
```

You can pass more than one setting in the same update operation.

```azurecli-interactive
az k8s-extension update \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --config "configs.cm.url=https://argocd.contoso.com" \
  --config "configs.params.application\.namespaces=argocd,team-a"
```

> [!IMPORTANT]
> Don't directly update Argo CD ConfigMaps. Apply configuration changes through the extension configuration API or the infrastructure-as-code template that manages the extension.

## Inspect application status

List Application resources across namespaces.

```bash
kubectl get applications.argoproj.io --all-namespaces
```

View the synchronization and health status of a specific Application.

```bash
APPLICATION_NAME="my-application"
APPLICATION_NAMESPACE="argocd"

kubectl get application "$APPLICATION_NAME" \
  --namespace "$APPLICATION_NAMESPACE" \
  --output jsonpath='{.status.sync.status}{"\t"}{.status.health.status}{"\n"}'
```

Review Application conditions, events, and recent status details.

```bash
kubectl describe application "$APPLICATION_NAME" \
  --namespace "$APPLICATION_NAMESPACE"
```

## Troubleshoot the extension

### Extension provisioning doesn't succeed

Inspect the extension resource and review `provisioningState`, error details, and configuration values.

```azurecli-interactive
az k8s-extension show \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --output yaml
```

Then inspect Kubernetes events and workloads in the extension namespace.

```bash
kubectl get events \
  --namespace "$ARGOCD_NAMESPACE" \
  --sort-by=.lastTimestamp

kubectl get deployments,pods \
  --namespace "$ARGOCD_NAMESPACE"
```

Check whether an admission policy blocks creation of the Argo CD namespace or workloads.

### Installation times out on a small cluster

High availability is enabled by default and requires at least four nodes. For a smaller test cluster, disable Redis HA.

```azurecli-interactive
az k8s-extension update \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --config "redis-ha.enabled=false"
```

### Application synchronization fails

Inspect the Application conditions and controller workloads.

```bash
kubectl describe application "$APPLICATION_NAME" \
  --namespace "$APPLICATION_NAMESPACE"

kubectl get pods \
  --namespace "$ARGOCD_NAMESPACE"
```

Confirm that the cluster has outbound access to:

- The configured repository on TCP port 22 for SSH or TCP port 443 for HTTPS.
- `https://management.azure.com`
- `https://<region>.dp.kubernetesconfiguration.azure.com`
- `https://login.microsoftonline.com`
- `https://mcr.microsoft.com`

For extension-platform diagnostics, see [Troubleshoot extension issues for Azure Arc-enabled Kubernetes clusters](extensions-troubleshooting.md).

## Delete the extension

Before deleting the extension, review the applications, repository credentials, and cluster registrations that depend on it.

```azurecli-interactive
az k8s-extension delete \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --yes
```

## Related content

- [Quickstart: Deploy an application using GitOps with Argo CD](quickstart-deploy-gitops-argo-cd.md)
- [Tutorial: Deploy applications using GitOps with Argo CD](tutorial-use-gitops-argocd.md)

