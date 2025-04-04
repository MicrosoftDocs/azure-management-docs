---
title: "Azure Arc-enabled Kubernetes validation"
ms.date: 01/07/2025
ms.topic: how-to
description: "Describes Arc validation program for Kubernetes distributions"
---

# Azure Arc-enabled Kubernetes validation

The Azure Arc team works with key industry Kubernetes offering providers to validate Azure Arc-enabled Kubernetes with their Kubernetes distributions. Future major and minor versions of Kubernetes distributions released by these providers will be validated for compatibility with Azure Arc-enabled Kubernetes.

> [!IMPORTANT]
> Azure Arc-enabled Kubernetes works with any Kubernetes clusters that are certified by the Cloud Native Computing Foundation (CNCF), even if they haven't been validated through conformance tests and are not listed on this page.

## Validated distributions

The following providers and their corresponding Kubernetes distributions have successfully passed the conformance tests for Azure Arc-enabled Kubernetes:

| Provider name | Distribution name | Version |
| ------------ | ----------------- | ------- |
| RedHat       | [OpenShift Container Platform](https://www.openshift.com/products/container-platform) |[4.15.0](https://docs.openshift.com/container-platform/4.15/release_notes/ocp-4-15-release-notes.html)|
| VMware       | [Tanzu Kubernetes Grid](https://tanzu.vmware.com/kubernetes-grid) |TKGm 2.3; upstream K8s v1.26.5+vmware.2|
| Canonical    | [Charmed Kubernetes](https://ubuntu.com/kubernetes)|[1.28](https://ubuntu.com/kubernetes/docs/1.28/components), [1.31](https://ubuntu.com/kubernetes/docs/1.31/components)|
| SUSE Rancher      | [K3s](https://rancher.com/products/k3s/) | [v1.27.4+k3s1](https://github.com/k3s-io/k3s/releases/tag/v1.27.4%2Bk3s1), [v1.26.7+k3s1](https://github.com/k3s-io/k3s/releases/tag/v1.26.7%2Bk3s1)|
| Wind River | [Wind River Cloud Platform](https://www.windriver.com/studio/operator/cloud-platform) |Wind River Cloud Platform 24.09; Upstream K8s version: 1.28.4|


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





