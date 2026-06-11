---
title: What's new for Flux (GitOps) in Azure Arc enabled Kubernetes
description: Learn about supported versions of the microsoft.flux extension, along with important changes and improvements.
ms.date: 06/11/2026
ms.topic: release-notes
---

# Flux (GitOps) extension release notes

The Flux (GitOps) extension is updated on an ongoing basis. This article provides information about the most recent releases of the extension.

> [!IMPORTANT]
> To ensure continued compatibility and avoid errors, update your sources to [remove references to deprecated APIs](#deprecation-and-removal-notice-changes-to-microsoftflux-extension) and ensure that clusters are running the latest version of the extension.

The [most recent version of the Flux (GitOps) extension](flux-gitops-release-notes.md) and the two previous versions (N-2) are supported. We generally recommend that you use the most recent version of the extension.

When a new version of the `microsoft.flux` extension is released, it can take several days for the new version to become available in all regions.

<a name='deprecation-and-removal-notice-upcoming-changes-to-microsoftflux-extension'></a>

## Deprecation and removal notice: Changes to `microsoft.flux` extension

Several upstream Flux APIs that were retired by the Flux project are removed in version 1.23.0 of the `microsoft.flux` extension. These changes align with the Flux community's efforts to streamline and modernize the API surface.

> [!CAUTION]
> If you don't update your sources to remove references to deprecated APIs, you might experience disruptions in Flux functionality and see error messages referencing a removed kind/API, such as:
>
> `Error Signature: no matches for kind "HelmRepository" in version "source.toolkit.fluxcd.io/v1beta1"`
>
> `Error Signature: no matches for kind "HelmRelease" in version "source.toolkit.fluxcd.io/v1beta1"`

The following Flux APIs have been removed:

- Deprecated APIs in group `source.toolkit.fluxcd.io/v1beta1` and `source.toolkit.fluxcd.io/v1beta2`
- Deprecated APIs in group `kustomize.toolkit.fluxcd.io/v1beta1` and `kustomize.toolkit.fluxcd.io/v1beta2`
- Deprecated APIs in group `helm.toolkit.fluxcd.io/v2beta1` and `helm.toolkit.fluxcd.io/v2beta2`
- Deprecated APIs in group `notification.toolkit.fluxcd.io/v1beta1`
- Deprecated APIs in group `image.toolkit.fluxcd.io/v1beta1`

For more information, see <https://github.com/fluxcd/flux2/issues/5572>.

**Required action:** To ensure continued compatibility and avoid disruptions, update your sources to remove references to deprecated APIs as soon as possible. Use the supported API versions for all impacted resources and upgrade your clusters to use `microsoft.flux` version 1.23.0, which introduces the Flux 2.7 API version. If your clusters are running 1.20.4 or lower, upgrade them to an interim version (1.21.0 - 1.22.2) first, and then upgrade to version 1.23.0.

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

## June 2026 - `microsoft.flux` version 1.23.0

> [!IMPORTANT]
> Clusters must be running version 1.21.0 or higher in order to subsequently upgrade to version 1.23.0, which includes the Flux 2.7 API version.

Flux version: [Release v2.7.5](https://github.com/fluxcd/flux2/releases/tag/v2.7.5)

- source-controller: v1.7.4-10
- kustomize-controller: v1.7.3-9
- helm-controller: v1.4.5-8
- notification-controller: v1.7.5-7
- image-automation-controller: v1.0.4-9
- image-reflector-controller: v1.0.4-6

Changes in this version include:

- Flux extension updates to v2.7.5 with new CRDs and controllers. A number of API versions have been removed. For more information, see the [API removal notice](#deprecation-and-removal-notice-changes-to-microsoftflux-extension).
- Allow configuring Flux controllers' Helm cache, index, and jitter interval percentage.
- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller` `kustomize-controller`, `notification-controller`, `image-automation-controller`, `image-reflector-controller` and `helm-controller` by updating the Go packages and base images.

## May 2026 - `microsoft.flux` version 1.22.2

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.4-19
- kustomize-controller: v1.6.1-20
- helm-controller: v1.3.2-11
- notification-controller: v1.6.0-16
- image-automation-controller: v0.41.2-19
- image-reflector-controller: v0.35.2-15

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller` `kustomize-controller`, `notification-controller`, `image-automation-controller`, `image-reflector-controller` and `helm-controller` by updating the Go packages and base images.

## May 2026 - `microsoft.flux` version 1.22.1

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.4-14
- kustomize-controller: v1.6.1-15
- helm-controller: v1.3.2-8
- notification-controller: v1.6.0-13
- image-automation-controller: v0.41.2-14
- image-reflector-controller: v0.35.2-12

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller` `kustomize-controller`, `notification-controller`, `image-automation-controller`, `image-reflector-controller` and `helm-controller` by updating the Go packages and base images.
- Made Flux extension pods compliant with pod security standards.
- Implemented design improvements for `WaitForReconciliation` logic in `fluxconfig-agent` and `fluxconfig-controller` to address bugs where provisioning state would get stuck in `Failed` state.
- Added support for cosign verification in `source-controller`.

## April 2026 - `microsoft.flux` version 1.21.1

Flux version: [Release v2.6.4](https://github.com/fluxcd/flux2/releases/tag/v2.6.4)

- source-controller: v1.6.4-13
- kustomize-controller: v1.6.1-15
- helm-controller: v1.3.2-8
- notification-controller: v1.6.0-12
- image-automation-controller: v0.41.2-14
- image-reflector-controller: v0.35.2-12

Changes in this version include:

- Addressed security vulnerabilities in `fluxconfig-agent`, `fluxconfig-controller`, `fluent-bit-mdm`, `source-controller` `kustomize-controller`, `notification-controller`, `image-automation-controller`, `image-reflector-controller` and `helm-controller` by updating the Go packages and base images.

## April 2026 - `microsoft.flux` version 1.21.0

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

## Related content

- Learn more about the [Flux (GitOps) extension](conceptual-gitops-flux2.md).
- Get started with the Flux (GitOps) extension by using our [tutorial](tutorial-use-gitops-flux2.md).