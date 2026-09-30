---
title: "Quickstart: Deploy an application by using GitOps with Argo CD"
description: Install the Azure-managed Argo CD extension on an AKS or Azure Arc-enabled Kubernetes cluster, and deploy a sample application from Git.
ms.topic: quickstart
author: ponatara
ms.author: ponatara
ms.date: 09/17/2026
---

# Deploy an application using GitOps with Argo CD extension

In this quickstart, you install the Argo CD cluster extension and create an Argo CD Application that deploys the AKS store demo from Git.

## Prerequisites

You need one of the following target clusters:

- An Azure Arc-enabled Kubernetes cluster that's connected and running. You need read and write permissions on `Microsoft.Kubernetes/connectedClusters`.
- An AKS cluster that's running and uses managed identity. You need read and write permissions on `Microsoft.ContainerService/managedClusters`.

For either cluster type, you also need:

- Read and write permissions on `Microsoft.KubernetesConfiguration/extensions`.
- The latest Azure CLI. The source tutorial lists Azure CLI version 2.15 or later, but use the latest version.
- The latest `k8s-extension` and `k8s-configuration` Azure CLI extensions.
- `kubectl`. It's preinstalled in Azure Cloud Shell. To install it locally through Azure CLI, run `az aks install-cli`.
- Outbound access to the repository and required Azure endpoints. See [Network requirements](tutorial-use-gitops-argocd.md#network-requirements).

Run the following commands in Bash. Set `CLUSTER_TYPE` to `managedClusters` for AKS or `connectedClusters` for Azure Arc-enabled Kubernetes.

```azurecli-interactive
RESOURCE_GROUP="my-resource-group"
CLUSTER_NAME="my-cluster"
CLUSTER_TYPE="managedClusters"
EXTENSION_NAME="argocd"
```

Sign in and select the subscription that contains the cluster.

```azurecli-interactive
az login
az account set --subscription "my-subscription-id"
```

## Verify AKS managed identity

Skip this section for Azure Arc-enabled Kubernetes.

Check the AKS identity type.

```azurecli-interactive
az aks show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --query identity.type \
  --output tsv
```

If the command doesn't return `SystemAssigned` or `UserAssigned`, enable managed identity.

```azurecli-interactive
az aks update \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --enable-managed-identity
```

## Register resource providers

Register the resource providers used by the extension platform.

```azurecli-interactive
az provider register --namespace Microsoft.Kubernetes --wait
az provider register --namespace Microsoft.ContainerService --wait
az provider register --namespace Microsoft.KubernetesConfiguration --wait
```

## Install or update Azure CLI extensions

The `--upgrade` option installs each extension if it isn't present and updates it if an earlier version is installed.

```azurecli-interactive
az extension add --name k8s-extension --upgrade --yes
az extension add --name k8s-configuration --upgrade --yes
```

## Install the Argo CD extension

This quickstart disables Redis high availability so the extension can run on a cluster with fewer than four nodes. It allows Argo CD Application resources in the `argocd` namespace.

```azurecli-interactive
az k8s-extension create \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --extension-type Microsoft.ArgoCD \
  --config "redis-ha.enabled=false" \
  --config "configs.params.application\.namespaces=argocd"
```

> [!NOTE]
> For a production-oriented deployment, review high availability and workload identity in the [Argo CD deployment tutorial](tutorial-use-gitops-argocd.md). High availability is the default and requires at least four nodes.

Verify that the extension reached the `Succeeded` provisioning state.

```azurecli-interactive
az k8s-extension show \
  --resource-group "$RESOURCE_GROUP" \
  --cluster-name "$CLUSTER_NAME" \
  --cluster-type "$CLUSTER_TYPE" \
  --name "$EXTENSION_NAME" \
  --query provisioningState \
  --output tsv
```

## Get cluster credentials

For AKS, run:

```azurecli-interactive
az aks get-credentials \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --overwrite-existing
```

For Azure Arc-enabled Kubernetes, use your existing kubeconfig and confirm that `kubectl` points to the intended cluster.

```bash
kubectl config current-context
kubectl get nodes
```

## Deploy the sample application

Create an Argo CD Application in the `argocd` namespace.

```bash
kubectl apply -f - <<'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: aks-store-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/Azure-Samples/aks-store-demo.git
    targetRevision: HEAD
    path: kustomize/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated: {}
EOF
```

The example follows the source tutorial and tracks `HEAD`. For repeatable production deployments, pin `targetRevision` to a reviewed commit, tag, or branch according to your release process.

## Verify the application

Check the Argo CD Application resource.

```bash
kubectl get application aks-store-demo \
  --namespace argocd \
  --output wide
```

Inspect synchronization and health details.

```bash
kubectl describe application aks-store-demo \
  --namespace argocd
```


## Clean up resources

Delete the sample Application.

```bash
kubectl delete application aks-store-demo --namespace argocd
```


## Next steps

- [Configure and operate the Argo CD extension](how-to-manage-gitops-argo-cd-extension.md)
- [Review the complete deployment tutorial](tutorial-use-gitops-argocd.md)
