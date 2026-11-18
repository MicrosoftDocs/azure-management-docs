---
title: Use eBPF on Azure Linux
description: Inspect and tighten eBPF restrictions on Azure Linux, inventory loaded programs with bpftool, and install the packages needed to build eBPF programs.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Use eBPF on Azure Linux

Extended Berkeley Packet Filter (eBPF) supports in-kernel observability,
networking, and security tools.

## Default eBPF restrictions

The image disables unprivileged loading with
`kernel.unprivileged_bpf_disabled=2`. Just-in-time (JIT) compilation is enabled
(`net.core.bpf_jit_enable=1`), and JIT constant blinding is off
(`net.core.bpf_jit_harden=0`). To turn on constant blinding, see
[Decide whether to harden the JIT](#decide-whether-to-harden-the-jit).

`bpftool`, kernel development files, and eBPF build libraries aren't installed
on the image.

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.

The program inventory and build checks on this page require packages that
aren't installed by default. Install them before you run those commands:

```bash
sudo dnf install -y bpftool kernel-headers kernel-devel libbpf-devel
```

## Verify eBPF restrictions and loaded programs

Runtime restrictions, kernel build options, and loaded programs provide
separate views of eBPF use.

### Read the current restrictions

```bash
sysctl \
  kernel.unprivileged_bpf_disabled \
  net.core.bpf_jit_enable \
  net.core.bpf_jit_harden
```

Expected result for the default configuration:

```output
kernel.unprivileged_bpf_disabled = 2
net.core.bpf_jit_enable = 1
net.core.bpf_jit_harden = 0
```

`kernel.unprivileged_bpf_disabled` has three states, and the difference between
two of them matters for auditing:

| Value | Behavior |
| ----- | -------- |
| `0` | Any process can call `bpf()`. |
| `1` | Unprivileged calls are rejected, and the value can't be lowered again until the system restarts. |
| `2` | Unprivileged calls are rejected, and a privileged process can still set the value back to `0`. |

### Verify the build-time configuration

```bash
grep -E \
  '^CONFIG_(BPF|BPF_SYSCALL|BPF_JIT|BPF_JIT_ALWAYS_ON|BPF_UNPRIV_DEFAULT_OFF)=' \
  /boot/config-$(uname -r)
```

Expected result for the image kernel:

```output
CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_BPF_JIT_ALWAYS_ON=y
CONFIG_BPF_UNPRIV_DEFAULT_OFF=y
```

- `CONFIG_BPF_JIT_ALWAYS_ON` compiles out the eBPF interpreter, so a program
  either compiles to native code or doesn't run.
- `CONFIG_BPF_UNPRIV_DEFAULT_OFF` is why `kernel.unprivileged_bpf_disabled`
  starts at `2` rather than `0`.

### Inventory what's loaded

List loaded programs and maps. These commands require the `bpftool` package
from [Prerequisites](#prerequisites):

```bash
bpftool version
sudo bpftool prog list
sudo bpftool map show
```

The output is similar to the following example. Program IDs, tags, load times,
and process IDs vary:

```output
bpftool v7.6.0
using libbpf v1.6
features: llvm, skeletons
41: lsm  name restrict_filesystems  tag 31efb7c56239148b  gpl
        loaded_at 2026-10-30T10:07:35+0000  uid 0
        xlated 520B  jited 297B  memlock 4096B  map_ids 22
        btf_id 45
        pids systemd(1)
94: cgroup_skb  name sd_fw_egress  tag 772db7720b2728e9  gpl
        loaded_at 2026-10-30T10:13:07+0000  uid 0
        xlated 64B  jited 67B  memlock 4096B
        pids systemd(1)
...
```

Systemd and container runtimes can load programs, so a nonempty list isn't
necessarily unexpected. On the default image, systemd loads programs such as
`restrict_filesystems`, `sd_fw_egress`, `sd_fw_ingress`, and `sd_devices`.
Investigate entries you can't attribute to a deployed component:

```bash
sudo bpftool prog show id <program-id>
```

Use the program type, name, load time, and available process information to
correlate it with an agent. A loaded program isn't necessarily attached to a
hook, and its owner ID alone doesn't identify the application.

## Harden eBPF and prepare a build environment

Changes to loading restrictions or JIT settings can affect agents that use
eBPF. Test compatibility and performance before rollout.

### Make the loading restriction one-way

If your policy requires that the restriction can't be lifted while the system
runs, set it to `1`:

> [!CAUTION]
> You can't return to `0` or `2` without restarting the VM. Confirm first that
> nothing on the system loads eBPF programs without privileges. Agents
> typically run with `CAP_BPF` or `CAP_SYS_ADMIN` and aren't affected.

```bash
echo 'kernel.unprivileged_bpf_disabled = 1' |
  sudo tee /etc/sysctl.d/90-bpf-hardening.conf
sudo sysctl --load /etc/sysctl.d/90-bpf-hardening.conf
sysctl kernel.unprivileged_bpf_disabled
```

### Decide whether to harden the JIT

`net.core.bpf_jit_enable = 1` means programs are compiled to native code
instead of interpreted. `net.core.bpf_jit_harden` controls constant blinding,
which stops an attacker from smuggling chosen values into executable memory
through program constants:

| Value | Effect |
| ----- | ------ |
| `0` | No blinding. |
| `1` | Blinds constants in programs loaded by unprivileged users. |
| `2` | Blinds constants in all programs, including privileged ones. |

With unprivileged loading disabled, use `2` if your requirement also covers
privileged loaders. Measure its effect on your agents and network performance:

```bash
echo 'net.core.bpf_jit_harden = 2' |
  sudo tee /etc/sysctl.d/90-bpf-jit-harden.conf
sudo sysctl --load /etc/sysctl.d/90-bpf-jit-harden.conf
sysctl net.core.bpf_jit_harden
```

To undo the JIT change, remove the file and restore the prior runtime value
with `sudo sysctl -w net.core.bpf_jit_harden=<previous-value>`. Unlike the
one-way unprivileged-loading restriction, this setting can be changed without
a restart.

### Install build dependencies

Kernel development files and eBPF build libraries aren't installed on the
image. Install them before you build eBPF programs:

```bash
sudo dnf install -y kernel-headers kernel-devel libbpf-devel
```

Follow your loader's build requirements. Check the kernel development package
and whether the kernel exposes BPF Type Format (BTF) information for
compile-once-run-everywhere (CO-RE) relocations:

```bash
uname -r
rpm -q kernel-devel
ls -l /sys/kernel/btf/vmlinux
```

After you install the build dependencies, the result is similar to the
following example. The `kernel-devel` version matches the running kernel:

```output
6.18.50-1.5.azl4.x86_64
kernel-devel-6.18.50-1.5.azl4.x86_64
-r--r--r--. 1 root root 6642883 Oct 30 10:22 /sys/kernel/btf/vmlinux
```

If `rpm -q kernel-devel` returns `package kernel-devel is not installed`,
install the build dependencies first.

For builds that require matching kernel development files, compare the package
version with the running kernel. If `/sys/kernel/btf/vmlinux` is absent, check
the tool's BTF requirements before attempting to load it.

## Troubleshoot eBPF loading and build failures

### Loading a program fails with "Operation not permitted"

That's the expected result for an unprivileged process. Run the loader with
root privileges, or grant the service `CAP_BPF` and the capabilities its
program type needs.

### Setting kernel.unprivileged_bpf_disabled back to 0 fails

The value was previously set to `1`, which the kernel treats as one-way. Remove
the setting from `/etc/sysctl.d/` and restart the VM.

### bpftool prog list returns nothing

Confirm the command succeeded with `sudo`. An empty successful result means no
loaded programs were listed; it isn't an attachment-specific check.

### A build fails after a kernel update

The `kernel-devel` package now matches the newly installed kernel, not the one
you're running. This check requires the `kernel-devel` package from
[Prerequisites](#prerequisites). Restart into the updated kernel, then rebuild:

```bash
uname -r
rpm -q kernel kernel-devel
```

The running kernel version and the `kernel-devel` version must match, as in
the following example:

```output
6.18.50-1.5.azl4.x86_64
kernel-6.18.50-1.5.azl4.x86_64
kernel-devel-6.18.50-1.5.azl4.x86_64
```

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Inspect kernel hardening](./kernel-hardening.md)
- [Manage the Azure Linux kernel](./kernel-management.md)
- [eBPF documentation](https://docs.kernel.org/bpf/index.html)
