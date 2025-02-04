---
title: "Available extensions for Azure Arc-enabled Kubernetes clusters"
ms.date: 11/21/2024
ms.topic: how-to
description: "See a list of extensions that are currently available for Azure Arc-enabled Kubernetes clusters. View extension release notes."
---

# Available extensions for Azure Arc-enabled Kubernetes clusters

[Cluster extensions for Azure Arc-enabled Kubernetes](conceptual-extensions.md) provide an Azure Resource Manager-based experience to install and manage lifecycles for different Azure capabilities in your cluster. You can [deploy extensions to your clusters](extensions.md) to support different scenarios and to improve cluster management.

The following extensions are currently available to use with Azure Arc-enabled Kubernetes clusters. With one exception, all the extensions that are described in this article are [cluster-scoped](conceptual-extensions.md#extension-scope). Azure API Management on Azure Arc is namespace-scoped.

## Container insights in Azure Monitor

- **Supported distributions**: All Cloud Native Computing Foundation (CNCF)-certified Kubernetes clusters.

The Container insights feature in Azure Monitor gives you a view into the performance of workloads that are deployed on your Kubernetes cluster. Use this extension to collect memory and CPU utilization metrics from controllers, nodes, and containers.

For more information, see [Container insights for Azure Arc-enabled Kubernetes clusters](/azure/azure-monitor/containers/container-insights-enable-arc-enabled-clusters?toc=/azure/azure-arc/kubernetes/toc.json&bc=/azure/azure-arc/kubernetes/breadcrumb/toc.json).

## Azure Policy

Azure Policy extends [Gatekeeper](https://github.com/open-policy-agent/gatekeeper), an admission controller webhook for [Open Policy Agent](https://www.openpolicyagent.org/) (OPA). Use Gatekeeper with OPA to consistently apply centralized, at-scale enforcements and safeguards on your clusters.

For more information, see [Understand Azure Policy for Kubernetes clusters](/azure/governance/policy/concepts/policy-for-kubernetes?toc=/azure/azure-arc/kubernetes/toc.json&bc=/azure/azure-arc/kubernetes/breadcrumb/toc.json).

## Azure Key Vault Secrets Provider

- **Supported distributions**: AKS on Azure Local, AKS enabled by Azure Arc, Cluster API Azure, Google Kubernetes Engine, Canonical Kubernetes Distribution, OpenShift Kubernetes Distribution, Amazon Elastic Kubernetes Service, and VMware Tanzu Kubernetes Grid.

Use the Azure Key Vault Provider for Secrets Store CSI Driver to integrate an instance of Azure Key Vault as a secrets store with a Kubernetes cluster via a CSI volume. For Azure Arc-enabled Kubernetes clusters, you can install the Azure Key Vault Secrets Provider extension to fetch secrets.

For more information, see [Use the Azure Key Vault Secrets Provider extension to fetch secrets into Azure Arc-enabled Kubernetes clusters](tutorial-akv-secrets-provider.md).

## Secret Store

- **Supported distributions**: All CNCF-certified Kubernetes clusters that are connected to Azure Arc and running Kubernetes 1.27 or later.

The Azure Key Vault Secret Store extension for Kubernetes (Secret Store) automatically syncs secrets from an instance of Azure Key Vault to a Kubernetes cluster for offline access. You can use Azure Key Vault to store, maintain, and rotate your secrets, even when you run your Kubernetes cluster in a semi-disconnected state.

We recommend the Secret Store extension for scenarios that require offline access, or if you need secrets synced to the Kubernetes secret store. If you don't need to use these features, we recommend that you instead use the Azure Key Vault Secrets Provider extension.

For more information, see [Use the Secret Store extension to fetch secrets for offline access in Azure Arc-enabled Kubernetes clusters](secret-store-extension.md).

> [!IMPORTANT]
> Secret Store is currently in preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Microsoft Defender for Containers

- **Supported distributions**: AKS enabled by Azure Arc, Cluster API Azure, Azure Red Hat OpenShift, Red Hat OpenShift (version 4.6 or later), Google Kubernetes Engine Standard, Amazon Elastic Kubernetes Service, VMware Tanzu Kubernetes Grid, Rancher Kubernetes Engine, and Canonical Kubernetes Distribution.

Microsoft Defender for Containers is the cloud-native solution that is used to secure your containers so you can improve, monitor, and maintain the security of your clusters, containers, and their applications. Microsoft Defender for Containers gathers information related to security, such as audit log data, from the Kubernetes cluster. Then, it provides recommendations and threat alerts based on the gathered data.

For more information, see [Enable Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-kubernetes-azure-arc?toc=/azure/azure-arc/kubernetes/toc.json&bc=/azure/azure-arc/kubernetes/breadcrumb/toc.json).

> [!IMPORTANT]
> Defender for Containers support for Azure Arc-enabled Kubernetes clusters is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Azure Arc-enabled Open Service Mesh

- **Supported distributions**: AKS, AKS on Azure Local,  AKS enabled by Azure Arc, Cluster API Azure, Google Kubernetes Engine, Canonical Kubernetes Distribution, Rancher Kubernetes Engine, OpenShift Kubernetes Distribution, Amazon Elastic Kubernetes Service, and VMware Tanzu Kubernetes Grid.

[Open Service Mesh (OSM)](https://docs.openservicemesh.io/) is a lightweight, extensible, Cloud Native service mesh that allows users to uniformly manage, secure, and get out-of-the-box observability features for highly dynamic microservice environments.

For more information, see [Azure Arc-enabled Open Service Mesh](tutorial-arc-enabled-open-service-mesh.md).

## Azure Arc-enabled data services

- **Supported distributions**: AKS, AKS on Azure Local, Azure Red Hat OpenShift, Google Kubernetes Engine, Canonical Kubernetes Distribution, OpenShift Container Platform, and Amazon Elastic Kubernetes Service.

This extension makes it possible for you to run Azure data services on-premises, at the edge, and in public clouds by using Kubernetes and the infrastructure of your choice. This extension enables the *custom locations* feature, providing a way to configure Azure Arc-enabled Kubernetes clusters as target locations for deploying instances of Azure offerings.

For more information, see [Azure Arc-enabled data services](../data/create-data-controller-direct-prerequisites.md) and [Create custom locations](custom-locations.md#create-custom-location).

## Azure App Service on Azure Arc

- **Supported distributions**: AKS, AKS on Azure Local, Azure Red Hat OpenShift, Google Kubernetes Engine, OpenShift Container Platform.

Use this extension to provision an App Service Kubernetes environment on top of an Azure Arc-enabled Kubernetes cluster.

For more information, see [Set up an Azure Arc-enabled Kubernetes cluster to run App Service apps, function apps, and logic apps (Preview)](/azure/app-service/overview-arc-integration).

> [!IMPORTANT]
> App Service on Azure Arc is currently in public preview. Review the [public preview limitations for App Service Kubernetes environments](/azure/app-service/overview-arc-integration#public-preview-limitations) before you deploy this extension.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Azure Event Grid on Kubernetes

- **Supported distributions**: AKS, Red Hat OpenShift.

Event Grid is an event broker you can use to integrate workloads that use event-driven architectures. Use this extension to create and manage Event Grid resources such as topics and event subscriptions with Azure Arc-enabled Kubernetes clusters.

For more information, see [Event Grid on Kubernetes with Azure Arc (Preview)](/azure/event-grid/kubernetes/overview).

> [!IMPORTANT]
> Event Grid on Kubernetes with Azure Arc is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Azure API Management on Azure Arc

- **Supported distributions**: All CNCF-certified Kubernetes clusters.

With the integration between Azure API Management and Azure Arc on Kubernetes, you can deploy the API Management gateway component as an extension in an Azure Arc-enabled Kubernetes cluster. This extension is [namespace-scoped](conceptual-extensions.md#extension-scope), not cluster-scoped.

For more information, see [Deploy an Azure API Management gateway on Azure Arc (preview)](/azure/api-management/how-to-deploy-self-hosted-gateway-azure-arc).

> [!IMPORTANT]
> The API Management self-hosted gateway on Azure Arc is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability (GA).

## Azure Arc-enabled Machine Learning

- **Supported distributions**: All CNCF-certified Kubernetes clusters. Not currently supported for ARM64 architectures.

Use the Azure Machine Learning extension to deploy and run Azure Machine Learning on an Azure Arc-enabled Kubernetes cluster.

For more information, see [Introduction to the Kubernetes compute target in Azure Machine Learning](/azure/machine-learning/how-to-attach-kubernetes-anywhere) and [Deploy the Azure Machine Learning extension on an AKS or Arc Kubernetes cluster](/azure/machine-learning/how-to-deploy-kubernetes-extension).

## Flux (GitOps)

- **Supported distributions**: All CNCF-certified Kubernetes clusters.

[GitOps on AKS and Azure Arc-enabled Kubernetes](conceptual-gitops-flux2.md) uses [Flux v2](https://fluxcd.io/docs/), a popular open-source tool set, to help manage cluster configuration and application deployment. GitOps is enabled in the cluster as a `Microsoft.KubernetesConfiguration/extensions/microsoft.flux` cluster extension resource.

For more information, see [Tutorial: Deploy applications using GitOps with Flux v2](tutorial-use-gitops-flux2.md).

The most recent version of the Flux v2 extension and the two previous versions (N-2) are supported. We generally recommend that you use the most recent version of the extension.

> [!IMPORTANT]
> The release [Flux v2.3.0](https://fluxcd.io/blog/2024/05/flux-v2.3.0/) includes API changes to the HelmRelease and HelmChart APIs, with deprecated fields removed, and an updated version of the Kustomize package. An upcoming minor version update of the Microsoft Flux extension will include these changes, consistent with the upstream open-source software (OSS) Flux project.
>
> The [HelmRelease](https://fluxcd.io/flux/components/helm/helmreleases/) API will be promoted from `v2beta1` to `v2` (GA). The `v2` API is backward compatible with `v2beta1`, with the exception of these deprecated fields:
>
> - **`.spec.chart.spec.valuesFile`**: Replaced by `.spec.chart.spec.valuesFiles` in `v2`.
> - **`.spec.postRenderers.kustomize.patchesJson6902`**: Replaced by `.spec.postRenderers.kustomize.patches` in `v2`.
> - **`.spec.postRenderers.kustomize.patchesStrategicMerge`**: Replaced by `.spec.postRenderers.kustomize.patches` in `v2`.
> - **`.status.lastAppliedRevision`**: Replaced by `.status.history.chartVersion` in `v2`.
>
> The [HelmChart](https://fluxcd.io/flux/components/source/helmcharts/) API will be promoted from `v1beta2` to `v1` (GA). The `v1` API is backward compatible with `v1beta2`, with the exception of the `.spec.valuesFile` field, which is replaced by `.spec.valuesFiles`.
>
> The new fields are already available in the current version of the APIs. Use the new fields instead of the fields that will be removed in the upcoming release.
>
> The Kustomize package will be updated to v5.4.0. The version contains the following breaking changes:
>
> - [Kustomization build fails when the resources key is missing](https://github.com/kubernetes-sigs/kustomize/issues/5337)
> - [Components are now applied after generators and before transformers](https://github.com/kubernetes-sigs/kustomize/pull/5170) in [v5.1.0](https://github.com/kubernetes-sigs/kustomize/releases/tag/kustomize%2Fv5.1.0)
> - [Null YAML values are replaced by "null"](https://github.com/kubernetes-sigs/kustomize/pull/5519) in [v5.4.0](https://github.com/kubernetes-sigs/kustomize/releases/tag/kustomize%2Fv5.4.0)
>
> To avoid issues caused by breaking changes, we recommend that you update your manifest to ensure that your Flux configurations remain compliant with this release.

> [!NOTE]
> When a new version of the `microsoft.flux` extension is released, it might take several days for the new version to become available in all regions.

### 1.14.1 (January 2025)


> [!NOTE]
> The API version `2024-11-01` for `fluxconfiguration` is not yet released. Once this version is available, we'll add instructions on how to use AKS Workload Identity for Azure DevOps repositories.

Flux version: [Release v2.4.0](https://github.com/fluxcd/flux2/releases/tag/v2.4.0)

- source-controller: v1.4.1
- kustomize-controller: v1.4.0
- helm-controller: v1.1.0
- notification-controller: v1.4.0
- image-automation-controller: v0.39.0
- image-reflector-controller: v0.33.0

Changes in this version include:

- Addressed security vulnerabilities in the `fluxconfig-agent`, `fluxconfig-controller` and `fluent-bit-mdm` by updating the Go packages.
- Added support for authentication against Azure DevOps repositories using AKS Workload Identity for AKS clusters. Although these changes have been implemented in the agents, this feature won't be available for use until the API version `2024-11-01` is fully released.

### 1.13.1 (October 2024)

Flux version: [Release v2.4.0](https://github.com/fluxcd/flux2/releases/tag/v2.4.0)

- source-controller: v1.4.1
- kustomize-controller: v1.4.0
- helm-controller: v1.1.0
- notification-controller: v1.4.0
- image-automation-controller: v0.39.0
- image-reflector-controller: v0.33.0

Changes in this version include:

- Added support for the `--feature-gates=StrictPostBuildSubstitutions=true controller` flag to enable [strict post-build variable substitution](tutorial-use-gitops-flux2.md#strict-post-build-variable-substitution).
- Addressed security vulnerabilities in the `fluxconfig-agent` by updating the Go packages.

### 1.13.0 (October 2024)

Flux version: [Release v2.4.0](https://github.com/fluxcd/flux2/releases/tag/v2.4.0)

- source-controller: v1.4.1
- kustomize-controller: v1.4.0
- helm-controller: v1.1.0
- notification-controller: v1.4.0
- image-automation-controller: v0.39.0
- image-reflector-controller: v0.33.0

Changes in this version include:

- Implemented fix to retrieve certificates from the correct location, resolving failures that occurred after switching the image from Alpine to Mariner.

## Dapr extension for Azure Kubernetes Service (AKS) and Azure Arc-enabled Kubernetes

[Dapr](https://dapr.io/) is a portable, event-driven runtime that simplifies building resilient, stateless, and stateful applications that run in the cloud and edge and embrace the diversity of languages and developer frameworks. The Dapr extension eliminates the overhead of downloading Dapr tooling and manually installing and managing the runtime on your clusters.

For more information, see [Dapr extension for AKS and Azure Arc-enabled Kubernetes](/azure/aks/dapr).

## Azure AI Video Indexer

- **Supported distributions**: All CNCF-certified Kubernetes clusters.

Azure AI Video Indexer enabled by Arc runs video and audio analysis on edge devices. The solution is designed to run on an Azure Stack Edge profile, which is a heavy edge device. The solution supports many video formats, including MP4 and other common formats. It supports several languages in all basic audio-related models.

For more information, see [Try Azure AI Video Indexer enabled by Azure Arc](/azure/azure-video-indexer/azure-video-indexer-enabled-by-arc-quickstart).

## Azure Container Storage enabled by Azure Arc

- **Supported distributions**: All CNCF-certified Kubernetes clusters.

[Azure Container Storage enabled by Azure Arc](../container-storage/index.yml) is a first-party storage system that's designed for Azure Arc-connected Kubernetes clusters. You can deploy Azure Container Storage enabled by Azure Arc to write files to a 'ReadWriteMany' persistent volume claim (PVC), where they're transferred to Azure Blob Storage. Azure Container Storage enabled by Azure Arc offers a range of features to support Azure IoT operations and other Azure Arc features.

For more information, see [What is Azure Container Storage enabled by Azure Arc?](../container-storage/overview.md).

## Connected registry on Azure Arc-enabled Kubernetes

- **Supported distributions**: AKS enabled by Azure Arc, Kubernetes by using the kind tool.

Use the connected registry extension for Azure Arc to sync container images between your instance of Azure Container Registry and your on-premises Azure Arc-enabled Kubernetes cluster. You can deploy this extension to either a local cluster or to a remote cluster. The extension uses a sync schedule and window to ensure seamless syncing of images between the on-premises connected registry and the cloud-based instance of Azure Container Registry.

For more information, see [Connected registry for Azure Arc-enabled Kubernetes clusters](../../container-registry/quickstart-connected-registry-arc-cli.md).

## Related content

- Read more about [cluster extensions for Azure Arc-enabled Kubernetes](conceptual-extensions.md).
- Learn how to [deploy extensions to an Azure Arc-enabled Kubernetes cluster](extensions.md).
