---
title: Inspect kernel hardening in Azure Linux
description: Inspect and interpret kernel memory protections, information disclosure limits, processor mitigations, and service sandboxing on Azure Linux.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Inspect kernel hardening in Azure Linux

Azure Linux 4 uses kernel build options, runtime parameters, and processor
features to reduce exploit opportunities.

## Default kernel hardening settings

The image enables process address space layout randomization and restricts
access to kernel logs and pointer output. The kernel includes strong stack
protection and zero-initialization of memory allocations by default.
Processor-dependent protections vary with VM size and hardware. Kernel lockdown
support is built in but isn't active by default.

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.

## Verify kernel parameters, lockdown, and processor mitigations

Check runtime settings, the running kernel's build configuration, and the
features exposed by the VM's processor separately.

### Identify the running system

Record the operating system and running kernel:

```bash
cat /etc/os-release
uname -r
```

From Azure CLI, you can also record the image used to create the VM:

```azurecli
az vm show \
  --resource-group <resource-group-name> \
  --name <vm-name> \
  --query "storageProfile.imageReference"
```

### Inspect runtime kernel parameters

Read process address-space randomization and kernel information-disclosure
restrictions:

```bash
sysctl \
  kernel.randomize_va_space \
  kernel.dmesg_restrict \
  kernel.kptr_restrict
```

Expected result for the default configuration:

```output
kernel.randomize_va_space = 2
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 1
```

What each value means:

- `kernel.randomize_va_space = 2` enables process address space layout
  randomization, including the heap. It isn't a check of kernel address-space
  randomization.
- `kernel.dmesg_restrict = 1` limits the kernel ring buffer to processes with
  `CAP_SYSLOG`, limiting disclosure of addresses and other kernel details.
- `kernel.kptr_restrict = 1` replaces kernel pointers printed through the
  `%pK` format with zeros unless the reader has `CAP_SYSLOG`. It restricts
  disclosure based on credentials. It doesn't hide addresses from a privileged
  process.

Locate configured assignments to the parameter:

```bash
grep -rns -E 'kernel\.(randomize_va_space|dmesg_restrict|kptr_restrict)' \
  /etc/sysctl.conf /etc/sysctl.d /usr/lib/sysctl.d /run/sysctl.d
```

If there's no match, also check kernel defaults, provisioning, and runtime
configuration management.

### Verify build-time protections

Read the configuration for the running kernel:

```bash
grep -E \
  '^CONFIG_(STACKPROTECTOR_STRONG|INIT_ON_ALLOC_DEFAULT_ON|SECURITY_LANDLOCK)=' \
  /boot/config-$(uname -r)
```

Expected result for the image kernel:

```output
CONFIG_STACKPROTECTOR_STRONG=y
CONFIG_INIT_ON_ALLOC_DEFAULT_ON=y
CONFIG_SECURITY_LANDLOCK=y
```

- `CONFIG_STACKPROTECTOR_STRONG` enables stack canaries in compiler-selected
  functions to detect certain stack overwrites.
- `CONFIG_INIT_ON_ALLOC_DEFAULT_ON` enables zero-initialization of memory
  allocations by default. Check `/proc/cmdline` for an `init_on_alloc`
  override before treating the build default as the runtime setting.
- `CONFIG_SECURITY_LANDLOCK` compiles in support for application-controlled
  sandboxing. See [Check Landlock availability](#check-landlock-availability)
  for the runtime check.

### Check Landlock availability

Landlock lets an application restrict its own access. Applications must use
the interface explicitly; compiling Landlock into the kernel doesn't impose
a system-wide sandboxing policy.

List the active Linux security modules:

```bash
cat /sys/kernel/security/lsm
```

The `landlock` entry confirms that the interface is available in the running
kernel. It doesn't establish that any application uses it.

Expected result for the image kernel:

```output
lockdown,capability,yama,selinux,bpf,landlock,ipe,ima,evm
```

### Check kernel lockdown

Kernel lockdown restricts interfaces that let a privileged user modify the
running kernel. The `lockdown` entry in the security module list means that
lockdown support is built into the kernel. It doesn't mean that lockdown is
active.

Read the current lockdown mode:

```bash
cat /sys/kernel/security/lockdown
```

Expected result for the default configuration:

```output
[none] integrity confidentiality
```

The value in brackets is the active mode. By default, the mode is `none`, so
lockdown restrictions aren't applied. The kernel is built to enable lockdown
when it starts with UEFI Secure Boot enabled. Check whether Secure Boot is
enabled on the VM:

```bash
sudo dmesg | grep -i 'secure boot'
```

On the general-purpose image, the result shows `Secure boot disabled`.

### Check processor mitigations

Processor features and microcode mitigations differ by VM size and hardware
generation. Record them per VM size you deploy rather than once per image.

Check for Supervisor Mode Execution Prevention (SMEP) and Supervisor Mode Access
Prevention (SMAP), which support restrictions on supervisor-mode execution and
access to user-space memory:

```bash
grep -m1 -oE '\b(smep|smap)\b' /proc/cpuinfo | sort -u
```

Example output when both features are exposed to the VM:

```output
smap
smep
```

Read the kernel's own summary of processor vulnerabilities and the mitigation
it applied for each one:

```bash
head -n1 /sys/devices/system/cpu/vulnerabilities/*
```

Each file reports `Not affected`, `Mitigation: <method>`, or `Vulnerable`. In a
VM, several of these results depend on host microcode and hypervisor settings
that the guest can't influence, so treat the output as a record of the platform
you were given, not as a configuration item you own. If the directory doesn't
exist, the kernel doesn't publish per-vulnerability status on that platform.

### Interpret service sandboxing

Review systemd sandboxing for the services that support your workload.

List the exposure score for every loaded service:

```bash
systemd-analyze security
```

Inspect one service and the directives behind its score:

```bash
systemd-analyze security sshd.service
```

Example summary line, with the per-directive table omitted:

```output
→ Overall exposure level for sshd.service: 9.6 UNSAFE
```

The score measures configured sandboxing directives, not whether the service is
secure. Some restrictions conflict with legitimate service requirements, such
as creating login sessions. Don't use the score alone as a pass/fail signal.

## Persist sysctl changes and harden systemd services

Changes to kernel parameters and service sandboxing require a documented
workload requirement and compatibility testing before production.

### Persist a parameter change

Configuration files in `/etc/sysctl.d/`, `/run/sysctl.d/`, and
`/usr/lib/sysctl.d/` are applied in filename order regardless of which
directory they're in. The last assignment to a parameter wins. A file in
`/etc/sysctl.d/` replaces a vendor file with the same name. The image also links
`/etc/sysctl.d/99-sysctl.conf` to `/etc/sysctl.conf`.

Choose a local file name after the applicable vendor file and inspect for later
assignments. A numeric prefix alone doesn't guarantee precedence.

> [!CAUTION]
> Tightening kernel parameters can break tools that legitimately read kernel
> state. With `kernel.kptr_restrict = 2`, tools that depend on exposed kernel
> addresses can lose symbol-resolution information even as root. Test the
> change against your profiling and monitoring tools before rollout.

1. Record the current value with `sysctl kernel.kptr_restrict`. This example
   restricts `%pK` pointer output for all users, including privileged users:

   ```bash
   echo 'kernel.kptr_restrict = 2' |
     sudo tee /etc/sysctl.d/90-local-hardening.conf
   ```

1. Apply only the new file and confirm the parameter:

   ```bash
   sudo sysctl --load /etc/sysctl.d/90-local-hardening.conf
   sysctl kernel.kptr_restrict
   ```

   Expected output:

   ```output
   kernel.kptr_restrict = 2
   ```

To undo this example, remove the local file and restore the recorded runtime
value with `sudo sysctl -w kernel.kptr_restrict=<previous-value>`. Confirm the
effective setting again after the next restart.

### Harden a service

To harden a service you own, add a drop-in rather than editing the packaged
unit file:

1. Open a drop-in for the unit:

   ```bash
   sudo systemctl edit <service-name>.service
   ```

1. Add only the directives you've tested, for example:

   ```ini
   [Service]
   NoNewPrivileges=yes
   PrivateTmp=yes
   ProtectHome=read-only
   ```

1. Apply the change and confirm the service still works:

   ```bash
   sudo systemctl restart <service-name>.service
   systemctl status <service-name>.service
   systemd-analyze security <service-name>.service
   ```

   Confirm the expected service state and exercise its workload. A lower score
   doesn't prove that the service still works. If it fails, inspect
   `journalctl -u <service-name>.service` and revert the responsible directive.

## Troubleshoot sysctl and kernel configuration issues

### A parameter doesn't match the documented default

Inspect all assignments to the key and check provisioning or configuration
management. Avoid using `sysctl --system` as a read-only diagnostic: it
reapplies every configured parameter, including network settings.

### A file in /etc/sysctl.d has no effect

Confirm the file name ends in `.conf`, then check whether a file that sorts
later sets the same key:

```bash
grep -rns '<parameter-name>' /etc/sysctl.d /usr/lib/sysctl.d /run/sysctl.d
```

### sysctl reports "cannot stat" for a key

The key doesn't exist on the running kernel. Either the feature isn't compiled
in, or the module or protocol that creates it isn't loaded. IPv6 keys, for
example, disappear when IPv6 is disabled.

### The kernel configuration file is missing

The `/boot/config-*` file ships with the kernel package. A missing file can
indicate that the running kernel's package was removed or that the VM booted
an older entry after an update. Compare the running and installed versions:

```bash
uname -r
rpm -q kernel
```

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Configure SELinux security](./security-enhanced-linux-security.md)
- [Use eBPF](./extended-berkeley-packet-filter.md)
- [Manage the Azure Linux kernel](./kernel-management.md)
- [Kernel sysctl documentation](https://docs.kernel.org/admin-guide/sysctl/kernel.html)
