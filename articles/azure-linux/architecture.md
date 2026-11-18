---
title: Azure Linux Architecture Overview
description: Learn about the layered architecture of Azure Linux, including the kernel, core OS, user-space packages, and workload layers, and how these layers are optimized for running workloads on Azure.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 10/30/2026
ai-usage: ai-assisted
---

# Azure Linux architecture overview

Azure Linux is a Microsoft-maintained Linux distribution built on the Fedora ecosystem and optimized for Azure. This article explains how Azure Linux is structured, what it includes by default, and where it intentionally draws boundaries.

## How Azure Linux relates to Fedora

Azure Linux derives from [Fedora](https://fedoraproject.org/) and uses the same RPM-based packaging ecosystem. You get the familiarity of `dnf5`, Fedora build tooling (`mock`, `fedpkg`, `koji`, `kiwi`), and modern compiler toolchains — all tracked from upstream Fedora. On top of that foundation, Microsoft adds Azure-specific hardening, a custom kernel, and a managed lifecycle suited for cloud workloads.

The kernel ships as version 6.18 LTS, extended with Hyper-V guest drivers, Azure-specific performance tuning, and security hardening validated across Azure VM SKUs.

## Layered architecture

Azure Linux uses a layered model. Each layer builds on the one below it, from hardware and firmware at the base up to your workloads at the top.

:::image type="content" source="./media/layered-architecture-azure-linux.png" alt-text="Screenshot of a diagram showing the Azure Linux layered architecture." lightbox="./media/layered-architecture-azure-linux.png":::

### Kernel layer

The custom Azure Linux kernel includes Hyper-V guest drivers, security hardening patches, and performance optimizations tuned for Azure infrastructure. See [Manage the Azure Linux kernel](./kernel-management.md) for kernel management guidance.

### Core OS layer

This layer provides the minimal root filesystem: `systemd` for service management, `glibc` as the C runtime, and OpenSSL for cryptography. These are Tier 1 components, meaning they're version-locked for the lifetime of a major release and receive only security backports. This keeps application binary interface (ABI) stable and behavior predictable.

### User-space packages layer

Language runtimes (Python, Go, Rust, Node.js), container tooling, and application dependencies live here. Tier 1 packages are locked for stability. Tier 2 packages are updated on a predictable cadence. For more information, see [Release cadence and lifecycle](./release-cadence-lifecycle.md).

### Workload layer

Your Azure services and applications run here. Azure Linux presents the same OS foundation, package set, and behavior whether you're running on AKS, Azure VMs, or container images.

## Platform scope

Azure Linux is designed for Azure cloud workloads. Although Azure Linux is open source, Microsoft support and lifecycle commitments apply only to Azure scenarios.

The following table outlines what is and isn't supported on Azure Linux:

| Area | Supported | Not supported |
| ---- | --------- | ------------- |
| **Architectures** | x64 (`x86_64`) and ARM64 (`aarch64`) | 32-bit architectures |
| **Environments** | Azure VMs, AKS, container images | ISO images, on-premises, multicloud, IoT, edge devices |
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
| **Firewall** | None enabled | firewalld (available, not installed) | No host firewall is enabled by default. nftables and iptables-legacy are installed with no rules. Use Azure network security groups (NSGs) to control inbound traffic. |
| **Firewall backend** | nftables | iptables (legacy, available) | nftables is the modern replacement for iptables. iptables legacy support is available but not default. |
| **IPv6** | Enabled and hardened | N/A | Strict sysctl hardening applied for both IPv4 and IPv6. |

## Storage defaults

Azure Linux storage defaults are tuned for Azure-attached disks and the Hyper-V hypervisor underneath every Azure VM. The filesystem, boot loader, and clock source are chosen for predictable performance and compatibility with Azure platform features such as snapshots and managed disks.

The following table summarizes the default storage settings in Azure Linux and the alternatives that are available if you need to override the defaults for a specific scenario:

| Setting | Default | Alternatives |
| ------- | ------- | ----------- |
| **Filesystem** | ext4 | xfs, btrfs |
| **Boot** | GRUB2 bootloader | N/A |
| **Clock** | Hyper‑V PTP clock source | N/A |
| **NVMe** | Timeout tuned for Azure attached storage | N/A |

## Security architecture

Azure Linux includes security controls at several layers. The following sections summarize them. For defaults and how to check them, see [Security and compliance](./security-overview.md).

### Mandatory access control

Mandatory access control (MAC) confines processes to the access they actually need, even when they run as root.

Azure Linux 4 enables SELinux in enforcing mode as its MAC framework.

### Kernel and system hardening

The kernel and userspace are built with mitigations that make exploits harder to write and easier to contain if they succeed.

The following table summarizes the kernel and system hardening capabilities in Azure Linux:

| Capability | Description |
| ---------- | ----------- |
| **ASLR** | Strong address space layout randomization enabled. |
| **Build hardening** | Hardened RPM build flags (stack protector, `_FORTIFY_SOURCE=3`, PIE, full RELRO) are on by default; individual packages can opt out. See [Build hardening](./build-hardening.md). |
| **Syscall restrictions** | Kernel seccomp filtering is enabled; many system services run with systemd `SystemCallFilter` restrictions. |
| **Minimal base image** | Reduced attack surface through minimal package set. |

### Cryptography

Azure Linux centralizes cryptographic policy so that algorithms and key sizes stay consistent across OpenSSL, GnuTLS, NSS, and OpenSSH.

The following table summarizes the cryptography settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **FIPS** | FIPS 140-3: In progress. Certification pending; use a certified release for FIPS-required workloads. |
| **Crypto policies** | Fedora crypto‑policies adopted for consistent algorithm selection. |

### Logging and auditing

The following table summarizes the logging and auditing settings in Azure Linux:

| Setting | Value |
| ------- | ----- |
| **Audit daemon** | auditd runs. The default rules suppress syscall auditing (`-a never,task`). See [Logging and auditing](./logging-and-auditing.md). |
| **Journal storage** | Persistent journald storage. |

## Related content

- To learn more about Azure Linux, see the [Azure Linux overview](./azure-linux-overview.md).
- To plan for native application compatibility, see [CPU architectures and toolchain compatibility](./cpu-architectures-toolchains.md).
- To plan your update strategy, review the [Release cadence and lifecycle](./release-cadence-lifecycle.md).
