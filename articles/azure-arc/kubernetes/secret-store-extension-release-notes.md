---
title: What's new for AKV Secret Store extension
description: The release notes identify important updates and improvements in the Azure Key Vault Secret Store extension.
ms.date: 08/01/2025
ms.topic: release-notes
---

# Release notes
Important updates and improvements to the Azure Key Vault Secret Store extension are listed here.

## August 2025
### 1.0.2
- SSE is no longer in preview from this version. 🎉
- Failure to find the SecretSync resource during a SecretSync reconciliation no longer causes an error.
- Security updates to internal components
    - Update Go to 1.24.4

## May 2025
### 0.10.0 [PREVIEW]
- The controller will now attempt to partially sync any secrets, instead of failing to sync any secret if at least one failed.
- Installation is now possible on OpenShift without need to configure Security Context Constraints.
- A ValidatingAdmissionPolicy has been added to prevent the SecretSync type from being changed.
- Security updates to internal components
    - Update Go to 1.24.3
    - Update Kubectl to v1.30.12
    - Update provider to v1.7.0
    - Update golang.org/x/net to v0.39.0
    - Azure Linux 3 is now used as the base for the controller image.

### 0.9.6 [PREVIEW]
- Security updates to internal components
    - Update Go to 1.24.3
    - Update Kubectl to v1.30.12
    - Update provider to v1.7.0
    - Update golang.org/x/net to v0.39.0
    - Azure Linux 3 is now used as the base for the controller image.





