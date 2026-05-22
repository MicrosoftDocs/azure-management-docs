---
title: Logging and Auditing in Azure Linux
description: Learn how Azure Linux provides security logging and auditing through auditd, journald, and integrations with Azure Monitor and third-party SIEM solutions.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Logging and auditing in Azure Linux

Azure Linux includes built-in security logging and auditing to help you monitor system activity, detect threats, and meet compliance requirements. The default configuration provides persistent journal storage, an enabled audit daemon with opt-in rule profiles, and straightforward integration with centralized log management platforms.

This article explains how Azure Linux leverages `auditd` and `systemd-journald` to capture security-relevant events, how to verify that logging is functioning correctly, and how to query and analyze audit and journal logs. It also covers integration points with Azure Monitor and third-party SIEM solutions for centralized log collection and analysis.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Verify logging configuration

Confirm that audit and journal services are running and review recent security events:

```bash
# Check auditd status
systemctl status auditd

# Check journald status
systemctl status systemd-journald

# View recent authentication events in the journal
journalctl -u sshd --since "1 hour ago" --no-pager
```

## auditd

The Linux Audit daemon (`auditd`) is enabled by default on Azure Linux. The service starts at boot and writes events to `/var/log/audit/audit.log`. Rule profiles are shipped under `/usr/share/audit-rules/` and can be activated by copying the profile you need into `/etc/audit/rules.d/`, then restarting `auditd`.

### Audit rule profiles

Azure Linux ships compliance-oriented audit rule profiles under `/usr/share/audit-rules/` that you can opt into:

| Profile | Files | Use case |
| ------- | ----- | -------- |
| OSPP v4.2 | `30-ospp-v42*.rules` | Common Criteria Operating System Protection Profile baseline; a good general starting point for security event coverage. |
| PCI-DSS v3.1 | `30-pci-dss-v31.rules` | Payment card processing environments. |
| STIG | `30-stig.rules` | U.S. Department of Defense systems that require Security Technical Implementation Guide compliance. |

The directory also includes building-block files such as `10-base-config.rules`, `11-loginuid.rules`, `31-privileged.rules`, `43-module-load.rules`, and `99-finalize.rules`. See `/usr/share/audit-rules/README-rules` for the recommended ordering when combining files.

Activate a profile by copying its rule files into `/etc/audit/rules.d/` and restarting `auditd`. For example, to enable PCI-DSS:

```bash
sudo cp /usr/share/audit-rules/10-base-config.rules \
        /usr/share/audit-rules/11-loginuid.rules \
        /usr/share/audit-rules/30-pci-dss-v31.rules \
        /usr/share/audit-rules/99-finalize.rules \
        /etc/audit/rules.d/
sudo systemctl restart auditd
sudo auditctl -l
```

### View active audit rules

List all currently loaded audit rules:

```bash
sudo auditctl -l
```

### Search audit logs

Use `ausearch` to query audit events by message type or affected file:

```bash
# Search for all authentication events
sudo ausearch -m USER_LOGIN,USER_AUTH

# Search for sudo usage
sudo ausearch -m USER_CMD

# Search for file access to /etc/shadow
sudo ausearch -f /etc/shadow

# Search for kernel module load events
sudo ausearch -m KERN_MODULE
```

## journald

`systemd-journald` stores logs persistently by default on Azure Linux. The image ships with `/var/log/journal/` pre-created, so the journald default `Storage=auto` mode writes to disk and log data survives reboots.

### Default configuration

The default journald configuration is defined in `/etc/systemd/journald.conf`. On Azure Linux, the relevant settings for persistent logging are:

| Setting | Value | Purpose |
| ------- | ----- | ------- |
| `Storage` | `auto` (effectively persistent on Azure Linux) | The Azure Linux image pre-creates `/var/log/journal/`, so the journald `auto` mode stores logs persistently across reboots. |
| Log directory | `/var/log/journal` | Standard location for persistent journal files |

Log rotation is configured by default to prevent disk exhaustion on production systems.

### Query security related events

Use `journalctl` to filter and export log data:

```bash
# View SSH authentication events
journalctl -u sshd --no-pager -n 50

# View all events at warning priority or higher
journalctl -p warning --since "24 hours ago" --no-pager

# View sudo-related events
journalctl _COMM=sudo --no-pager -n 50

# View kernel messages (useful for module load/security events)
journalctl -k --since "1 hour ago" --no-pager
```

### Structured output for SIEM ingestion

Export journal data in JSON format for ingestion into log management platforms:

```bash
# Output in JSON format
journalctl -o json --since "1 hour ago"

# Output in JSON format, one entry per line (ideal for pipeline processing)
journalctl -o json-seq --since "1 hour ago"
```

## Azure Monitor integration

We recommend using Azure Monitor Agent (AMA) to forward security logs from Azure Linux to centralized log management services.

### Azure Monitor Agent (AMA) capabilities

AMA provides the following capabilities for Azure Linux:

- Collects auditd logs, journald entries, and syslog data.
- Forwards events to Azure Sentinel, Log Analytics workspaces, and other Azure Monitor destinations.
- Supports data collection rules (DCRs) for filtering and routing specific event types.

## Third-party SIEM integration

Azure Linux supports forwarding logs to external SIEM platforms through standard syslog forwarding.

### Supported forwarders

`rsyslog` is available in the Azure Linux package repositories. Install and configure it to ship logs to external destinations.

### Configure log forwarding

1. Install rsyslog and enable it:

    ```bash
    # Install rsyslog
    sudo dnf install -y rsyslog
    
    # Enable and start the service
    sudo systemctl enable --now rsyslog
    ```

1. Add a forwarding rule to `/etc/rsyslog.d/50-remote.conf`:

    ```text
    # Forward all security-relevant logs to a remote SIEM
    auth,authpriv.*    @@<siem-server-address>:514
    ```

1. Restart the service to apply changes:

    ```bash
    sudo systemctl restart rsyslog
    ```

## File integrity monitoring

File integrity monitoring (FIM) catches unexpected changes to critical files (binaries, configuration, package state). On Azure Linux you can:

- Use **dm-verity** to enforce integrity of the root filesystem at the block level. dm-verity is mandatory on Azure Container Linux (ACL) and available as an opt-in on general-purpose Azure Linux. For more information, see [Disk and data protection in Azure Linux](./disk-data-protection.md).
- Use **fs-verity** to verify integrity of individual files and signed binaries.
- Use **rpm -V** as a lightweight on-demand integrity check against the package database. For example, running `rpm -Va` verifies all installed packages against the RPM database and reports any discrepancies:

    ```bash
    sudo rpm -Va
    ```

    The output flags any file with a size, mode, owner, or hash that differs from what the installing RPM recorded.

## Related content

- [Azure Linux certifications and compliance](./certifications-compliance.md)
- [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md)
- [Kernel hardening in Azure Linux](./kernel-hardening.md)
