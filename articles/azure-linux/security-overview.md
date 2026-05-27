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
| **Secure by default** | Minimal package set, key-only SSH authentication, firewalld enabled, hardened kernel settings |
| **Defense in depth** | UEFI Secure Boot, TPM 2.0 measured boot, SELinux mandatory access control, network hardening |
| **Least privilege** | SELinux policies, systemd service sandboxing, kernel capability restrictions |
| **Transparency** | Documentation of security capabilities and verification steps for each feature |

:::image type="content" source="./media/security.png" alt-text="Screenshot of a diagram of the Azure Linux security model components." lightbox="./media/security.png":::

## Azure Linux security topics

The following table covers the key security features and protections in Azure Linux:

| Topic | Description |
| ----- | ----------- |
| Secure Boot and trusted boot | UEFI Secure Boot, TPM 2.0, measured boot, and kernel lockdown |
| Kernel hardening | ASLR, stack canaries, sysctl hardening, and systemd sandboxing |
| SELinux and Landlock | Mandatory access control with SELinux and Landlock LSM |
| Network security | firewalld, nftables, and network sysctl hardening defaults |
| Identity and access | Entra ID SSH, key-based authentication, and hybrid identity |
| Disk and data protection | Full disk encryption, fs-verity, and dm-verity |
| Logging and auditing | auditd, journald, and Azure Monitor integration |
| FIPS 140-3 and cryptography | FIPS mode, crypto-policies, and post-quantum cryptography |
| System crypto libraries | Configure applications to use OS-level cryptographic libraries |
| eBPF security | eBPF security model, restrictions, and tooling |
| Kernel livepatching | Zero-reboot patching for critical kernel CVEs |
| Certifications | FIPS, CIS, STIG, and FedRAMP compliance artifacts |
| [Managing CVEs](./manage-cves.md) | CVE tracking, security updates, and vulnerability scanning |

<!-- | [Supply chain security](14.%20supply-chain-security.md) | Package signing, SBOMs, and SLSA compliance | -->

## Core security features

### Secure by default baseline

- **Hardened kernel**: With ASLR, stack canaries, and pointer restrictions.
- **Minimal package set**: Only essential packages for core operating system (OS) functionality. No legacy protocols, development tools, or non-essential daemons in the base image.
- **No external network connections by default**: firewalld denies all inbound traffic except SSH.
- **Key-only SSH authentication**: Password authentication is disabled.
- **Package integrity**: All packages and repository metadata are GPG-signed.
- **systemd service sandboxing**: Default services use aggressive sandboxing directives.

### Boot and platform integrity

- **Secure Boot**: Only signed kernels, SHIM, GRUB, and systemd-boot components are allowed.
- **Measured boot**: Full boot chain measured into TPM 2.0 PCRs for attestation workflows.
- **Kernel lockdown**: Integrity mode when Secure Boot is enabled.
- **dm-verity**: Read-only verification of critical filesystem content.
- **Confidential VM (CVM) support**: Encrypted image boot and resealing workflows.

### Runtime protections

- **SELinux**: Targeted policy with pre-labeled packages.
- **Landlock**: Application-level sandboxing compiled into the kernel.
- **eBPF hardening**: Unprivileged BPF disabled, interpreter disabled in favor of JIT.
- **systemd sandboxing**: PrivateTmp, NoNewPrivileges, capability restrictions.

### Network security

- **firewalld**: Enabled by default with nftables backend.
- **Network sysctl hardening**: Strict reverse path filtering, ICMP redirects disabled, SYN cookies enabled.
- **IPv6**: Enabled with hardening settings applied.

### Identity and access

- **Entra ID SSH**: Passwordless SSH via Microsoft Entra ID using the `AADSSHLoginForLinux` Azure VM extension.
- **SSSD/realmd**: Available from the Azure Linux repositories for hybrid identity scenarios with Active Directory and LDAP.

### Cryptography

- **FIPS 140-3**: Validated cryptographic modules with a simple enablement mechanism.
- **Crypto-policies**: Unified framework for cryptographic configuration across all applications.
- **Post-quantum cryptography**: ML-KEM hybrid key exchange support.
- **System crypto libraries**: Applications can use OS-level crypto.

### Logging, auditing, and monitoring

- **auditd**: enabled by default; rule profiles ship under `/usr/share/audit-rules/` and can be activated by copying the profile you need into `/etc/audit/rules.d/`.
- **journald**: Persistent storage with structured JSON output.
- **Azure Monitor Agent**: Native integration for centralized log collection.

<!-- ### Supply chain security

Microsoft is investing in supply chain security across Azure Linux 4.0. For more information see [Supply chain security](14.%20supply-chain-security.md) and [Certifications](15.%20certifications.md). -->

### Vulnerability scanning integrations

Azure Linux supports integration with common vulnerability scanning and security tooling, including:

- Qualys
- Trivy / Aqua Security
- Tenable
- Anchore
- Grype
- Prisma Cloud
- Palo Alto Networks tooling

Availability and configuration vary by environment. Refer to your scanner documentation for details.

## Validate your security baseline

1. **Check boot integrity**: Verify Secure Boot and TPM status.

    ```bash
    mokutil --sb-state
    cat /sys/kernel/security/lockdown
    ```

1. **Confirm runtime protections**: Check SELinux mode, kernel hardening settings, and firewall status.

    ```bash
    getenforce
    sysctl kernel.randomize_va_space kernel.dmesg_restrict kernel.kptr_restrict
    sudo firewall-cmd --state
    ```

1. **Validate compliance**: Verify FIPS mode and audit configuration where applicable.

    ```bash
    cat /proc/sys/crypto/fips_enabled
    sudo auditctl -l | head -5
    ```

## Get started with Azure Linux security

Use the following paths based on your requirements:

- [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md)
- **Need to validate your security baseline?** Follow the steps in [Validate your security baseline](#validate-your-security-baseline).
