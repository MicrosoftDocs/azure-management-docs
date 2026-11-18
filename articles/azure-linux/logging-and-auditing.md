---
title: Configure security logging and auditing in Azure Linux
description: Configure audit rules, record changes to SSH configuration, manage log retention, and investigate security events on Azure Linux 4.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Configure security logging and auditing in Azure Linux

Audit rules select the security events recorded by `auditd`. The system
journal records service messages separately.

## Default auditd rules and journal settings

The audit daemon (`auditd`) is enabled and running. Its default rule disables
syscall and file-watch auditing, so configure rules for that coverage. The
system journal is persistent, using `Storage=auto` with `/var/log/journal/`
present.

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.

## Verify auditd rules and journal persistence

Check the daemon, loaded rules, and journal storage before adding coverage or
changing retention.

### Check what's recorded today

Confirm the audit daemon is running and will start again after a restart:

```bash
systemctl is-enabled auditd
systemctl is-active auditd
```

Expected result for the default configuration:

```output
enabled
active
```

List the loaded rules:

```bash
sudo auditctl -l
```

The image loads a single rule:

```output
-a never,task
```

The source file expresses this rule as `-a task,never`. It tells the kernel not
to create syscall audit context for any process, so syscall and file-watch
rules that you add after it never match. This is the image default, which
keeps syscall auditing off until you define the coverage your workload needs.
Remove the rule before adding syscall or file-watch rules, as shown in
[Record SSH configuration changes](#record-ssh-configuration-changes). Other
events, including authentication and `sudo` records submitted by applications,
still reach the audit log.

Locate the source rule:

```bash
sudo ls -1 /etc/audit/rules.d/
sudo grep -rnE '^-a (task,never|never,task)$' \
  /etc/audit/rules.d/ \
  /etc/audit/audit.rules
```

Expected result for the default configuration:

```output
audit.rules
/etc/audit/rules.d/audit.rules:12:-a task,never
/etc/audit/audit.rules:5:-a task,never
```

`/etc/audit/audit.rules` is generated from the files in
`/etc/audit/rules.d/`. Edit the files in `rules.d`, not the generated file.

### Confirm the journal persists

The journal stores logs persistently only when the configuration allows it and
the directory exists. Check both:

```bash
systemd-analyze cat-config systemd/journald.conf | grep -E '^#?Storage='
test -d /var/log/journal && echo "Persistent journal directory exists"
journalctl --disk-usage
```

The vendor default is `Storage=auto`, and the packaged file carries it as a
comment. Expected lines for the default configuration:

```output
#Storage=auto
Persistent journal directory exists
```

With `Storage=auto`, the presence of `/var/log/journal/` is what makes the
journal persist across restarts. `journalctl --disk-usage` reports the current
size, which varies with workload and retention.

## Add audit rules and set log retention

Audit coverage and retention depend on monitoring requirements. The SSH watch
below records one type of activity; it isn't a complete audit policy.

### Record SSH configuration changes

This example watches writes and attribute changes under `/etc/ssh/`. It assumes
the default rule file hasn't been customized. If you already maintain audit
rules, integrate the watch into your existing ruleset instead of replacing it.

> [!CAUTION]
> Add rules on a nonproduction VM first and monitor event volume. If
> `sudo auditctl -s` reports `enabled 2`, the running rules are immutable. See
> [Change an immutable ruleset](#change-an-immutable-ruleset) before continuing.

1. Back up and remove the default no-audit rules file:

   ```bash
   sudo cp -a \
     /etc/audit/rules.d/audit.rules \
     /etc/audit/rules.d/audit.rules.bak
   sudo rm /etc/audit/rules.d/audit.rules
   ```

1. Install the base configuration and the SSH watch:

   ```bash
   sudo cp \
     /usr/share/audit-rules/10-base-config.rules \
     /etc/audit/rules.d/
   echo '-w /etc/ssh/ -p wa -k ssh-config' |
     sudo tee /etc/audit/rules.d/70-ssh-config.rules
   ```

   `w` records writes, `a` records attribute changes, and `ssh-config` is the
   search key. This rule doesn't record the contents of files.

1. Load the rules and check the result:

   ```bash
   sudo augenrules --load
   sudo auditctl -l
   sudo auditctl -s
   ```

   Review the output from `augenrules --load`. If a rule can't be parsed,
   `auditctl` reports the offending line. Confirm that `auditctl -l` no longer
   contains `never,task`.

   Watch the `lost` counter for discarded kernel audit records. An increasing
   value can indicate that the event rate exceeds the collection capacity.

#### Verify the watch

1. Create a temporary file under the watched directory. Use a name that doesn't
   already exist. This file isn't an SSH configuration file:

   ```bash
   sudo touch /etc/ssh/azl-audit-example
   ```

1. Query the event by its key:

   ```bash
   sudo ausearch -k ssh-config -ts recent
   ```

   Find a `type=SYSCALL` record with `comm="touch"`, `success=yes`, and
   `key="ssh-config"`. Its associated path record identifies the file.
   `auid` is the login identity and normally survives `sudo`; automation without
   a login session can report it as unset (`4294967295`). Don't attribute an
   unset identity to a person.

1. Remove the temporary file. Keep the watch for ongoing monitoring:

   ```bash
   sudo rm /etc/ssh/azl-audit-example
   ```

Additional sample rules and their combination instructions are in
`/usr/share/audit-rules/README-rules`. Samples named after standards aren't
evidence of compliance and require review for your workload.

### Set retention that matches your policy

Read the retention and low-space actions before changing them:

```bash
sudo grep -E \
  '^(max_log_file|num_logs|max_log_file_action|space_left_action|admin_space_left_action|disk_full_action)' \
  /etc/audit/auditd.conf
```

| Setting or action | Effect |
| ----------------- | ------ |
| `max_log_file` | Size threshold for an individual log, in megabytes. |
| `ROTATE` | Rotates logs and uses `num_logs` to limit retained files. Older records are eventually overwritten. |
| `KEEP_LOGS` | Retains rotated logs without the `num_logs` limit. Requires capacity monitoring and an archival policy. |
| `SUSPEND` | Stops audit log writes; the daemon and workload can remain running. This creates a gap in recorded activity. |
| `SINGLE` or `HALT` | Enters single-user mode or halts the host. These actions affect workload availability. |

The default `admin_space_left_action` and `disk_full_action` are `SUSPEND`.
Choose actions with your security and workload owners, including how alerts
reach an operator before space runs out. No setting alone guarantees lossless
collection.

After editing `/etc/audit/auditd.conf`, ask the daemon to reload it:

```bash
sudo auditctl --signal reload
systemctl is-active auditd
sudo journalctl -u auditd --since '5 minutes ago' --no-pager
```

Inspect the journal for configuration errors. Reloading the daemon
configuration isn't the same as loading rules with `augenrules`.
The packaged unit doesn't support `systemctl reload auditd`.

Ship the records off the VM if they need to survive the VM. Local logs are
under the control of anyone who gains root on the host. `auditd` writes to
`/var/log/audit/audit.log`; Azure Monitor Agent Syslog collection doesn't
collect that file automatically. Collect the audit file with a text log data
collection rule, or explicitly configure an audit dispatcher before you rely
on Syslog collection. For text log collection, see
[Collect text logs with Azure Monitor Agent](/azure/azure-monitor/vm/data-collection-log-text).

### Set journal retention

To bound its size, add a drop-in rather than editing the vendor file:

```bash
sudo mkdir -p /etc/systemd/journald.conf.d
printf '[Journal]\nSystemMaxUse=2G\n' |
  sudo tee /etc/systemd/journald.conf.d/99-retention.conf
sudo systemctl restart systemd-journald
journalctl --disk-usage
```

The limit governs journal retention; it isn't an immediate guarantee that
reported disk use equals the configured value.

## Query security events

Audit records answer "who did what to which object". Journal entries answer
"what did this service report".

### Query audit records

```bash
sudo ausearch -m USER_LOGIN,USER_AUTH -ts today
sudo ausearch -m USER_CMD -ts today
sudo ausearch -f /etc/shadow
sudo ausearch -m KERN_MODULE
```

With the default rules, `sudo` command records are present. A search with no
matching records returns `<no matches>`. The output is similar to the
following partial example. The `cmd` field is hex-encoded; use
`ausearch -i` to decode it:

```output
<no matches>
----
time->Fri Oct 30 10:22:31 2026
type=USER_CMD msg=audit(1793355751.136:600): pid=2377 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 msg='cwd="/tmp" cmd=73736864202D54 exe="/usr/bin/sudo" terminal=? res=success'
...
```

### Query the journal

```bash
sudo journalctl -u sshd --since '24 hours ago' --no-pager
sudo journalctl _COMM=sudo --since '24 hours ago' --no-pager
sudo journalctl -p warning --since '24 hours ago' --no-pager
```

The output is similar to the following partial example:

```output
Oct 30 10:13:02 <host-name> sshd[884]: Server listening on 0.0.0.0 port 22.
Oct 30 10:13:02 <host-name> systemd[1]: Started sshd.service - OpenSSH server daemon.
Oct 30 10:22:31 <host-name> sudo[2377]:     root : ****** ; USER=root ; COMMAND=/usr/sbin/sshd -T
Oct 30 10:22:31 <host-name> sudo[2377]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=0)
...
```

### Interpret empty results

An empty result means no matching record exists in what's retained. It's not
evidence that the activity didn't happen: the rule might not have been loaded
at the time, or the record might have rotated out. For general journal usage,
see [Logging and monitoring](./logging-monitoring.md).

### Export events as JSON

To feed events into a collector, emit JSON:

```bash
journalctl -o json --since '1 hour ago'
```

Each line is one JSON object. The output is similar to the following partial
example:

```output
{"SYSLOG_IDENTIFIER":"kernel","PRIORITY":"5","_TRANSPORT":"kernel","_BOOT_ID":"2c3b30b44f19435ea08448db13a40089","MESSAGE":"Linux version 6.18.50-1.5.azl4.x86_64 ...", ...}
```

## Send logs to Azure services

To keep security events outside the VM and analyze them centrally, you can
connect the VM to Azure monitoring and security services:

- [Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-overview)
  collects syslog and other guest data into a Log Analytics workspace by using
  data collection rules.
- [Microsoft Sentinel](/azure/sentinel/overview) analyzes the collected data
  for threats and supports investigation and automated response.
- [Microsoft Defender for Servers](/azure/defender-for-cloud/defender-for-servers-overview)
  provides threat detection and vulnerability assessment for VMs through
  Microsoft Defender for Cloud.

Before you deploy an agent or extension, check the service's list of supported
Linux distributions.

## Check installed files against the package database

RPM verification compares files on disk with the metadata recorded when the
package was installed:

```bash
rpm -V <package-name>
sudo rpm -Va
```

`sudo rpm -Va` checks every installed package and can take several minutes to
finish.

No output means the checked attributes match the local RPM database. On a new
VM, `sudo rpm -Va` reports a few expected differences, similar to the
following partial example:

```output
.M.......    /
.M.......  g /boot/initramfs-6.18.50-1.5.azl4.x86_64.img
.......T.    /boot/efi/EFI/BOOT/BOOTX64.EFI
```

Each reported line starts with a character per attribute; `.` means unchanged:

| Character | Attribute that differs |
| --------- | ---------------------- |
| `S` | File size |
| `M` | Mode, including permissions and file type |
| `5` | Content digest |
| `D` | Device major or minor number |
| `L` | Symbolic link target |
| `U` | Owner |
| `G` | Group |
| `T` | Modification time |
| `P` | Capabilities |

A `c` after the flags marks a configuration file. Compare changes with approved
configuration. Investigate unexpected digest differences in executables.

The comparison uses the local RPM database, which an attacker with root can
modify. Treat the result as triage, not as tamper-proof attestation, and
compare against a copy of the database you trust when the answer matters.

## Troubleshoot auditd rules and ausearch results

### Rules don't survive a restart

Rules loaded with `auditctl` directly are runtime only. Put them in a file
under `/etc/audit/rules.d/`, then load them with `sudo augenrules --load`.
Review the load output and confirm the effective rules with
`sudo auditctl -l`. A syntax error in any file in that directory can stop the
whole set from loading.

To undo this article's example on an unlocked ruleset, remove its two files
and restore the backup. This restores the original no-audit behavior:

```bash
sudo rm /etc/audit/rules.d/10-base-config.rules
sudo rm /etc/audit/rules.d/70-ssh-config.rules
sudo mv \
  /etc/audit/rules.d/audit.rules.bak \
  /etc/audit/rules.d/audit.rules
sudo augenrules --load
sudo auditctl -l
```

The final command returns `-a never,task`. Don't remove files belonging to
other audit rules.

### The lost counter keeps increasing

The kernel is producing events faster than the daemon consumes them:

```bash
sudo auditctl -s
```

Reduce rule scope, or raise the backlog limit with a `-b` line in your base
rules file, then reload and watch the counter again.

### Change an immutable ruleset

`enabled 2` in `sudo auditctl -s` means the running configuration is locked.
Prepare the intended files under `/etc/audit/rules.d/` **before** restarting.
For a rollback, remove or change the persistent `-e 2` entry as well. Otherwise,
the next boot can reload and lock the same rules again.

During a maintenance window, restart the VM, then check `sudo auditctl -l` and
`sudo auditctl -s`. The enabled `audit-rules.service` loads persistent rules at
boot. If you retain `-e 2`, place it last and expect the new ruleset to lock.

### ausearch finds nothing you expect

Check that a rule covers the object, that the time window includes the event,
that the default `never,task` rule isn't loaded, and that the log hasn't
rotated:

```bash
sudo auditctl -l
sudo ls -l /var/log/audit/
```

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Manage security updates](./manage-common-vulnerabilities-exposures.md)
- [Logging and monitoring](./logging-monitoring.md)
- [Configure SELinux security](./security-enhanced-linux-security.md)
- [Linux audit documentation](https://github.com/linux-audit/audit-documentation/wiki)
