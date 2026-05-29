---
title: Manage the Azure Linux Kernel
description: Learn how to inspect the running Azure Linux kernel, manage kernel modules, and view or modify kernel parameters at runtime.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/29/2026
---

# Manage the Azure Linux kernel

Azure Linux is built on the latest declared long-term support (LTS) Linux kernel, version 6.18. This kernel provides a balanced combination of stability, security, and modern hardware and cloud support, and it serves as the foundation for every Azure Linux deployment.

This article shows you how to inspect the running kernel, work with kernel modules, and view or change kernel parameters at runtime on an Azure Linux system.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- An Azure Linux system that you can authenticate to with shell access.
- A user account with `sudo` privileges, which is required to load or unload kernel modules and to change kernel parameters.

## Inspect the running kernel

The following sections show how to gather information about the kernel that your Azure Linux system currently runs, including the kernel version, boot parameters, build configuration, and installed kernel packages.

### Display the kernel version

Identify the exact version of the kernel that your system currently runs using the `uname -r` command:

```bash
uname -r
```

The output shows the kernel release string, which includes the upstream version, the Azure Linux build identifier, and the architecture.

### Display the kernel boot parameters

Display the boot parameters passed to the kernel when the system started using the `cat /proc/cmdline` command:

```bash
cat /proc/cmdline
```

The output shows the kernel command line, including the root device, console settings, and any boot-time options applied by the bootloader.

### Display the kernel build configuration

Show all the compile-time configuration options used to build the running kernel using the `cat /boot/config-$(uname -r)` command:

```bash
cat /boot/config-$(uname -r)
```

This command lists thousands of build-time options, including enabled and disabled features (`CONFIG_*`), security settings, networking options, and filesystem support, that determine how the kernel was compiled.

The following example shows a small portion of the output:

```output
CONFIG_RUSTC_HAS_FILE_WITH_NUL=y
CONFIG_RUSTC_HAS_FILE_AS_C_STR=y
CONFIG_IRQ_WORK=y
CONFIG_TIME_KUNIT_TEST=m
CONFIG_BPF_PRELOAD_UMD=m
CONFIG_IKHEADERS=m
# CONFIG_PREEMPT is not set
# CONFIG_PREEMPT_RT is not set
# CONFIG_PSI_DEFAULT_DISABLED is not set
```

#### Configuration value meanings

Each configuration entry uses one of the following values:

| Value | Meaning | Description |
| ----- | ------- | ----------- |
| `=y` | Built-in | The feature is compiled statically into the main kernel image and is loaded into memory as soon as the kernel boots. |
| `=m` | Module | The feature is compiled as a loadable kernel module, provided that module support is enabled in the kernel. |
| `is not set` | Disabled | The feature isn't compiled at all. This state is often shown as a commented-out line, such as `# CONFIG_X is not set`. |

### List installed kernel packages

Multiple copies of `/boot/config-*` might exist on your system, with one file for each installed kernel. You can list every kernel package that's currently installed using the following command:

```bash
rpm -qa | grep -i kernel
```

The output includes every installed package with a name that contains `kernel`, such as the kernel image, kernel headers, and any kernel-related tooling.

## Work with kernel modules

Kernel modules are pieces of code that the kernel can load and unload on demand to add or remove functionality, such as device drivers and filesystem support, without requiring a reboot.

### Locate kernel modules on disk

Kernel modules are stored under the following directory:

```bash
/lib/modules
```

Each installed kernel version has its own subdirectory in `/lib/modules`, which contains the modules that were built for that specific kernel.

#### Browse modules for the running kernel

Browse the modules for the kernel that you're currently running using the following path:

```bash
/lib/modules/$(uname -r)
```

#### Inspect modules for a different kernel version

Inspect the modules for a different installed kernel version by listing the available directories under `/lib/modules`:

```bash
ls /lib/modules
```

#### Locate kernel module files

Locate every kernel module file (`.ko`) for the kernel that you're currently running using the following command:

```bash
find /lib/modules/$(uname -r) -type f -name "*.ko*"
```

### Load, unload, and list modules

Use the `modprobe` command to load or unload a kernel module, and use the `lsmod` command to see which modules are currently loaded in the kernel.

#### Load a kernel module

Load a module using the following command. Replace `<MODULE_NAME>` with the name of the module that you want to load:

```bash
sudo modprobe <MODULE_NAME>
```

#### Unload a kernel module

Unload a module using the following command. Replace `<MODULE_NAME>` with the name of the module that you want to unload:

```bash
sudo modprobe -r <MODULE_NAME>
```

#### List currently loaded kernel modules

List the modules that are currently loaded in the kernel using the following command:

```bash
lsmod
```

The output lists each loaded module, the amount of memory that it uses, and the modules or subsystems that depend on it.

## View and modify kernel parameters at runtime

Use the `sysctl` tool to view and modify Linux kernel parameters while the system is running. Kernel parameters control areas such as:

- Networking behavior.
- Memory allocation.
- General system behavior.
- Filesystem and I/O behavior.
- Kernel security features.

### View all kernel parameters

Display every kernel parameter and its current value using the following command:

```bash
sudo sysctl -a
```

The following example shows a small portion of the output:

```output
kernel.ftrace_dump_on_oops = 0
kernel.oops_all_cpu_backtrace = 0
kernel.oops_limit = 10000
kernel.panic_on_oops = 0
```

### View a single kernel parameter

View a single parameter by passing its name to `sysctl`. The following example command shows how to view the value of `kernel.panic_on_oops`:

```bash
sysctl kernel.panic_on_oops
```

### Set a temporary kernel parameter

Change a kernel parameter for the current boot only using the following command.  Replace `<SYSCTL_PARAMETER>` and `<VALUE>` with the parameter name and the value that you want to set:

```bash
sudo sysctl -w <SYSCTL_PARAMETER>=<VALUE>
```

The change takes effect immediately and remains in effect until the next reboot.

### Set a persistent kernel parameter

1. Make a kernel parameter change persist across reboots, add the parameter to `/etc/sysctl.conf` in the following form:

    ```text
    <SYSCTL_PARAMETER>=<VALUE>
    ```

1. Apply the updated configuration without rebooting using the following command:

    ```bash
    sudo sysctl -p
    ```

    After `sysctl -p` runs successfully, the new value is applied to the running kernel and is also reapplied automatically each time the system starts.

## Enable kdump

> [!IMPORTANT]
> Kdump requires a dedicated disk space for storing crash dumps. Ensure that `/var/crash` is mounted on a disk with at least as much space as the total physical memory of the system to avoid running out of space when a crash occurs.

1. Install kdump-utils and makedumpfile packages using the following command:

    ```bash
    sudo dnf install -y kdump-utils makedumpfile
    ```

1. Determine the recommended crash kernel size to add using the following command:

    ```bash
    sudo kdumpctl estimate
    ```

    In the output, refer to the _Recommended crashkernel_ value. For example, the following output shows a recommended crashkernel size of 256M:

    ```output
    kdump: Rebuilding /boot/initramfs-6.18.5-1.8.azl4~20260420.x86_64kdump.img
    Reserved crashkernel:    0M
    Recommended crashkernel: 256M 
    
    Kernel image size:   56M
    Kernel modules size: 11M
    Initramfs size:      35M
    Runtime reservation: 16M
    Large modules:
        mlx5_core: 2826240
        cfg80211: 1380352
        kvm: 1433600
        btrfs: 2097152
    WARNING: Current crashkernel size is lower than recommended size 256M.
    ```

1. Add the _Recommended crashkernel_ value to the kernel boot parameters using the following command:

    ```bash
    sudo grubby --args="crashkernel=256M" --update-kernel=ALL
    ```

1. Configure kdump systemd service to start automatically during the boot process using the following command:

    ```bash
    sudo systemctl enable kdump.service
    ```

    Example output:

    ```output
    Created symlink '/etc/systemd/system/multi-user.target.wants/kdump.service' → '/usr/lib/systemd/system/kdump.service'.
    ```

1. Reboot your system for the change to be effective.
1. After the reboot, you can validate that kdump is operational using the following command:

    ```bash
    sudo kdumpctl status
    ```

    Example output:

    ```output
    kdump: Kdump is operational
    kdump: Notice: No vmcore creation test performed!
    ```

## Related content

For more information about the Linux kernel parameters and the `sysctl` tool, see the following upstream resources:

- [Kernel `sysctl` parameters](https://www.kernel.org/doc/Documentation/sysctl/kernel.txt): The upstream reference for every parameter that's exposed under `kernel.*`, including allowed values and default behavior.
- [`sysctl(8)` man page](https://man7.org/linux/man-pages/man8/sysctl.8.html): Full command-line reference for the `sysctl` tool, including how to load configuration from `/etc/sysctl.d/` drop-in files.
