---
title: What's new for AKV Secret Store extension
description: The release notes identify important updates and improvements in the Azure Key Vault Secret Store extension.
ms.date: 08/01/2025
ms.topic: release-notes
---

# Release notes
Important updates and improvements to the Azure Key Vault Secret Store extension are listed here.

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





