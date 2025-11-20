---
title: "Available extensions for Azure Arc-enabled Kubernetes clusters"
ms.date: 11/20/2025
ms.topic: how-to
description: "See a list of extensions that are currently available for Azure Arc-enabled Kubernetes clusters. View Flux extension release notes."
ms.custom:
  - build-2025
# Customer intent: "As a Kubernetes administrator, I want to explore and install available extensions for Azure Arc-enabled Kubernetes clusters, so that I can enhance cluster management and implement necessary functionalities efficiently."
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

## Azure Key Vault Secret Store

- **Supported distributions**: Arc-enabled Kubernetes clusters running Kubernetes 1.27 or later, including: AKS on Azure Local, AKS Edge Essentials, OpenShift Kubernetes Distribution, and VMware Tanzu Kubernetes Grid.

The Azure Key Vault Secret Store extension for Kubernetes (Secret Store) automatically syncs secrets from an instance of Azure Key Vault to a Kubernetes cluster for offline access. You can use Azure Key Vault to store, maintain, and rotate your secrets, even when you run your Kubernetes cluster in a semi-disconnected state.

We recommend the Secret Store extension for clusters at the edge where internet connectivity cannot be guaranteed, or if you need secrets synced to the Kubernetes secret store. For clusters in Azure cloud that do not require local secret storage, we recommend that you use the Azure Key Vault Secrets Provider extension instead.

For more information, see [Use the Secret Store extension to fetch secrets for offline access in Azure Arc-enabled Kubernetes clusters](secret-store-extension.md).

## Microsoft Defender for Containers

- **Supported distributions**: AKS enabled by Azure Arc, Cluster API Azure, Azure Red Hat OpenShift, Red Hat OpenShift (version 4.6 or later), Google Kubernetes Engine Standard, Amazon Elastic Kubernetes Service, VMware Tanzu Kubernetes Grid, Rancher Kubernetes Engine, and Canonical Kubernetes Distribution.

Microsoft Defender for Containers is the cloud-native solution that is used to secure your containers so you can improve, monitor, and maintain the security of your clusters, containers, and their applications. Microsoft Defender for Containers gathers information related to security, such as audit log data, from the Kubernetes cluster. Then, it provides recommendations and threat alerts based on the gathered data.

For more information, see [Enable Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-kubernetes-azure-arc?toc=/azure/azure-arc/kubernetes/toc.json&bc=/azure/azure-arc/kubernetes/breadcrumb/toc.json).

> [!IMPORTANT]
> Defender for Containers support for Azure Arc-enabled Kubernetes clusters is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Azure Arc-enabled Open Service Mesh

> [!IMPORTANT]
> Microsoft has announced the retirement of the [Open Service Mesh (OSM) add-on for AKS](https://azure.microsoft.com/updates?id=open-service-mesh-add-on-for-aks-will-be-retired-on-september-30-2027) on September 30, 2027. The upstream OSM project has also been retired by the [Cloud Native Computing Foundation (CNCF)](https://docs.openservicemesh.io/).

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

## Azure Kubernetes Fleet Manager

- **Supported distributions**: AKS (Azure Kubernetes Service), K3s (Lightweight Kubernetes), OCP (Red Hat OpenShift), EKS (Amazon Elastic Kubernetes Service), GKE (Google Kubernetes Engine), and Rancher (RKE).

Azure Kubernetes Fleet Manager is a comprehensive multi-cluster management solution that simplifies the process of managing clusters at scale and across hybrid environments. The extension is automatically installed when you join an Arc-enabled Kubernetes cluster to a Fleet.

For more information, see key concepts of [Azure Kubernetes Fleet Manager](/azure/kubernetes-fleet/concepts-fleet), and its [multicluster workload management](/azure/kubernetes-fleet/concepts-multi-cluster-workload-management), and supported [member cluster types](/azure/kubernetes-fleet/concepts-member-cluster-types).

> [!IMPORTANT]
> Azure Kubernetes Fleet Manager's extension for Arc-enabled Kubernetes clusters is in public preview. See **[important considerations for Arc-enabled Kubernetes cluster members](/azure/kubernetes-fleet/concepts-member-cluster-types#arc-enabled-kubernetes-clusters-important-considerations)** for a complete list of requirements and considerations.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

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

### Deprecation and removal notice: Upcoming changes to `microsoft.flux` extension

Several upstream Flux APIs that have been retired by the Flux project will be removed in an upcoming release of the `microsoft.flux` extension. These changes align with the Flux community's efforts to streamline and modernize the API surface.

The following Flux APIs are being deprecated and will be removed:

- Deprecated APIs in group `source.toolkit.fluxcd.io/v1beta1`
- Deprecated APIs in group `kustomize.toolkit.fluxcd.io/v1beta1`
- Deprecated APIs in group `helm.toolkit.fluxcd.io/v2beta1`
- Deprecated APIs in group `notification.toolkit.fluxcd.io/v1beta1`
- Deprecated APIs in group `image.toolkit.fluxcd.io/v1beta1`

For a complete list of deprecated APIs and their replacements, see the PRs linked in <https://github.com/fluxcd/flux2/issues/5474>.

**Required action:** To ensure continued compatibility and avoid disruptions, update your sources to remove references to deprecated APIs. Use the supported API versions for all impacted resources. We strongly recommend completing these steps before October 2025 to avoid deployment failures or resource reconciliation issues.

Migrate all your resources to the Flux stable APIs in your sources (Git repositories, OCI repositories, buckets, blob storage) by replacing the API version in the manifests:

-	`Kustomization` to `kustomize.toolkit.fluxcd.io/v1`
-	`HelmRelease` to `helm.toolkit.fluxcd.io/v2`
-	`Bucket` to `source.toolkit.fluxcd.io/v1`
-	`GitRepository` to `source.toolkit.fluxcd.io/v1`
-	`HelmChart` to `source.toolkit.fluxcd.io/v1`
-	`HelmRepository` to `source.toolkit.fluxcd.io/v1`
-	`OCIRepository` to `source.toolkit.fluxcd.io/v1`
-	`Receiver` to `notification.toolkit.fluxcd.io/v1`
-	`Alert` to `notification.toolkit.fluxcd.io/v1beta3`
-	`Provider` to `notification.toolkit.fluxcd.io/v1beta3`
-	`ImageRepository` to `image.toolkit.fluxcd.io/v1beta2`
-	`ImagePolicy` to `image.toolkit.fluxcd.io/v1beta2`
-	`ImageUpdateAutomation` to `image.toolkit.fluxcd.io/v1beta2`

Note that the `ImageUpdateAutomation` commit template should use the fields `.Changed.FileChanges`, `.Changed.Objects` and `.Changed.Changes` instead of the deprecated `.Updated` and `.Changed.ImageResult` fields.

Once the manifests are updated in the sources, Flux will reconcile the new API versions.

### `microsoft.flux` version 1.18.2 (October 2025)

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.3
- kustomize-controller: v1.6.1
- helm-controller: v1.3.1
- notification-controller: v1.6.0
- image-automation-controller: v0.41.2
- image-reflector-controller: v0.35.2

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller`, and `helm-controller` by updating the Go packages.

### `microsoft.flux` version 1.17.3 (September 2025)

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.2
- kustomize-controller: v1.6.1
- helm-controller: v1.3.0
- notification-controller: v1.6.0
- image-automation-controller: v0.41.2
- image-reflector-controller: v0.35.2

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller` and `fluent-bit-mdm` by updating the Go packages.

### `microsoft.flux` version 1.17.2 (August 2025)

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.2
- kustomize-controller: v1.6.1
- helm-controller: v1.3.0
- notification-controller: v1.6.0
- image-automation-controller: v0.41.2
- image-reflector-controller: v0.35.2

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller` and `fluent-bit-mdm` by updating the Go packages.

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

## Edge RAG Preview enabled by Azure Arc

- **Supported distributions**: AKS enabled by Azure Arc

Edge RAG Preview is an Azure Arc-enabled Kubernetes extension that enables you to search on-premises data with generative AI, using Retrieval Augmented Generation (RAG). RAG is an industry-standard architecture that augments the capabilities of a language model with private data. With Edge RAG, build custom chat assistants and derive insights from your private data.

For more information, see [What is Edge RAG?](../edge-rag/overview.md)

## Related content

- Read more about [cluster extensions for Azure Arc-enabled Kubernetes](conceptual-extensions.md).
- Learn how to [deploy extensions to an Azure Arc-enabled Kubernetes cluster](extensions.md).
