---
title: Overview of Azure Linux Certifications and Compliance
description: Learn about the certifications, assessments, and compliance benchmarks that Azure Linux meets, including FIPS, CIS, STIG, and FedRAMP alignment.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux certifications and compliance

This article summarizes the certifications, assessments, compliance benchmarks, and supply chain validations relevant to Azure Linux security.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Locate compliance artifacts

1. Identify which certifications apply to your workload (for example, FIPS, CIS, or STIG).
1. Collect the public security policy and certification references for Azure Linux.
1. Share the artifacts with your compliance stakeholders.

## FIPS 140-3

Azure Linux supports FIPS 140-3 mode for cryptographic compliance. FIPS 140-3 certification enables adoption in government and regulated industry environments. For more information, see [FIPS 140-3 and cryptography in Azure Linux](./fips-cryptography.md).

## Center for Internet Security (CIS) benchmarks

Azure Linux supports Center for Internet Security (CIS) benchmark compliance at both Level 1 and Level 2:

- **CIS Level 1**: Baseline security configuration suitable for most environments. Focuses on essential security settings with minimal impact on system functionality.
- **CIS Level 2**: Extended security configuration for environments with stricter compliance requirements. Might limit some functionality in exchange for extra hardening.

Azure Linux ships with audit logging aligned to CIS Level 2 by default. For more information, see [Logging and auditing in Azure Linux](./logging-auditing.md).

## Security Technical Implementation Guide (STIG) assessments

Automated Security Technical Implementation Guide (STIG) assessments are completed for each release to validate baseline security controls. STIG profiles provide prescriptive security guidance used by U.S. Department of Defense and government organizations.

## FedRAMP alignment

Azure Linux security configurations are aligned with FedRAMP High requirements across multiple control families:

- **AC-3, AC-6**: SELinux enforcing mode for mandatory access control
- **AU-2, AU-12**: auditd with CIS Level 2 baseline ruleset
- **SC-12, SC-13**: FIPS 140-3 validated cryptographic modules
- **SI-7**: Signed packages, SBOMs, and supply chain integrity

For detailed FIPS guidance, see [FIPS 140-3 and cryptography in Azure Linux](./fips-cryptography.md). For audit configuration, see [Logging and auditing in Azure Linux](./logging-auditing.md).

<!-- ## Supply chain security

Microsoft is investing in supply chain security across Azure Linux 4.0.
The following are goals being delivered through the AL4 lifecycle:

- **GPG-signed packages** and repository metadata.
- **SBOMs** (Software Bill of Materials) in SPDX and CycloneDX formats
  for images and packages.
- **SLSA Level 3** target for the build pipeline, including build
  provenance, hermetic builds, and non-falsifiable attestations.
- **VEX advisories** published by MSRC for vulnerability disclosure.

Capability availability evolves through preview; see [Managing CVEs](16.%20managing-cves.md) for the current state and
consumption guidance.
[Supply chain security](14.%20supply-chain-security.md) and

For details, see [Supply chain security](14.%20supply-chain-security.md). -->

## Build security

The Azure Linux build system and tooling are regularly audited to maintain a secure supply chain. Build artifacts include cryptographically signed provenance records that enable verification of the build pipeline integrity.

## Related content

- Review the full security baseline in [Security and compliance overview](./security-overview.md)
- Manage CVEs and security updates in [Managing CVEs](./manage-cves.md)
