---
title: "Available extensions for Azure Arc-enabled Kubernetes clusters"
ms.date: 04/25/2025
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

## Azure Container Apps on Azure Arc and Azure Logic Apps Hybrid

- **Supported distributions**: AKS, AKS on Azure Local, Azure Red Hat OpenShift, Google Kubernetes Engine, and OpenShift Container Platform.

Use this extension to provision an Azure Container Apps Connected Environment and Container Apps on top of an Azure Arc-enabled Kubernetes cluster.  This extension also enables the [Logic Apps Hybrid Deployment Model (public preview)](/azure/logic-apps/set-up-standard-workflows-hybrid-deployment-requirements).

For more information, see [Azure Container Apps on Azure Arc (Preview)](/azure/container-apps/azure-arc-overview).

> [!IMPORTANT]
> Azure Container Apps on Azure Arc is currently in public preview. Review the [public preview limitations](/azure/container-apps/azure-arc-overview#public-preview-limitations) before you deploy this extension.  This extension can't be installed on the same cluster as the Application services extension. If installed, the Application services extension must be removed before deploying this extension.
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

- **Supported distributions**: All CNCF-certified Kubernetes clusters. Not currently supported for Arm64 architectures.

Use the Azure Machine Learning extension to deploy and run Azure Machine Learning on an Azure Arc-enabled Kubernetes cluster.

For more information, see [Introduction to the Kubernetes compute target in Azure Machine Learning](/azure/machine-learning/how-to-attach-kubernetes-anywhere) and [Deploy the Azure Machine Learning extension on an AKS or Arc Kubernetes cluster](/azure/machine-learning/how-to-deploy-kubernetes-extension).

## ArgoCD (GitOps)

- **Supported distributions**: All CNCF-certified Kubernetes clusters.

The ArgoCD (GitOps) extension (preview) lets you use your Git repository as the source of truth for cluster configuration and application deployment.

For more information, see [Tutorial: Deploy applications using GitOps with ArgoCD](tutorial-use-gitops-argocd.md).

> [!IMPORTANT]
> ArgoCD (GitOps) is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability (GA).

## Flux (GitOps)

- **Supported distributions**: All CNCF-certified Kubernetes clusters.

[GitOps on AKS and Azure Arc-enabled Kubernetes](conceptual-gitops-flux2.md) can be enabled through [Flux v2](https://fluxcd.io/docs/), a popular open-source tool set, to help manage cluster configuration and application deployment. With the Flux extension, GitOps is enabled in the cluster as a `Microsoft.KubernetesConfiguration/extensions/microsoft.flux` cluster extension resource.

For more information, see [Tutorial: Deploy applications using GitOps with Flux v2](tutorial-use-gitops-flux2.md).

The most recent version of the Flux v2 extension and the two previous versions (N-2) are supported. We generally recommend that you use the most recent version of the extension.

> [!NOTE]
> When a new version of the `microsoft.flux` extension is released, it might take several days for the new version to become available in all regions.

### Breaking change: Semantic versioning changes in source controller

The `source-controller` recently updated its dependency on the "`github.com/Masterminds/semver/v3`" Go package from version v3.3.0 to v3.3.1. This update changed semantic versioning (semver) validation rules.

**What changed?** In the latest version (v3.3.1) of the semver package, certain version formats that were previously considered valid are now being rejected. Specifically, version strings with leading zeroes in numeric segments (e.g., 1.0.029903) are no longer accepted as valid semver.

- GitHub Issue for reference: [Previously supported chart version numbers are now invalid – fluxcd/source-controller #17380](https://github.com/Masterminds/semver/compare/v3.3.0...v3.3.1)
- Package change log: [Comparing v3.3.0...v3.3.1 · Masterminds/semver](https://github.com/Masterminds/semver/compare/v3.3.0...v3.3.1)

:::image type="content" source="media/flux-breaking-change.png" lightbox="media/flux-breaking-change.png" alt-text="Screenshot with examples of version formats that are now rejected.":::

**Impact on users:**

- **Existing deployments are unaffected**. Anything currently deployed will continue to function as expected.
- **Future deployments or reconciliations may fail** if they rely on chart versions that don’t follow the stricter semver rules.
- A common error you might see: `invalid chart reference: validation: chart.metadata.version "1.0.029903" is invalid`

**What you should do:** Review your chart versions and ensure they comply with proper semantic versioning. Avoid leading zeroes in version components, and follow the [semver.org](https://semver.org/) specification closely.

### 1.16.2 (March 2025)

Flux version: [Release v2.5.1](https://github.com/fluxcd/flux2/releases/tag/v2.5.1)

- source-controller: v1.5.0
- kustomize-controller: v1.5.1
- helm-controller: v1.2.0
- notification-controller: v1.5.0
- image-automation-controller: v0.40.0
- image-reflector-controller: v0.34.0

Changes in this version include:

- Addressed security vulnerabilities in the `fluxconfig-agent`, `fluxconfig-controller` and `fluent-bit-mdm` by updating the Go packages.
- Can now specify tenant ID when enabling [workload identity in Arc-enabled Kubernetes clusters and AKS clusters](tutorial-use-gitops-flux2.md#workload-identity-in-arc-enabled-kubernetes-clusters-and-aks-clusters).
- Support for image-automation controller in [workload identity in Arc-enabled Kubernetes clusters and AKS clusters](tutorial-use-gitops-flux2.md#workload-identity-in-arc-enabled-kubernetes-clusters-and-aks-clusters).

Breaking changes:

- Semantic versioning changes in source controller (see note above)

### 1.15.1 (February 2025)

Flux version: [Release v2.4.0](https://github.com/fluxcd/flux2/releases/tag/v2.4.0)

- source-controller: v1.4.1
- kustomize-controller: v1.4.0
- helm-controller: v1.1.0
- notification-controller: v1.4.0
- image-automation-controller: v0.39.0
- image-reflector-controller: v0.33.0

Changes in this version include:

- Addressed security vulnerabilities in the `fluxconfig-agent`, `fluxconfig-controller` and `fluent-bit-mdm` by updating the Go packages.
- Support of workload identity in Arc-enabled clusters. For more information, see [Workload identity in Arc-enabled Kubernetes clusters and AKS clusters](tutorial-use-gitops-flux2.md#workload-identity-in-arc-enabled-kubernetes-clusters-and-aks-clusters).

### 1.14.1 (January 2025)

Flux version: [Release v2.4.0](https://github.com/fluxcd/flux2/releases/tag/v2.4.0)

- source-controller: v1.4.1
- kustomize-controller: v1.4.0
- helm-controller: v1.1.0
- notification-controller: v1.4.0
- image-automation-controller: v0.39.0
- image-reflector-controller: v0.33.0

Changes in this version include:

- Addressed security vulnerabilities in the `fluxconfig-agent`, `fluxconfig-controller` and `fluent-bit-mdm` by updating the Go packages.
- Added support for [authentication against Azure DevOps repositories using workload identity for AKS clusters](tutorial-use-gitops-flux2.md#use-workload-identity-with-azure-devops).

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
