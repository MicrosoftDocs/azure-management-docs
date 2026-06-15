---
title: Manage Azure Linux Packages with DNF5
description: Learn how to use DNF5 to install, upgrade, and remove packages on Azure Linux, including handling repository configuration, caching, and install-only packages like the kernel.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Manage Azure Linux packages with DNF5

This article covers common package management tasks on Azure Linux, including:

- Where repository configuration lives.
- How legacy package-manager commands map to DNF5.
- How DNF caches metadata and resolves packages.
- How kernels and other install-only packages are handled.
- How to enable unattended updates.
- The most commonly used DNF commands.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Repository configuration

DNF5 reads repository definitions from `/etc/yum.repos.d/`. The directory name is preserved from YUM for compatibility; DNF5 reads from this directory.

Each `.repo` file in that directory is a plain-text configuration file that tells DNF where to find packages, whether to verify them with GPG, and whether the repository is enabled by default. The main DNF configuration file is `/etc/dnf/dnf.conf`, which controls global behavior such as cache settings, the install-only limit, and proxy configuration.

## Compatibility symlinks for legacy commands

Azure Linux ships compatibility symlinks so that legacy commands (`yum`, `dnf` (DNF4), `microdnf`, `tdnf`) all dispatch to DNF5:

```text
lrwxrwxrwx.  1 root root       4  yum -> dnf5
lrwxrwxrwx.  1 root root       4  microdnf -> dnf5
-rwxr-xr-x.  1 root root 1614408  dnf5
lrwxrwxrwx.  1 root root       4  dnf -> dnf5
lrwxrwxrwx.  1 root root       4  tdnf -> dnf5
```

The common subcommands all work through the symlinks. Edge cases that depended on DNF4 internals or `microdnf`-specific behavior might not. For details on what changed, see the following resources:

- [Changes between DNF and DNF5](https://dnf5.readthedocs.io/en/latest/changes_from_dnf4.7.html)
- [Migrating to DNF5](https://dnf5.readthedocs.io/en/latest/migrating_to_dnf5.7.html)
- [Changes in DNF CLI compared to YUM](https://dnf.readthedocs.io/en/latest/cli_vs_yum.html)

## How DNF5 resolves packages

When you run a DNF command, DNF5 performs the following steps to determine what packages to install, upgrade, or remove:

1. Loads every `.repo` file in `/etc/yum.repos.d/` and selects the repositories marked as enabled.
1. Refreshes repository metadata, caching it locally under `/var/cache/libdnf5`. Cached metadata includes the package list, version and dependency information, checksums, and signatures.
1. Resolves the requested install, remove, or upgrade against the enabled repositories, picks package versions, and verifies integrity and signatures before applying the transaction.

Disabled repositories are ignored unless you enable them explicitly with `--enablerepo=<repo-id>` for a single command.

The `/var/cache/libdnf5` cache exists so DNF doesn't have to re-download metadata on every invocation. Run `dnf clean all` to empty it (see [Frequently used DNF commands](#frequently-used-dnf-commands)).

## Install-only packages and multiple installed versions

Some packages, most notably the kernel, are installed side by side rather than upgraded in place. DNF5 controls this through two settings in `/etc/dnf/dnf.conf`:

- **`installonlypkgs`**: List of package names that are never upgraded in place. Kernel packages are the canonical entry.
- **`installonly_limit`**: Maximum number of versions of an install-only package that might coexist on the system. The default is `3`.

When a new version is installed and the limit is exceeded, DNF removes the oldest version automatically.

For the kernel, this gives you:

- **Fallback**: If a new kernel fails to boot, GRUB can still launch a previous, known-working kernel.
- **Validation**: You can install and test a new kernel without removing the running one.
- **Variant coexistence**: You can install different kernel builds (for example, with different feature sets) at the same time.

Each kernel version is a separate RPM, so DNF tracks and removes them like any other package.

## Unattended updates with `dnf-automatic`

`dnf-automatic` is a service that periodically checks for, downloads, and optionally applies package updates. It's driven by a systemd timer rather than running as a long-lived daemon.

### Install `dnf-automatic`

Install `dnf-automatic` using the following command:

```bash
sudo dnf install -y dnf-automatic
```

### Enable the `dnf5-automatic` timer

Enable the `dnf5-automatic` timer immediately and across reboots using the following command:

```bash
sudo systemctl enable --now dnf5-automatic.timer
```

- `enable` configures the timer to start at boot.
- `--now` starts the timer in the current session.

### Check status of the `dnf5-automatic` timer

Check that the timer is active and see when it last ran using the following command:

```bash
systemctl status dnf5-automatic.timer
```

Typical output looks like the following example:

```output
● dnf5-automatic.timer - dnf-automatic timer
     Loaded: loaded (/usr/lib/systemd/system/dnf5-automatic.timer; enabled; preset: disabled)
     Active: active (waiting) since Thu 2026-03-26 14:36:54 UTC; 13min ago
 Invocation: f6984eeceb7c40df99bca243f3da728c
    Trigger: Fri 2026-03-27 06:34:18 UTC; 15h left
   Triggers: ● dnf5-automatic.service
```

`Active: active (waiting)` is expected: the underlying service is a one-shot that runs only when the timer fires, then exits. The `Trigger:` line shows the next scheduled run.

### List the next scheduled run of the `dnf5-automatic` timer

List the next run alongside any other timers using the following command:

```bash
systemctl list-timers dnf5-automatic.timer
```

Example output:

```output
NEXT                        LEFT LAST PASSED UNIT                 ACTIVATES
Fri 2026-03-27 06:34:18 UTC  15h -         - dnf5-automatic.timer dnf5-automatic.service

1 timers listed.
Pass --all to see loaded but inactive timers, too.
```

### Optional configuration of `dnf-automatic`

You can configure automatic update behavior by adding the `/etc/dnf/automatic.conf` file. This file controls whether updates are only downloaded, automatically applied, or restricted to security‑only updates. For full configuration details, see the [dnf5‑automatic documentation](https://dnf5.readthedocs.io/en/stable/dnf5_plugins/automatic.8.html)

## Frequently used DNF commands

The following commands cover the bulk of day-to-day package management on Azure Linux.

### `dnf clean all`

`dnf clean all` removes everything DNF has cached under `/var/cache/libdnf5`. Use it to:

- Recover from corrupted or stale metadata that's causing transaction errors.
- Shrink container images by stripping the package cache after installs in the same `RUN` layer.
- Reclaim disk space.

Container example:

```dockerfile
RUN dnf install -y package1 package2 \
    && dnf clean all
```

### `dnf search`

`dnf search` matches keywords against package names and summaries. For example:

```bash
dnf search lsof
```

Example output:

```output
Updating and loading repositories:
 Azure Linux 4.0 - x86_64 - Microsoft                        100% |   7.6 KiB/s |   4.8 KiB |  00m01s
Repositories loaded.
Matched fields: name (exact)
 lsof.x86_64   A utility which lists open files on a Linux/UNIX system
```

### `dnf provides`

`dnf provides` finds the package that owns a given file, command, or capability. For example:

```bash
dnf provides lsof
```

Example output:

```output
Updating and loading repositories:
Repositories loaded.
lsof-4.98.0-8.azl4.20260303.x86_64 : A utility which lists open files on a Linux/UNIX system
Repo         : @System
Matched From :
Provide      : lsof = 4.98.0-8.azl4.20260303
```

### `dnf install`

`dnf install` installs one or more packages along with their runtime dependencies:

```bash
sudo dnf install lsof
```

Example output:

```output
Updating and loading repositories:
Repositories loaded.
Package           Arch     Version                  Repository    Size
Installing:
 lsof             x86_64   4.98.0-8.azl4.20260303   azurelinux  578.5 KiB

Transaction Summary:
 Installing:         1 package

Total size of inbound packages is 226 KiB. Need to download 226 KiB.
After this operation, 579 KiB extra will be used (install 579 KiB, remove 0 B).
Is this ok [y/N]: y
[1/1] lsof-0:4.98.0-8.azl4.20260303.x86_64                100% | 641.9 KiB/s | 226.0 KiB |  00m00s
[1/1] Total                                               100% | 640.1 KiB/s | 226.0 KiB |  00m00s
Running transaction
[1/3] Verify package files                                100% | 500.0   B/s |   1.0   B |  00m00s
[2/3] Prepare transaction                                 100% |  23.0   B/s |   1.0   B |  00m00s
[3/3] Installing lsof-0:4.98.0-8.azl4.20260303.x86_64     100% |   3.0 MiB/s | 580.2 KiB |  00m00s
Complete!
```

### `dnf remove`

`dnf remove` uninstalls a package along with any dependencies that were pulled in solely for it:

```bash
sudo dnf remove -y lsof
```

Example output:

```output
Package           Arch     Version                  Repository    Size
Removing:
 lsof             x86_64   4.98.0-8.azl4.20260303   azurelinux  578.5 KiB

Transaction Summary:
 Removing:           1 package

After this operation, 579 KiB will be freed (install 0 B, remove 579 KiB).
Is this ok [y/N]: y
Running transaction
[1/2] Prepare transaction                                 100% |  27.0   B/s |   1.0   B |  00m00s
[2/2] Removing lsof-0:4.98.0-8.azl4.20260303.x86_64       100% |  72.0   B/s |  11.0   B |  00m00s
Complete!
```

### `dnf upgrade`

`dnf upgrade` updates installed packages to their latest available versions.

```bash
sudo dnf upgrade
```

When everything is current, the output is short. For example:

```output
Updating and loading repositories:
Repositories loaded.
Nothing to do.
```

> [!TIP]
> Pass `-y` to `install`, `remove`, or `upgrade` in non-interactive contexts such as scripts, CI pipelines, and container builds. Any DNF operation that modifies the system requires root, so run it under `sudo` or as `root`.

## Related content

- [DNF5 package management utility](https://dnf5.readthedocs.io/en/latest/dnf5.8.html): Full command-line reference for `dnf5`.
- [Changes between DNF and DNF5](https://dnf5.readthedocs.io/en/latest/changes_from_dnf4.7.html): What changed from DNF4, including removed commands and behavior differences.
- [Migrating to DNF5](https://dnf5.readthedocs.io/en/latest/migrating_to_dnf5.7.html): Guidance for moving scripts and automation from DNF4 to DNF5.
