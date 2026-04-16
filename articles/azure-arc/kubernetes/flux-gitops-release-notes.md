---
title: What's new for Flux v(GitOps) in Azure Arc enabled Kubernetes
description: The release notes identify important updates and improvements in the microsoft.flux extension and important notices about changes.
ms.date: 04/16/2026
ms.topic: release-notes
---

# Flux (GitOps) extension release notes

The Flux (GitOps) extension is updated on an ongoing basis. This article provides information about the most recent releases of the extension.

> [!IMPORTANT]
> To ensure continued compatibility and avoid disruptions, update your sources to [remove references to deprecated APIs](#deprecation-and-removal-notice-upcoming-changes-to-microsoftflux-extension) and ensure that clusters are running the latest version of the extension.

The [most recent version of the Flux (GitOps) extension](flux-gitops-release-notes.md) and the two previous versions (N-2) are supported. We generally recommend that you use the most recent version of the extension.

When a new version of the `microsoft.flux` extension is released, it can take several days for the new version to become available in all regions.

## April 2026 - `microsoft.flux` version 1.21.0

> [!IMPORTANT]
> If you don't have automatic extensions upgraded on your cluster, we strongly recommend upgrading manually to this version as soon as possible. Clusters must be running version 1.21.0 in order to subsequently upgrade to an upcoming extension release that will include the Flux 2.7 API version.

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.4-8
- kustomize-controller: v1.6.1-11
- helm-controller: v1.3.2-6
- notification-controller: v1.6.0-10
- image-automation-controller: v0.41.2-11
- image-reflector-controller: v0.35.2-10

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller` `kustomize-controller`, `notification-controller`, `image-automation-controller`, `image-reflector-controller` and `helm-controller` by updating the Go packages and base images.
- Migrated CRs in `etcd` storage for Flux CRDs that have deprecated API versions.
- Optimized logging to reduce logging footprint in `fluxconfig-agent` and `fluxconfig-controller`.

## March 2026 - `microsoft.flux` version 1.20.4

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.4-7
- kustomize-controller: v1.6.1-10
- helm-controller: v1.3.2-5
- notification-controller: v1.6.0-9
- image-automation-controller: v0.41.2-10
- image-reflector-controller: v0.35.2-9

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller` `kustomize-controller`, `notification-controller`, `image-automation-controller`, `image-reflector-controller` and `helm-controller` by updating the Go packages and base images.
- Workload identity support for notification controller.
- Ensure workload identity tenant ID and client ID settings areR correctly reflected in Flux controller deployments when updated.
- Compliance with deployment safeguards for AKS automatic.
- Object-level workload identity support for Flux controllers.

## February 2026 - `microsoft.flux` version 1.19.5

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.4
- kustomize-controller: v1.6.1
- helm-controller: v1.3.2
- notification-controller: v1.6.0
- image-automation-controller: v0.41.2
- image-reflector-controller: v0.35.2

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller`, and `helm-controller` by updating the Go packages and base images.

## Deprecation and removal notice: Upcoming changes to `microsoft.flux` extension

Several upstream Flux APIs that have been retired by the Flux project will be removed in upcoming releases of the `microsoft.flux` extension. These changes align with the Flux community's efforts to streamline and modernize the API surface.

The following Flux APIs are being deprecated and will be removed:

- Deprecated APIs in group `source.toolkit.fluxcd.io/v1beta1` and `source.toolkit.fluxcd.io/v1beta2`
- Deprecated APIs in group `kustomize.toolkit.fluxcd.io/v1beta1` and `kustomize.toolkit.fluxcd.io/v1beta2`
- Deprecated APIs in group `helm.toolkit.fluxcd.io/v2beta1` and `helm.toolkit.fluxcd.io/v2beta2`
- Deprecated APIs in group `notification.toolkit.fluxcd.io/v1beta1`
- Deprecated APIs in group `image.toolkit.fluxcd.io/v1beta1`

For more information, see <https://github.com/fluxcd/flux2/issues/5572>.

**Required action:** To ensure continued compatibility and avoid disruptions, update your sources to remove references to deprecated APIs as soon as possible. Use the supported API versions for all impacted resources. Ensure that all clusters are upgraded to use `microsoft.flux` version 1.21.0 so that they will be able to upgrade to the upcoming release that introduces the Flux 2.7 API version.

Migrate all your resources to the Flux stable APIs in your sources (Git repositories, OCI repositories, buckets, blob storage) by replacing the API version in the manifests:

- `Kustomization` to `kustomize.toolkit.fluxcd.io/v1`
- `HelmRelease` to `helm.toolkit.fluxcd.io/v2`
- `Bucket` to `source.toolkit.fluxcd.io/v1`
- `GitRepository` to `source.toolkit.fluxcd.io/v1`
- `HelmChart` to `source.toolkit.fluxcd.io/v1`
- `HelmRepository` to `source.toolkit.fluxcd.io/v1`
- `OCIRepository` to `source.toolkit.fluxcd.io/v1`
- `Receiver` to `notification.toolkit.fluxcd.io/v1`
- `Alert` to `notification.toolkit.fluxcd.io/v1beta3`
- `Provider` to `notification.toolkit.fluxcd.io/v1beta3`
- `ImageRepository` to `image.toolkit.fluxcd.io/v1beta2`
- `ImagePolicy` to `image.toolkit.fluxcd.io/v1beta2`
- `ImageUpdateAutomation` to `image.toolkit.fluxcd.io/v1beta2`

Note that the `ImageUpdateAutomation` commit template should use the fields `.Changed.FileChanges`, `.Changed.Objects` and `.Changed.Changes` instead of the deprecated `.Updated` and `.Changed.ImageResult` fields.

Once the manifests are updated in the sources, Flux will reconcile the new API versions.

## Related content

- Learn more about the [Flux (GitOps) extension](conceptual-gitops-flux2.md).
- Get started with the Flux (GitOps) extension by using our [tutorial](tutorial-use-gitops-flux2.md).