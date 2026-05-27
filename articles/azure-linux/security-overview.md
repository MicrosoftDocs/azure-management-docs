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

The following table provides links to detailed articles covering each key Azure Linux security topic so you can explore the capabilities and configuration guidance for each area:

| Topic | Description |
| ----- | ----------- |
| [Secure Boot and trusted boot](./secure-boot-trusted-boot.md) | UEFI Secure Boot, TPM 2.0, measured boot, and kernel lockdown |
| [Kernel hardening](./kernel-hardening.md) | ASLR, stack canaries, sysctl hardening, and systemd sandboxing |
| [SELinux and Landlock](./mandatory-access-control.md) | Mandatory access control with SELinux and Landlock LSM |
| [Network security](./network-security-overview.md) | firewalld, nftables, and network sysctl hardening defaults |
| [Identity and access](./identity-access-management.md) | Entra ID SSH, key-based authentication, and hybrid identity |
| [Disk and data protection](./disk-data-protection.md) | Full disk encryption, fs-verity, and dm-verity |
| [Logging and auditing](./logging-auditing.md) | auditd, journald, and Azure Monitor integration |
| [FIPS 140-3 and cryptography](./fips-cryptography.md) | FIPS mode, crypto-policies, and post-quantum cryptography |
| [System crypto libraries](./system-crypto-libraries.md) | Configure applications to use OS-level cryptographic libraries |
| [eBPF security](./use-ebpf.md) | eBPF security model, restrictions, and tooling |
| [Kernel livepatching](./kernel-live-patching.md) | Zero-reboot patching for critical kernel CVEs |
| [Certifications](./certifications-compliance.md) | FIPS, CIS, STIG, and FedRAMP compliance artifacts |
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

- **Secure Boot**: Only signed kernels, SHIM, GRUB, and systemd-boot components are allowed. For more information, see [Secure Boot and trusted boot](./secure-boot-trusted-boot.md).
- **Measured boot**: Full boot chain measured into TPM 2.0 PCRs for attestation workflows.
- **Kernel lockdown**: Integrity mode when Secure Boot is enabled.
- **dm-verity**: Read-only verification of critical filesystem content.
- **Confidential VM (CVM) support**: Encrypted image boot and resealing workflows.

### Runtime protections

- **SELinux**: Targeted policy with pre-labeled packages. For more information, see [SELinux and Landlock](./mandatory-access-control.md).
- **Landlock**: Application-level sandboxing compiled into the kernel.
- **eBPF hardening**: Unprivileged BPF disabled, interpreter disabled in favor of JIT. For more information, see [eBPF security](./use-ebpf.md).
- **systemd sandboxing**: PrivateTmp, NoNewPrivileges, capability restrictions. For more information, see [Kernel hardening](./kernel-hardening.md).

### Network security

- **firewalld**: Enabled by default with nftables backend. For more information, see [Network security](./network-security-overview.md).
- **Network sysctl hardening**: Strict reverse path filtering, ICMP redirects disabled, SYN cookies enabled.
- **IPv6**: Enabled with hardening settings applied.

### Identity and access

- **Entra ID SSH**: Passwordless SSH via Microsoft Entra ID using the `AADSSHLoginForLinux` Azure VM extension. For more information, see [Identity and access](./identity-access-management.md).
- **SSSD/realmd**: Available from the Azure Linux repositories for hybrid identity scenarios with Active Directory and LDAP.

### Cryptography

- **FIPS 140-3**: Validated cryptographic modules with a simple enablement mechanism. For more information, see [FIPS 140-3 and cryptography](./fips-cryptography.md).
- **Crypto-policies**: Unified framework for cryptographic configuration across all applications.
- **Post-quantum cryptography**: ML-KEM hybrid key exchange support.
- **System crypto libraries**: Applications can use OS-level crypto. For more information, see [System crypto libraries](./system-crypto-libraries.md).

### Logging, auditing, and monitoring

- **auditd**: enabled by default; rule profiles ship under `/usr/share/audit-rules/` and can be activated by copying the profile you need into `/etc/audit/rules.d/`. For more information, see [Logging and auditing](./logging-auditing.md).
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

- **Need compliance certification?** Start with [FIPS 140-3 and cryptography](./fips-cryptography.md) and [Certifications and compliance](./certifications-compliance.md).
- **Hardening a production workload?** Review [kernel hardening in Azure Linux](./kernel-hardening.md), [network security](./network-security-overview.md), and [logging and auditing](./logging-auditing.md).
- **Running containers?** See [SELinux and Landlock](./mandatory-access-control.md) and [disk and data protection](./disk-data-protection.md) for container-specific security guidance.
- **Need to validate your security baseline?** Follow the steps in [Validate your security baseline](#validate-your-security-baseline).
