---
title: What's new for AKV Secrets Store extension
description: The release notes identify important updates and improvements in the Azure Key Vault Secrets Store extension.
ms.date: 08/01/2025
---

## August 2025
### V1.0
* First GA release of AKV SSE! 🎉

## May 2025
### V0.1
- The controller will now attempt to partially sync any secrets, instead of failing to sync any secret if at least one failed.
- Installation is now possible on OpenShift without need to configure Security Context Constraints.
- A ValidatingAdmissionPolicy has been added to prevent the SecretSync type from being changed.
- Security updates to internal components
    - Bump Go to 1.24.3
    - Bump Kubectl to v1.30.12
    - Bump provider to v1.7.0
    - Bump golang.org/x/net to v0.39.0
    - Azure Linux 3 now used as base for controller.

### V0.9.6
- Security updates to internal components
    - Bump Go to 1.24.3
    - Bump Kubectl to v1.30.12
    - Bump provider to v1.7.0
    - Bump golang.org/x/net to v0.39.0
    - Azure Linux 3 now used as base for controller.





