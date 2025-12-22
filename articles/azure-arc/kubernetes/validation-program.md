---
title: "Azure Arc-enabled Kubernetes validation"
ms.date: 09/26/2025
ms.topic: how-to
description: "Describes Arc validation program for Kubernetes distributions"
# Customer intent: As a DevOps engineer, I want to understand which Kubernetes distributions have passed conformance tests for Azure Arc, so that I can ensure compatibility and successful integration for managing my clusters across cloud environments.
---

# Azure Arc-enabled Kubernetes validation

The Azure Arc team works with key industry Kubernetes offering providers to validate Azure Arc-enabled Kubernetes with their Kubernetes distributions. Future major and minor versions of Kubernetes distributions released by these providers will be validated for compatibility with Azure Arc-enabled Kubernetes.

> [!IMPORTANT]
> Azure Arc-enabled Kubernetes works with any Kubernetes clusters that are certified by the Cloud Native Computing Foundation (CNCF), even if they are not listed on this page.

## Validated distributions

The following Microsoft-provided Kubernetes distributions and infrastructure providers successfully passed the conformance tests for Azure Arc-enabled Kubernetes:

| Distribution and infrastructure provider | Version |
| ---------------------------------------- | ------- |
| Cluster API Provider on Azure            | Release version: [1.20.1](https://github.com/kubernetes-sigs/cluster-api-provider-azure/releases/tag/v1.20.1); API version v1.10.4; Kubernetes version: [v1.33.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.33.1) |
| AKS Enabled by Azure Arc                 | Release version: [AKS on Azure Local version 2503](/azure/aks/aksarc/aks-whats-new-local#release-2503); Kubernetes version: [1.30.4](https://github.com/kubernetes/kubernetes/releases/tag/v1.30.4); [1.29.9](https://github.com/kubernetes/kubernetes/releases/tag/v1.29.9); [1.28.14](https://github.com/kubernetes/kubernetes/releases/tag/v1.28.14) |
| K8s on Azure Stack Edge                  | Release version: Azure Stack Edge 2501 (3.3.2501.1176); Kubernetes version: [1.29.4](https://github.com/kubernetes/kubernetes/releases/tag/v1.29.4) |
| AKS Edge Essentials                      | Release version [1.10.868.0](https://github.com/Azure/AKS-Edge/releases); Kubernetes version [1.29.9](https://github.com/kubernetes/kubernetes/releases/tag/v1.29.9) |

The following providers and their corresponding Kubernetes distributions successfully passed the conformance tests for Azure Arc-enabled Kubernetes:

| Provider name | Distribution name | Validated versions|
| ------------ | ----------------- | -------------------- |
| SUSE Rancher | [Rancher Kubernetes Engine (RKE1/RKE2)](https://www.rancher.com/index.php/products/rke) | [v1.33.3-rc2+rke2r1](https://github.com/rancher/rke2/releases)<br>v1.32.7-rc2+rke2r1<br>v1.31.11-rc2+rke2r1 |
| SUSE Rancher      | [K3s](https://rancher.com/products/k3s/) | [K3S version v1.33.2+k3s1](https://github.com/k3s-io/k3s/releases/tag/v1.33.2%2Bk3s1)<br>[K3S version v1.32.3+k3s1](https://github.com/k3s-io/k3s/releases/tag/v1.32.3%2Bk3s1)<br> [K3S version v1.31.5+k3s1](https://github.com/k3s-io/k3s/releases/tag/v1.31.5%2Bk3s1) |
| Red Hat       | [OpenShift Container Platform](https://www.openshift.com/products/container-platform) | [4.19.4](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/release_notes/ocp-4-19-release-notes), [4.18.9](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/release_notes/ocp-4-18-release-notes),  [4.17.5](https://docs.redhat.com/en/documentation/openshift_container_platform/4.17/html/release_notes/ocp-4-17-release-notes),  |
| VMware       | [Tanzu Kubernetes Grid/vSphere Kubernetes Service](https://tanzu.vmware.com/kubernetes-grid) | VKS 3.3, TKr v1.32.0+vmware.6-fips, Upstream K8s 1.32<br>VKS 3.3, TKr v1.31.4+vmware.1-fips, Upstream K8s 1.31 |
| Canonical    | [Charmed Kubernetes](https://ubuntu.com/kubernetes/charmed-k8s/docs)| [1.34](https://ubuntu.com/kubernetes/charmed-k8s/docs/1.34/components), [1.33](https://ubuntu.com/kubernetes/docs/1.33/components), [1.32](https://ubuntu.com/kubernetes/charmed-k8s/docs/1.32/components)|
| Wind River | [Wind River Cloud Platform](https://www.windriver.com/products/cloud-platform) |Wind River Cloud Platform 24.09; Upstream K8s version: 1.30.6|

## Scenarios validated

The conformance tests run as part of the Azure Arc-enabled Kubernetes validation cover the following scenarios:

1. Connect Kubernetes clusters to Azure Arc:
    * Deploy Azure Arc-enabled Kubernetes agent Helm chart on cluster.
    * Agents send cluster metadata to Azure.

2. Configuration:
    * Create configuration on top of Azure Arc-enabled Kubernetes resource.
    * [Flux](https://docs.fluxcd.io/), needed for setting up [GitOps workflow](tutorial-use-gitops-flux2.md), is deployed on the cluster.
    * Flux pulls manifests and Helm charts from demo Git repo and deploys to cluster.

## Next steps

* [Learn how to connect an existing Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md)
* Learn about the [Azure Arc agents](conceptual-agent-overview.md) deployed on Kubernetes clusters when connecting them to Azure Arc.





