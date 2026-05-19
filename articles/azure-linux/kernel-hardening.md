---
title: Kernel Hardening in Azure Linux
description: Learn about kernel-level security hardening features in Azure Linux, including memory protection, pointer restrictions, kernel lockdown, and service sandboxing.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Kernel hardening in Azure Linux

Azure Linux applies defense-in-depth hardening at the kernel level to reduce the attack surface and limit the impact of potential exploits. These protections are enabled by default and require no extra configuration.

This article covers the key kernel hardening features active in Azure Linux and how to verify they're in place.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Verify kernel hardening

You can use the following to confirm that core hardening settings are active on your Azure Linux instance:

```bash
# Memory protection — full address space layout randomization
sysctl kernel.randomize_va_space

# Kernel pointer restriction
sysctl kernel.kptr_restrict

# dmesg access restriction
sysctl kernel.dmesg_restrict

# Check kernel config for compiled-in protections
grep -E "STACKPROTECTOR_STRONG|INIT_ON_ALLOC_DEFAULT_ON|LOCKDOWN" /boot/config-$(uname -r)
```

Example output for a hardened system:

```output
kernel.randomize_va_space = 2
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
```

## Memory protection

Azure Linux enables multiple layers of memory protection to make exploitation significantly harder: [**address space layout randomization (ASLR)**](#address-space-layout-randomization-aslr), [**stack canaries**](#stack-canaries), [**Supervisor Mode Access Prevention (SMAP)** and **Supervisor Mode Execution Prevention (SMEP)**](#smap-and-smep), and [**zero-fill on allocation**](#zero-fill-on-allocation).

### Address space layout randomization (ASLR)

ASLR randomizes the memory layout of processes, making it difficult for attackers to predict target addresses.

- **Setting**: `kernel.randomize_va_space=2` (full randomization).
- Full randomization covers the stack, VDSO page, shared memory regions, and data segments.

#### Verify ASLR

Verify ASLR is active:

```bash
sysctl kernel.randomize_va_space
```

### Stack canaries

Stack canaries detect buffer overflow attacks by placing a known value on the stack before the return pointer.

- **Kernel config**: `CONFIG_STACKPROTECTOR_STRONG` is enabled.
- This option instruments all functions that use stack buffers, not just those with obvious vulnerabilities.

#### Verify stack canaries

Verify that stack canaries are enabled by checking the kernel configuration:

```bash
grep CONFIG_STACKPROTECTOR_STRONG /boot/config-$(uname -r)
```

### SMAP and SMEP

Supervisor Mode Access Prevention (SMAP) and Supervisor Mode Execution Prevention (SMEP) prevent the kernel from accessing or executing user-space memory.

- Both are hardware-dependent and enabled automatically where the processor supports them.
- These protections block a common class of kernel exploits that trick the kernel into running attacker-controlled code in user space.

### Zero-fill on allocation

- **Kernel config**: `CONFIG_INIT_ON_ALLOC_DEFAULT_ON` is enabled.
- Allocated memory pages are zero-filled, preventing information leaks from previously freed memory.

## Kernel pointer and information disclosure restrictions

Azure Linux restricts access to sensitive kernel information, including [kernel pointers](#kernel-pointer-restriction) and [dmesg output](#dmesg-restriction), to limit what an attacker can learn about the running system.

### Kernel pointer restriction

Kernel pointer addresses can reveal the memory layout of the kernel, which aids exploitation.

- **Setting**: `kernel.kptr_restrict=2`.
- Kernel pointers are hidden from all users, including privileged users reading `/proc/kallsyms`.

#### Verify kernel pointer restriction

Verify that kernel pointer restriction is active by checking the value of `kernel.kptr_restrict`:

```bash
sysctl kernel.kptr_restrict
```

### dmesg restriction

The kernel message buffer can contain sensitive information about hardware, drivers, and kernel internals.

- **Setting**: `kernel.dmesg_restrict=1`.
- Only privileged users (with `CAP_SYSLOG`) can read kernel log messages via `dmesg`.

#### Verify dmesg restriction

Verify that dmesg restriction is active by checking the value of `kernel.dmesg_restrict`:

```bash
sysctl kernel.dmesg_restrict
```

## Kernel lockdown

When Secure Boot is active, Azure Linux enables kernel lockdown in **integrity mode**. Lockdown prevents modifications to the running kernel, even by the root user.

In integrity mode:

- Unsigned kernel modules can't be loaded.
- Direct access to `/dev/mem` and `/dev/kmem` is blocked.
- Writing to MSRs (model-specific registers) from user space is prohibited.
- Hibernation is restricted to prevent tampering with kernel memory images.

For a complete overview of the boot chain protections, including Secure Boot, UEFI validation, and the trusted boot process, see [Get started with Secure Boot and Trusted Boot in Azure Linux](./secure-boot-trusted-boot.md).

## systemd service sandboxing

All default systemd services in Azure Linux use aggressive sandboxing to limit the impact of a compromised service. Each service is individually assessed for maximum sandboxing while maintaining its required functionality.

### Baseline restrictions

The following systemd service sandboxing restrictions are applied where operationally feasible:

| Directive | Effect |
| --------- | ------ |
| `PrivateTmp=yes` | Isolates the service's `/tmp` and `/var/tmp` directories |
| `PrivateDevices=yes` | Prevents access to physical devices |
| `PrivateNetwork=yes` | Isolates the service from the network namespace |
| `ProtectSystem=strict` | Mounts the file system as read-only, except explicitly allowed paths |
| `ProtectHome=yes` | Makes `/home`, `/root`, and `/run/user` inaccessible |
| `ProtectKernelTunables=yes` | Makes `/proc/sys`, `/sys`, and related paths read-only |
| `ProtectControlGroups=yes` | Makes the cgroup hierarchy read-only |
| `NoNewPrivileges=yes` | Prevents the service from gaining new privileges through `setuid` or `setgid` binaries |
| `CapabilityBoundingSet=` | Drops all capabilities not explicitly required by the service |
| `SystemCallFilter=` | Restricts the set of system calls the service can make |

### Evaluate service sandboxing

Use `systemd-analyze security` to review the sandboxing posture of any service:

```bash
# List the security exposure score for all services
systemd-analyze security

# Review detailed sandboxing for a specific service
systemd-analyze security <service-name>
```

Lower scores indicate stronger sandboxing. A score of 0.0 represents a fully locked-down service.

## Minimal attack surface

Azure Linux follows a _secure by default, functional by design_ approach to package selection. The base image includes only the packages essential for core operating system functionality.

### Included in Azure Linux base image

| Category | Packages |
| -------- | -------- |
| Boot and system init | systemd, util-linux, coreutils |
| Networking | NetworkManager or systemd-networkd, openssh-server |
| Package management | dnf |
| Cloud provisioning | cloud-init, Azure Linux Agent |
| Security tooling | SELinux utilities, auditd |

### Excluded from Azure Linux base image

The following categories are intentionally excluded to reduce attack surface:

| Category | Packages |
| -------- | -------- |
| Legacy protocols | telnet, rsh, rlogin, and other unencrypted remote access tools |
| Development tools | gcc, make, gdb, and compiler toolchains |
| Non-essential daemons | Services not required for base OS functionality |

### Install additional packages

When you need additional packages, install them on demand from the Azure Linux repositories:

```bash
sudo dnf install <package-name>
```

## Related content

- [Get started with Secure Boot and Trusted Boot in Azure Linux](./secure-boot-trusted-boot.md)
- [Configure mandatory access control (MAC) in Azure Linux with SELinux and Landlock](./mandatory-access-control.md)
- [Azure Linux network security overview](./network-security-overview.md)
- [Logging and auditing in Azure Linux](./logging-auditing.md)
