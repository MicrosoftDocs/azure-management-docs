---
title: Azure Linux Architecture Overview
description: Learn about the layered architecture of Azure Linux, including the kernel, core OS, user-space packages, and workload layers, and how these layers are optimized for running workloads on Azure.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux architecture overview

Azure Linux is a Microsoft-maintained Linux distribution built on the Fedora ecosystem and optimized for Azure. This article explains how Azure Linux is structured, what it includes by default, and where it intentionally draws boundaries.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## How Azure Linux relates to Fedora

Azure Linux derives from [Fedora](https://fedoraproject.org/) and uses the same RPM-based packaging ecosystem. You get the familiarity of `dnf5`, Fedora build tooling (`mock`, `fedpkg`, `koji`, `kiwi`), and modern compiler toolchains — all tracked from upstream Fedora. On top of that foundation, Microsoft adds Azure-specific hardening, a custom kernel, and a managed lifecycle suited for cloud workloads.

The kernel ships as version 6.18 LTS, extended with Hyper-V guest drivers, Azure-specific performance tuning, and security hardening validated across Azure VM SKUs.

## Layered architecture

Azure Linux uses a layered model. Each layer builds on the one below it, from hardware and firmware at the base up to your workloads at the top.

:::image type="content" source="./media/layered-architecture-azure-linux.png" alt-text="Screenshot of a diagram showing the Azure Linux layered architecture." lightbox="./media/layered-architecture-azure-linux.png":::

### Kernel layer

The custom Azure Linux kernel includes Hyper-V guest drivers, security hardening patches, and performance optimizations tuned for Azure infrastructure. You can choose between Long-Term Support (LTS) kernels for stability or Hardware Enablement (HWE) kernels for new hardware and GPU support. See [Release cadence and lifecycle](./release-cadence-lifecycle.md) for details.

### Core OS layer

This layer provides the minimal root filesystem: `systemd` for service management, `glibc` as the C runtime, and OpenSSL for cryptography. These are Tier 1 components, meaning they're version-locked for the lifetime of a major release and receive only security backports. This keeps application binary interface (ABI) stable and behavior predictable.

### User-space packages layer

Language runtimes (Python, Go, Rust, Node.js), container tooling, and application dependencies live here. Tier 1 packages are locked for stability. Tier 2 packages are updated twice a year through H1/H2 update windows. See [Release cadence and lifecycle](./release-cadence-lifecycle.md) for the full tier model.

### Workload layer

Your Azure services and applications run here. Azure Linux presents the same OS foundation, package set, and behavior whether you're running on AKS, Azure VMs, or container images.

## Platform scope

Azure Linux is designed for Azure cloud workloads. Although Azure Linux is open source, Microsoft support and lifecycle commitments apply only to Azure scenarios.

The following table outlines what is and isn't supported on Azure Linux:

| Area | Supported | Not supported |
| ---- | --------- | ------------- |
| **Architectures** | x86-64 (v2 minimum), ARMv8 (64-bit) | 32-bit architectures |
| **Environments** | Azure VMs, AKS, container images | ISO images, on-premises, multi-cloud, IoT, edge devices |
| **User interface** | Text-based console, SSH | Graphical desktop environments, GUI installer |
| **Virtualization** | KVM guests, Hyper-V | Xen |
| **Peripheral hardware** | Azure-attached storage, networking, GPUs | Bluetooth, Wi-Fi, printers, audio/video, robotics |
| **Trusted platform** | TPM 2.0 | TPM 1.x |
| **Language support** | Global locale support available | Not all language packs are included in the base image |

Every Azure Linux image includes `waagent` and `cloud-init`. These components are required for Azure integration and must always be present.

## Repository structure

Azure Linux ships packages in several separate repositories. Knowing which repository a package comes from helps you reason about supportability, what's available out of the box, and what you need to opt into. For day-to-day package management information, see [Package management](./package-management-overview.md).

## Networking defaults

Azure Linux ships with a networking stack tuned for Azure VMs, AKS nodes, and container workloads. Most workloads can use the defaults without changes.

The following table summarizes the default networking components in Azure Linux and the alternatives that are available if you need to override the defaults for a specific scenario:

| Component | Default | Alternative | Notes |
| --------- | ------- | ----------- | ----- |
| **Network manager** | systemd‑networkd + cloud‑init | NetworkManager (available, not default) | systemd‑networkd is the default for Azure VM and container scenarios. |
| **Firewall** | firewalld | N/A | Enabled by default with a deny‑inbound, allow‑outbound policy. |
| **Firewall backend** | nftables | iptables (legacy, available) | nftables is the modern replacement for iptables. iptables legacy support is available but not default. |
| **IPv6** | Enabled and hardened | N/A | Strict sysctl hardening applied for both IPv4 and IPv6. |

## Storage defaults

Azure Linux storage defaults are tuned for Azure-attached disks and the Hyper-V hypervisor underneath every Azure VM. The filesystem, boot loader, and clock source are chosen for predictable performance and compatibility with Azure platform features such as snapshots, managed disks, and Trusted Launch.

The following table summarizes the default storage settings in Azure Linux and the alternatives that are available if you need to override the defaults for a specific scenario:

| Setting | Default | Alternatives |
| ------- | ------- | ----------- |
| **Filesystem** | ext4 | xfs, btrfs |
| **Boot** | GRUB2 bootloader | N/A |
| **Clock** | Hyper‑V PTP clock source | N/A |
| **NVMe** | Timeout tuned for Azure attached storage | N/A |

## Security architecture

Azure Linux is hardened at every layer, from the kernel up through the supply chain. The following sections describe the security controls that are enabled by default. See [Security and compliance](./security-overview.md) for more information.

### Mandatory access control

Mandatory access control (MAC) confines processes to the access they actually need, even when they run as root. Azure Linux uses SELinux as its sole MAC framework.

The following table summarizes the MAC settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **MAC framework** | Only SELinux is supported. AppArmor is not supported. |
| **Preview mode** | Permissive |
| **GA mode** | Enforcing |
| **Landlock** | Compiled in. Enforcing only for Azure Container Linux. |

### Secure Boot and trusted boot

Secure Boot ensures only signed bootloaders and kernels run, and trusted boot extends that chain of trust to the TPM so you can attest to a system's boot state.

The following table summarizes the Secure Boot and trusted boot settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **Secure Boot** | Mandatory for all images. |
| **Signed components** | shim, GRUB, kernel, systemd‑boot. |
| **Kernel lockdown** | Tied to Secure Boot state. |
| **Measured boot** | Supported via TPM 2.0. |

### Kernel and system hardening

The kernel and userspace are built with mitigations that make exploits harder to write and easier to contain if they succeed.

The following table summarizes the kernel and system hardening capabilities in Azure Linux:

| Capability | Description |
| ---------- | ----------- |
| **ASLR** | Strong address space layout randomization enabled. |
| **Stack protection** | Compiler‑level stack protections applied to all packages. |
| **Syscall restrictions** | Default seccomp profiles and syscall filtering. |
| **Service sandboxing** | Aggressive systemd service sandboxing with restricted capabilities. |
| **Minimal base image** | Reduced attack surface through minimal package set. |

### Cryptography

Azure Linux centralizes cryptographic policy so that algorithms and key sizes stay consistent across OpenSSL, GnuTLS, NSS, and OpenSSH.

The following table summarizes the cryptography settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **FIPS** | FIPS 140‑3 certification is mandatory. |
| **Crypto policies** | Fedora crypto‑policies adopted for consistent algorithm selection. |
| **Post‑quantum** | ML‑KEM planned, aligned with Fedora and RHEL roadmap. |

### Identity and access

Azure Linux integrates with Microsoft Entra ID and traditional directory services so you can manage logins centrally instead of per-host.

The following table summarizes the identity and access settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **Entra ID SSH** | Works out of the box on Azure VMs. |
| **Default SSH auth** | Key‑based only. Password authentication disabled by default. |
| **Hybrid identity** | SSSD and realmd included for Active Directory and Entra ID domain join. |

### Logging and auditing

Azure Linux ships with audit and journal logging configured to a CIS baseline so you have a forensic record without extra setup.

The following table summarizes the logging and auditing settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **Audit daemon** | auditd enabled (CIS Level 2 baseline). |
| **Journal storage** | Persistent journald storage. |
| **Monitoring** | Azure Monitor agent integration supported. |

### Supply chain security

Every artifact you receive — packages, repositories, and images — is signed and accompanied by provenance data so you can verify what you're running.

The following table summarizes the supply chain security capabilities in Azure Linux:

| Capability | Description |
| ---------- | ----------- |
| **Package signing** | All packages and repositories are cryptographically signed. |
| **SBOMs** | Software Bills of Materials published and signed for every release. |
| **Build provenance** | Target SLSA Level 3 for build integrity and auditability. |

## Related content

- To learn more about Azure Linux, see the [Azure Linux Container Host overview](./azure-linux-overview.md).
- To plan your update strategy, review the [Release cadence and lifecycle](./release-cadence-lifecycle.md).
