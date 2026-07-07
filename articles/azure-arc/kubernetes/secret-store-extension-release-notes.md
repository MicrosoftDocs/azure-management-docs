---
title: What's new for AKV Secret Store extension
description: The release notes identify important updates and improvements in the Azure Key Vault Secret Store extension.
ms.date: 05/26/2026
ms.topic: release-notes
---

# Azure Key Vault Secret Store extension release notes
Updates and improvements to the Azure Key Vault Secret Store extension are listed here.

## June 2026
### 1.5.1

 - Patch release to update dependencies with known vulnerabilities.

## May 2026
### 1.5.0
 - Security updates to internal components:
   - Updated Go to 1.26.2.
   - Updated kubectl container image to v1.36.0-3.
   - Updated AKV CSI Provider container image to v1.8.1-1.
   - Bumped `sigs.k8s.io/secrets-store-csi-driver` to v1.6.0.
   - Bumped `sigs.k8s.io/controller-runtime` to v0.24.0.
   - Bumped `github.com/Azure/azure-sdk-for-go/sdk/azidentity` to v1.13.1.

## April 2026
### 1.4.1
 - Security updates to internal components:
   - Updated Go to 1.26.1.
   - Updated kubectl container image to v1.35.3-1 to address CVEs in base image.
   - Updated AKV CSI Provider container image to v1.7.2-6 to address CVEs in base image.
   - Bumped `helm.sh/helm/v3` to v3.20.2 to address security advisories.
   - Bumped `go.opentelemetry.io/otel/sdk` to v1.43.0 to address security advisories.
   - Bumped `google.golang.org/grpc` to v1.79.3 to address security advisories.

## March 2026
### 1.4.0
 - Added a `kubectl.image.tag` Helm value to configure the kubectl image tag, alongside `repository` and `digest`. Both tag and digest are now provided for provider and kubectl images.
 - Further refined ownership checks to prevent `AKVSync` resources from overwriting existing `SecretSync` or `SecretProviderClass` resources when using TLS certificates.
 - Reinstated support for Kubernetes versions prior to 1.30, which was unintentionally dropped in 1.3.0, by making the new Validating Admission Policies (VAPs) conditional.
 - Security updates to internal components:
   - Updated kubectl container image to v1.35.1-1.
   - Updated AKV CSI Provider container image to v1.7.2-5.
   - Bumped `sigs.k8s.io/secrets-store-csi-driver` to v1.5.6, `helm.sh/helm/v3` to v3.20.0, `golang.org/x/crypto` to v0.48.0, and `google.golang.org/protobuf` to v1.36.11.

### 1.3.0
 - Added ownership checks to prevent AKVSync resources from overwriting existing SecretSync or SecretProviderClass resources.
 - Security updates to internal components:
   - Bumped Golang version to 1.25.7 which includes CVE patches.
   - Bumped kubectl container image to v1.35.0-2 to address CVEs in base image.
   - Bump AKV CSI Provider container image revision to v1.7.2-4 to address CVEs in base image.

## February 2026
### 1.2.2
 - HTTP proxy certificates provided via the --proxy-cert flag during cluster Arc enablement are now correctly handled.

## January 2026
### 1.2.1
 - Support for ARM64 architectures.
 - The Helm chart now uses SHA-256 digests to identify image versions instead of tags.
 - Security updates to internal components:
     - Update Go to 1.25.5
     - Update kubectl container image to v1.35.0-1

### 1.1.6
 - The Helm chart now uses SHA-256 digests to identify image versions instead of tags.
 - Security updates to internal components:
     - Update Go to 1.25.5
     - Update kubectl container image to v1.35.0-1


## November 2025
### 1.1.5
- (Preview feature) Simplified configurations are supported via `AKVSync` resources. See the getting started and reference guides.
- `jitterSeconds` extension setting (see [configuration reference](secret-store-extension-reference.md#arc-extension-configuration-settings)) added to help large deployments avoid overwhelming Azure Key Vault.
- Security updates to internal components:

    - Update Go to 1.25.1.
    - Updated kubectl container image to v1.33.5-3.

## August 2025
### 1.0.2
- SSE is generally available.

- Failure to find the SecretSync resource during a SecretSync reconciliation no longer causes an error.
- Security updates to internal components:

    - Update Go to 1.24.4

## May 2025
### 0.10.0 [PREVIEW]
- The controller will now attempt to partially sync any secrets, instead of failing to sync any secret if at least one failed.
- Installation is now possible on OpenShift without need to configure Security Context Constraints.
- A ValidatingAdmissionPolicy has been added to prevent the SecretSync type from being changed.
- Security updates to internal components:

    - Update Go to 1.24.3
    - Update Kubectl to v1.30.12
    - Update provider to v1.7.0
    - Update golang.org/x/net to v0.39.0
    - Azure Linux 3 is now used as the base for the controller image.

### 0.9.6 [PREVIEW]
- Security updates to internal components:

    - Update Go to 1.24.3
    - Update Kubectl to v1.30.12
    - Update provider to v1.7.0
    - Update golang.org/x/net to v0.39.0
    - Azure Linux 3 is now used as the base for the controller image.





