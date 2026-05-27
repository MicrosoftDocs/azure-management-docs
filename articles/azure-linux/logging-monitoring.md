---
title: Logging and Monitoring on Azure Linux
description: Learn how to view system logs with the systemd journal, locate traditional application log files, and use standard Linux utilities to monitor system health and performance on Azure Linux.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Logging and monitoring on Azure Linux

Azure Linux is built on [`systemd`](https://systemd.io/) and uses the systemd journal as its primary logging system. Logs from the kernel, system services, and applications are collected and indexed by `journalctl`, which provides a unified, queryable view of system activity. Azure Linux ships with the standard set of Linux tools for monitoring filesystems, disks, memory, processes, services, and the network.

This article shows you how to view system logs through the systemd journal, find traditional application log files, and use the standard utilities for monitoring system health and performance on an Azure Linux system.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- An Azure Linux system that you can authenticate to with shell access.
- A user account with `sudo` privileges. Some logs and audit events require elevated permissions to read.

## View system logs with the systemd journal

The systemd journal is the central logging system on Azure Linux. Use the `journalctl` command to query and filter log entries from the kernel, system services, and applications.

### View all log entries

View every log entry that the journal has collected using the following command:

```bash
journalctl
```

The command opens the journal in a pager so you can scroll through entries from oldest to newest.

### View logs for a specific service

View only the log entries that were emitted by a specific service using the following command. Replace `<SERVICE>` with the service name, such as `sshd` or `systemd-networkd`:

```bash
journalctl -u <SERVICE>
```

For example, to view the SSH service logs, run:

```bash
journalctl -u sshd
```

### Follow logs in real time

Follow new log entries as they're written, similar to `tail -f`, using the following command:

```bash
journalctl -f
```

Use `Ctrl+C` to stop following the journal.

### Filter logs by priority

Show only entries at or above a given severity by passing the priority to `journalctl`. For example, to show only error-level entries and higher, run the following command:

```bash
journalctl -p err
```

Valid priorities range from `emerg` (most severe) to `debug` (least severe).

> [!NOTE]
> The systemd journal stores logs persistently under `/var/log/journal/` when that directory exists. If `/var/log/journal/` is missing, journal data is kept only in memory and is lost on reboot.
> `journalctl` integrates with systemd timers for automated logging.

## View traditional log files

Many applications and services also write traditional log files to `/var/log/`. The following table lists common locations on an Azure Linux virtual machine (VM):

| Path | Purpose |
| ---- | ------- |
| `/var/log/messages` | General system messages from the kernel and system services. |
| `/var/log/secure` | Authentication and `sudo` events. |
| `/var/log/dmesg` | Kernel messages from the most recent boot. |
| `/var/log/httpd/` | Apache HTTP Server logs, when `httpd` is installed. |
| `/var/log/mysql/` | MySQL or MariaDB logs, when those packages are installed. |

Follow a traditional log file in real time using the following command. Replace the path with the log file that you want to follow:

```bash
sudo tail -f /var/log/messages
```

### View kernel ring buffer messages

View kernel ring buffer messages directly using the following command:

```bash
dmesg
```

## Monitor system health and performance

Azure Linux includes the standard set of Linux utilities for monitoring filesystems, memory, processes, services, and the network.

### Monitor filesystems and disks

Use the following commands to inspect disk space, block devices, and filesystem state:

- `df -h`: Shows disk usage of mounted filesystems in human-readable units.
- `du -sh <DIRECTORY>`: Shows the total disk usage of a directory.
- `lsblk`: Lists block devices and their relationships in a tree view.
- `blkid`: Displays the filesystem type and UUID for each block device.
- `tune2fs`: Inspects or modifies parameters of an `ext2`, `ext3`, or `ext4` filesystem.
- `fsck.ext4`: Checks and repairs an `ext4` filesystem.

### Monitor memory and processes

Use the following commands to inspect memory usage, CPU usage, and running processes:

- `free -h`: Shows memory and swap usage in human-readable units.
- `top`: Provides a real-time view of CPU, memory, and process activity.
- `htop`: Provides an interactive, color-coded alternative to `top`.
- `iostat`: Reports CPU and disk I/O statistics.
- `vmstat`: Reports memory, CPU, and I/O statistics at fixed intervals.
- `sar`: Collects and reports historical CPU, memory, disk, and network usage. `sar` is provided by the `sysstat` package.

### Monitor services and system health

Use the following commands to inspect service state and overall system health:

- `systemctl status <SERVICE>`: Shows whether a service is loaded, active, and enabled at boot, along with recent log entries.
- `systemd-analyze`: Analyzes system boot performance, including the time spent in each unit.
- `uptime`: Shows how long the system has been running, along with current load averages.

### Monitor the network

Use the following commands to inspect listening ports and active network connections:

- `ss`: Lists active sockets, listening ports, and established connections. `ss` is the modern replacement for `netstat`.
- `netstat`: Legacy tool for listing network connections, when `ss` isn't available or when you need its specific output format.

## Audit and rotate logs

Azure Linux includes extra tooling for auditing security-relevant events and managing log file growth:

- `auditd`: Collects audit records for system calls, file access, and security events.
- `ausearch`: Queries audit log records that `auditd` produced.
- `aureport`: Generates summary reports from audit log records.
- `logrotate`: Rotates, compresses, and removes log files according to a configured retention policy.

## Related content

To learn more about Azure Linux system management, see [Get started with Azure Linux system management](./get-started-system-management.md).
