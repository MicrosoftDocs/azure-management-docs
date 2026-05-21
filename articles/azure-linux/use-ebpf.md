---
title: Use eBPF on Azure Linux
description: Learn how to use extended Berkeley Packet Filter (eBPF) on Azure Linux for observability, networking, and security tooling while adhering to platform security restrictions.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Use extended Berkeley Packet Filter (eBPF) on Azure Linux

Azure Linux includes extended Berkeley Packet Filter (eBPF) support for observability, networking, and security tooling. The platform applies security restrictions to reduce risk from untrusted eBPF programs.

This article explains how to use eBPF on Azure Linux, including how to verify that eBPF is available, install the necessary tooling, and develop eBPF programs while respecting the platform's security restrictions.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Security model

Azure Linux enforces a security model for eBPF that limits the potential for untrusted programs to compromise the kernel or other workloads. The key restrictions include:

- Unprivileged users can't load eBPF programs.
- The eBPF interpreter is disabled; only JIT compilation is used. This mitigates interpreter-based exploitation techniques.

## Supported kernel versions

eBPF capabilities depend on the kernel version. Use the [upstream kernel matrix](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md) to understand feature availability.

## Verify eBPF tooling

1. Install `bpftool`.
1. Run `bpftool prog list` to confirm eBPF is available.

    ```bash
    sudo dnf install -y bpftool
    sudo bpftool prog list
    ```

## Install eBPF tooling

Install the kernel build headers and `bpftool`:

```bash
sudo dnf install -y kernel-headers kernel-devel bpftool
```

## Develop eBPF applications

Install the `libbpf` development package:

```bash
sudo dnf install -y libbpf-devel
```

## Related content

- [Configure mandatory access control (MAC) in Azure Linux with SELinux and Landlock](./mandatory-access-control.md)
- [Kernel hardening in Azure Linux](./kernel-hardening.md)
- [Security and compliance overview](./security-overview.md)
