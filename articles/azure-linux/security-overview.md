---
title: Overview of Azure Linux Security
description: Learn about the security features, protections, and compliance capabilities built into Azure Linux to help secure workloads running on Azure.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/26/2026
---

# Azure Linux security overview

Azure Linux is built with a secure-by-default baseline and layered protections across boot, runtime, and update workflows. This article covers the security model, key features, and recommendations for validating security settings on Azure Linux.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Security model

The following table summarizes the core security principles that guide the design of Azure Linux and how they are implemented across the platform:

| Security principle | Implementation in Azure Linux |
| ------------------ | ----------------------------- |
| Secure by default | Minimal package set, key-only SSH authentication, firewalld enabled |
| Defense in depth | SELinux mandatory access control, network hardening |
| Least privilege | SELinux policies, kernel capability restrictions |

## Core security features

### Secure by default baseline

- **Hardened kernel**: With ASLR, stack canaries, and pointer restrictions.
- **Minimal package set**: Only essential packages for core operating system (OS) functionality. No legacy protocols, development tools, or non-essential daemons in the base image.
- **No external network connections by default**: firewalld denies all inbound traffic except SSH.
- **Key-only SSH authentication**: Password authentication is disabled.
- **Package integrity**: All packages and repository metadata are GPG-signed.

### Runtime protections

- **SELinux:** In Enforcing mode. Targeted policy with pre-labeled packages.
- **eBPF hardening**: Unprivileged BPF disabled, interpreter disabled in favor of JIT.

### Network security

- **firewalld**: Enabled by default with nftables backend.
- **Network sysctl hardening**: Strict reverse path filtering, ICMP redirects disabled, SYN cookies enabled.
- **IPv6**: Enabled with hardening settings applied.

### Identity and access

- **SSSD/realmd**: Available from the Azure Linux repositories for hybrid identity scenarios with Active Directory and LDAP.

### Cryptography

- **FIPS 140-3**: Validated cryptographic modules with a simple enablement mechanism.
- **Post-quantum cryptography**: ML-KEM hybrid key exchange support.
- **System crypto libraries**: Applications can use OS-level crypto.

### Logging, auditing, and monitoring

- **auditd**: enabled by default; rule profiles ship under /usr/share/audit-rules/ and can be activated by copying the profile you need into /etc/audit/rules.d/.
- **journald**: Persistent storage with structured JSON output.

<!-- ### Supply chain security

Microsoft is investing in supply chain security across Azure Linux 4.0. For more information see [Supply chain security](14.%20supply-chain-security.md) and [Certifications](15.%20certifications.md). -->

### Vulnerability scanning integrations

Azure Linux supports integration with common vulnerability scanning and security tooling. For a list of supported tools, see [Azure Linux partner solutions](./ecosystem-support.md).

## Related content

- [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md)
- [Azure Linux Feature Roadmap](https://github.com/orgs/microsoft/projects/970/views/2)
