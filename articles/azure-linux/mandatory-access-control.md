---
title: Configure Mandatory Access Control in Azure Linux with SELinux and Landlock
description: Learn how Azure Linux uses mandatory access control (MAC) with SELinux and Landlock to enforce security policies beyond standard Linux permissions.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Configure mandatory access control (MAC) in Azure Linux with SELinux and Landlock

Azure Linux uses mandatory access control (MAC) to enforce security policies beyond standard Linux permissions. [SELinux](https://github.com/selinuxproject) is the primary MAC framework, and [Landlock](https://docs.kernel.org/userspace-api/landlock.html) provides an extra application-level sandboxing layer.

This article explains how Azure Linux leverages SELinux and Landlock to implement mandatory access control, how to check the current SELinux status, and how to manage SELinux modes for day-to-day operations.

> [!NOTE]
> Azure Linux 4.0 and Azure Container Linux (ACL) use SELinux exclusively. AppArmor isn't included. AKS workloads that require AppArmor should continue using Azure Linux 3.0 through its committed support lifecycle.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Why use mandatory access control

Mandatory access control policies help you achieve:

- **Least privilege**: Restrict processes and users to only the resources they need.
- **Integrity**: Prevent unauthorized modification of critical files and system components.
- **Isolation**: Contain compromised processes and limit lateral movement.
- **Confidentiality**: Control read access to sensitive data.
- **Role separation**: Assign distinct security contexts to different system roles.

## Check SELinux status

### Quick mode check

Use `getenforce` for a quick mode check:

```bash
getenforce
```

Example output:

```text
Enforcing
```

### Full status check

Use `sestatus` for the full runtime configuration including the loaded policy and kernel policy version:

```bash
sestatus
```

Example output:

```text
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      35
```

## SELinux on Azure Linux

Azure Linux ships with the [SELinux](https://github.com/selinuxproject) **targeted** policy. All packages and files are pre-labeled at build time to ensure correct security contexts from first boot. Advanced kernel features, including Multi-Level Security (MLS) support, are enabled and supported up to policy version 35.

### SELinux filesystem locations

The following two paths are critical for SELinux operation on Azure Linux:

- `/sys/fs/selinux`: The kernel-exposed sysfs interface that provides real-time access to SELinux status, current enforcement mode, and the loaded policy.
- `/etc/selinux`: The userspace configuration directory, including `/etc/selinux/config`, which controls the SELinux mode applied at boot.

### SELinux modes

The following table summarizes the three SELinux modes available on Azure Linux and their behavior:

| Mode | Behavior |
| ---- | -------- |
| **Permissive** | Logs policy violations but doesn't enforce them. Use this mode to identify and resolve issues before switching to enforcing. |
| **Enforcing** | Enforces the targeted policy and denies unauthorized access. |
| **Disabled** | Turns SELinux off entirely. **Not recommended** — disabling SELinux removes mandatory access control protections from the system. |

You can switch SELinux modes either temporarily (until the next reboot) or permanently. Permissive mode is useful when you need to diagnose denials without blocking workloads — it logs every policy violation but doesn't enforce them.

#### Temporarily change SELinux mode

Use `setenforce` to switch the running mode without modifying the SELinux configuration files. The change reverts on reboot.

```bash
sudo setenforce 0   # switch to permissive
sudo setenforce 1   # switch back to enforcing
```

#### Permanently change SELinux mode

To make a mode change persist across reboots, edit `/etc/selinux/config` and update the `SELINUX=` line.

```text
SELINUX=enforcing
```

Valid values are `enforcing`, `permissive`, and `disabled`. After saving the file, reboot the system for the change to take effect.

### Enable SELinux during image build

In your image configuration JSON, set the SELinux kernel command line option. For example:

```json
{
  "SystemConfigs": [
    {
      "Name": "My Image",
      "PackageLists": [
        "packagelists/selinux.json"
      ],
      "KernelCommandLine": {
        "SELinux": "permissive"
      }
    }
  ]
}
```

- **Valid values for `SELinux`**:
  - `permissive`: Boot in permissive mode (controlled by `/etc/selinux/config`).
  - `enforcing`: Boot in enforcing mode (controlled by `/etc/selinux/config`).
  - `force_enforcing`: Boot in enforcing mode and add `enforcing=1` on the kernel command line.
- **Minimum required package**:
  - `selinux-policy-targeted`: Already preinstalled on Azure Linux 4.0.
- **Optional packages**:
  - `policycoreutils-python-utils`
  - `setools-console`

### Enable SELinux on a running system

1. Confirm the targeted policy is installed (it ships preinstalled on Azure Linux 4.0):

    ```bash
    rpm -q selinux-policy-targeted
    ```

   If the package is missing, install it:

   ```bash
   sudo dnf install -y selinux-policy-targeted
   ```

1. Update the kernel command line in `/boot/grub2/grub.cfg`. Add `selinux=1 security=selinux` to the `linux` line.

   ```text
   menuentry "Azure Linux" {
     linux $bootprefix/$azurelinux rd.auto=1 root=$rootdevice $azurelinux_cmdline lockdown=integrity sysctl.kernel.unprivileged_bpf_disabled=1 $systemd_cmdline selinux=1 security=selinux
     if [ -f $bootprefix/$azurelinux_initrd ]; then
       initrd $bootprefix/$azurelinux_initrd
     fi
   }
   ```

1. Reboot the system. The first reboot performs filesystem labeling and might take time.

    If SELinux doesn't relabel pre-existing files, create an empty `/.autorelabel` file and reboot:

    ```bash
    sudo touch /.autorelabel && sudo reboot
    ```

## Landlock Linux Security Module (LSM)

[Landlock](https://docs.kernel.org/userspace-api/landlock.html) is a Linux Security Module that allows applications to restrict their own access to the filesystem, network, and other resources. Unlike SELinux, which enforces system-wide policies, Landlock enables application-level sandboxing without requiring administrator privileges.

### Landlock on Azure Linux (general-purpose)

Landlock support is compiled into the kernel and available in **opt-in** mode. Applications can leverage Landlock APIs to self-restrict, but no system-wide Landlock policy is enforced.

### Verify Landlock availability

Verify Landlock is available by checking the list of active Linux Security Modules (LSMs) on the system:

```bash
# Check if the Landlock LSM is active
cat /sys/kernel/security/lsm
```

The output should include `landlock` in the list of active LSMs.

### Landlock on Azure Container Linux (ACL)

Azure Container Linux (ACL) enables Landlock in **enforcing** mode. All shipped applications are required to leverage Landlock policies, providing defense-in-depth against container escape attacks layered on top of SELinux enforcement.

## Troubleshoot SELinux issues

The following sections describe how to diagnose and resolve common SELinux issues on Azure Linux systems.

### SELinux audit logs

SELinux records detailed information about security events, policy enforcement, and access denials in the system audit log. These logs are the primary source for diagnosing SELinux-related issues, application failures caused by missing labels, and unexpected policy denials.

Audit log entries cover:

- **Access denial tracking**: Recorded as Access Vector Cache (AVC) denials with the user, process, target object, and security context involved.
- **Policy enforcement visibility**: Shows which SELinux rules are being applied at runtime.
- **Detailed context information**: Each entry includes the source and target SELinux contexts so you can map a denial back to a policy rule.
- **Timestamps**: Every entry is timestamped to support correlation with application logs and incident timelines.

Audit logs live under the following directory:

```text
/var/log/audit/
```

Use the tools in the following sections to interpret these entries and generate suggested policy fixes.

### Use audit2allow

`audit2allow` helps diagnose and resolve SELinux denials by analyzing audit logs and suggesting policy changes. Install it from the `policycoreutils-python-utils` package:

```bash
sudo dnf install -y policycoreutils-python-utils
```

If `auditd` is running, use `audit2allow` to analyze all recorded denials:

```bash
audit2allow -a -w
```

If `auditd` isn't running, analyze recent denials from the journal:

```bash
journalctl -b | audit2allow -w
```

The output highlights the likely cause and suggests a remediation, such as enabling a SELinux boolean or creating a policy module.

## Related content

- [Use extended Berkeley Packet Filter (eBPF) on Azure Linux](./use-ebpf.md)
- [Kernel hardening in Azure Linux](./kernel-hardening.md)
