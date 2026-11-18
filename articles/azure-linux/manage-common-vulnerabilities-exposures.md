---
title: Manage security updates on Azure Linux
description: Learn how Azure Linux handles CVEs and delivers fixes to VMs, AKS, and Azure Container Linux, and find, apply, and verify signed package updates on Azure Linux 4 VMs.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Manage security updates on Azure Linux

This article explains how Azure Linux identifies Common Vulnerabilities and
Exposures (CVEs), how fixes reach each deployment option, and where advisories
are published. It then shows how to find, apply, and verify package updates on
Azure Linux 4 VMs with DNF.

## How Azure Linux handles CVEs

Microsoft is responsible for the full Azure Linux and Azure Container Linux
(ACL) stack, from the Linux kernel to the CVE infrastructure, support, and
end-to-end validation. You don't need to track and patch vulnerabilities from a
third-party distribution.

The Azure Linux team monitors the packages it ships for security
vulnerabilities through a process coordinated with the
[Microsoft Security Response Center (MSRC)](https://www.microsoft.com/msrc).
Automated scans against the
[National Vulnerability Database (NVD)](https://nvd.nist.gov/) run twice a day,
and they're one input to that process, not the whole of it. When a
vulnerability is confirmed, the team works with MSRC to evaluate its impact,
prioritize and ship a fix, publish advisories, and contribute fixes upstream.

Azure Linux ships security fixes monthly. Fixes for critical and
high-severity CVEs can ship before the next monthly update. For more
information, see
[Azure Linux release cadence and lifecycle](./release-cadence-lifecycle.md).

### CVE delivery by deployment option

CVE fixes reach your systems differently depending on how you run Azure Linux:

| Deployment option | How CVE fixes are delivered | Update guidance |
| ----------------- | --------------------------- | --------------- |
| General-purpose Azure Linux 4 VM, scale set, or custom image | Package updates that you apply with `dnf update`. | This article and [Update an Azure Linux VM](./update-vm-azure-linux.md) |
| Azure Linux Container Host for AKS | Package updates bundled into monthly node image releases. High and critical CVE fixes might be released out-of-band as package updates before the next node image. Medium and low CVE fixes are rolled into the next regular node image. | [Upgrade Azure Linux Container Host nodes](./tutorial-upgrade-azure-linux-nodes.md) |
| Azure Container Linux (ACL) for AKS | ACL is an immutable, image-based OS, so packages aren't updated in place. Fixes ship in weekly AKS node images. The `SecurityPatch` node OS upgrade channel isn't supported on ACL; use the `NodeImage` channel. | [ACL for AKS versioning and updates](./azure-container-linux-overview.md#acl-for-aks-versioning) |

The package procedures in this article apply only to general-purpose Azure
Linux 4 VMs. They aren't servicing instructions for ACL or AKS nodes.

### Published advisories

MSRC publishes Azure Linux security advisories in two forms:

- **Vulnerability Exploitability eXchange (VEX) documents** in the Common
  Security Advisory Framework (CSAF) format, at the
  [MSRC CSAF VEX feed](https://msrc.microsoft.com/csaf/vex/). For each CVE,
  the VEX document lists Azure Linux 4.0 package versions as fixed, known
  affected, or under investigation. Use the feed to ingest advisories into
  vulnerability management tools.
- **The [Security Update Guide (SUG)](https://msrc.microsoft.com/update-guide/)**,
  where you can look up a CVE or filter by the **Azure Linux 4.0** platform.
  The same data is available programmatically through the
  [Security Update Guide API](https://github.com/microsoft/MSRC-Microsoft-Security-Updates-API).

## Default package signature checking for DNF repositories

The image's repositories enable package signature checking (`gpgcheck=1`).
Repository metadata signature checking isn't enabled (`repo_gpgcheck=0`).
Keep package signature checking enabled when applying updates.

## Prerequisites

- An Azure Linux virtual machine (VM) that can reach its package repositories.
- A user account with `sudo` privileges.
- A maintenance window and a tested way to roll back, which for image-based
  deployments means redeploying a known-good image.

## Check before updating

Inspect the available updates and verify package trust before accepting a
transaction.

### Look up a CVE in the Security Update Guide

The [Microsoft Security Update Guide](https://msrc.microsoft.com/update-guide/vulnerability)
lists the Common Vulnerabilities and Exposures (CVE) identifiers that affect
Azure Linux packages and the package versions that fix them.

To check a specific CVE:

1. Open `https://msrc.microsoft.com/update-guide/vulnerability/<CVE-ID>`,
   replacing `<CVE-ID>` with the identifier, for example `CVE-2026-97229`.
1. In the **Security Updates** table, find the rows where **Platform** is
   **Azure Linux 4.0**.
1. Read the affected package from the **Product** column, for example
   `azl4 ruby 3.4.10-36.azl4`, and the fixed package version from the
   **Build Number** column, for example `0:3.4.12-38.azl4`. The `0:` prefix is
   the package epoch.
1. Compare the fixed version with the version installed on your VM:

   ```bash
   rpm -q <package-name>
   ```

To browse all CVEs for Azure Linux 4, open the
[Security Update Guide](https://msrc.microsoft.com/update-guide/), select
**Filters**, and then select **Azure Linux 4.0** under **Platform**.

### Check what updates are available

```bash
sudo dnf makecache
dnf check-update
echo "Exit code: $?"
```

When updates are available, the command lists them and returns:

```output
Exit code: 100
```

| Exit code | Meaning |
| --------- | ------- |
| `0` | No updates are available. |
| `100` | Updates are available. This isn't an error. |
| `1` | The command failed, for example because a repository is unreachable. |

Handle `100` explicitly in automation, including scripts that exit on a
nonzero status.

### Inspect a package before you update it

Review a package's version and changelog before you apply an update.

#### Compare installed and available versions

Compare the installed and available versions:

```bash
dnf info <package-name>
```

For example, `dnf info openssl` on a new VM shows the installed package. When
an update is available, the output also lists it under `Available packages`.
The output is similar to the following partial example:

```output
Installed packages
Name            : openssl
Epoch           : 1
Version         : 3.5.4
Release         : 14.azl4
Architecture    : x86_64
Installed size  : 1.8 MiB
Source          : openssl-3.5.4-14.azl4.src.rpm
...
Vendor          : Microsoft Corporation
```

#### Read package changelogs

The `dnf changelog` command isn't installed on the image. Install it once
before you read changelogs from the repository:

```bash
sudo dnf install 'dnf5-command(changelog)'
```

Without it, DNF reports `Unknown argument "changelog" for command "dnf5"`.
Then show the changelog entries that an available update adds beyond the
installed version:

```bash
dnf changelog --upgrades <package-name>
```

When no update is available for the package, the command lists no entries:

```output
Repositories loaded.
Listing only new changelogs since installed version of the package
```

To show the full changelog of the latest version in the repository, omit
`--upgrades`. Use `--count=<n>` to limit the number of entries.

`dnf changelog` reads changelogs from the repository, so it also works for
packages that aren't installed. `rpm -q --changelog` reads only the installed
package:

```bash
rpm -q --changelog <package-name> | head -n 50
```

| Command | Source | Works for packages that aren't installed |
| ------- | ------ | ---------------------------------------- |
| `dnf changelog <package-name>` | Repository metadata | Yes |
| `rpm -q --changelog <package-name>` | Installed package | No |

Changelogs can reference CVE identifiers but aren't a complete advisory feed.
Don't close a finding on a changelog entry alone.

### Verify package signatures

Confirm that DNF checks package signatures, inspect the keys it trusts, and
verify a package by hand when you need to.

#### Review repository signature settings

Inspect repository signature settings:

```bash
grep -hE '^(gpgcheck|repo_gpgcheck)=' /etc/yum.repos.d/*.repo | sort -u
```

Expected result for the default repository configuration:

```output
gpgcheck=1
repo_gpgcheck=0
```

What those two settings mean:

- `gpgcheck=1` means DNF verifies each package's signature against a key in the
  RPM database before installing it.
- `repo_gpgcheck=0` means repository metadata isn't signature-verified, so the
  integrity of the package listing rests on the transport rather than on a
  signature. The packages themselves are still checked.

#### List and inspect trusted keys

List the keys currently trusted by the RPM database:

```bash
rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
```

On a new VM, the command returns `package gpg-pubkey is not installed`. The
image ships the repository key as a file, and DNF imports it into the RPM
database during the first `dnf install` or `dnf update` transaction. After
that, the result is:

```output
gpg-pubkey-2bc94fff7015a5f28f1537ad0cd9fed33135ce90-5e6fda74	Mariner RPM Release Signing <marinerrpmprod@microsoft.com> public key
```

The version field is the full key fingerprint. Mariner is the earlier name of
Azure Linux; the same key signs Azure Linux 4 packages for both x86-64 and
Arm64. To read the fingerprint from the key file without importing it:

```bash
gpg --show-keys --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-azurelinux-4.0-x86_64
```

```output
pub   rsa2048 2020-03-16 [SC]
      2BC9 4FFF 7015 A5F2 8F15  37AD 0CD9 FED3 3135 CE90
uid                      Mariner RPM Release Signing <marinerrpmprod@microsoft.com>
```

#### Verify a package signature manually

To check a package yourself:

```bash
dnf download <package-name>
rpm -Kv <package-file>.rpm
```

A `NOKEY` result means the required public key isn't in the RPM database.
Confirm the key's trusted source before importing it. The image provides its
repository key under:

```bash
ls -1 /etc/pki/rpm-gpg/
```

Example output from an x86-64 image:

```output
RPM-GPG-KEY-4.0-azurelinux
RPM-GPG-KEY-azurelinux-4.0-aarch64
RPM-GPG-KEY-azurelinux-4.0-primary
RPM-GPG-KEY-azurelinux-4.0-x86_64
```

The repository files reference the key for the release and architecture, such
as `RPM-GPG-KEY-azurelinux-4.0-x86_64`. The other names are links to the same
key file.

```bash
sudo rpmkeys --import /etc/pki/rpm-gpg/<key-file>
rpm -Kv <package-file>.rpm
```

Repeat the signature check and confirm it succeeds. A valid signature
establishes trust only relative to the imported key; it doesn't establish that
the package has no vulnerabilities.

## Understand the update model

Before you plan updates, know what the image does and doesn't do on its own:

| Behavior | Default on the general-purpose image |
| -------- | ------------------------------------ |
| Automatic package updates | Off. Only `dnf-makecache.timer` runs, which refreshes repository metadata but doesn't install updates. To schedule updates, you can install the `dnf5-plugin-automatic` package from the Azure Linux repository and configure it. |
| Advisory metadata for DNF | Not published. `dnf updateinfo` and `dnf update --security` find no advisories. Use the [published advisories](#published-advisories) to decide which updates fix a CVE. |
| Kernel live patching | Not available. Kernel updates take effect after a restart. |
| Service restarts after a library update | Not automatic. `needrestart` isn't installed. Restart affected services yourself, as described in [Apply updates](#apply-updates). |

To check whether automatic updates are configured on a VM:

```bash
systemctl list-timers --all --no-pager | grep -iE 'dnf|automatic'
rpm -q dnf5-plugin-automatic
```

On the default image, only `dnf-makecache.timer` is listed, and the second
command returns `package dnf5-plugin-automatic is not installed`.

For Azure-managed patch orchestration, see
[Automatic VM guest patching](/azure/virtual-machines/automatic-vm-guest-patching)
and check its list of supported operating system images.

## Apply updates

1. Review the proposed transaction, including dependency changes, before
   accepting it:

   ```bash
   sudo dnf update
   ```

   To update one package and its required dependencies instead:

   ```bash
   sudo dnf update <package-name>
   ```

1. Restart what's still running old code. Updating a shared library on disk
   doesn't affect processes that already mapped the previous version. To find
   processes that still map deleted files, such as a replaced library, list
   their memory maps:

   ```bash
   sudo grep -l '(deleted)' /proc/[0-9]*/maps
   ```

   Each result names a process ID in its path. Identify the process with
   `ps -p <process-id> -o pid,comm`, then restart its service:

   ```bash
   sudo systemctl restart <service-name>
   ```

1. If the transaction included the kernel, restart the VM:

   ```bash
   sudo reboot
   ```

## Verify installed package and kernel versions after an update

```bash
rpm -q <package-name>
uname -r
rpm -q --qf '%{VERSION}-%{RELEASE}.%{ARCH}\n' kernel
```

Confirm the intended package version is installed and the intended kernel is
running. More than one kernel package can be installed; compare the running
version with the version you intended to boot.

Check for remaining updates at the same scope as the transaction:

```bash
dnf check-update <package-name>
echo "Exit code: $?"
```

For a full-system update, omit the package name. Exit `0` means no updates are
offered at that scope; it isn't proof that no vulnerabilities remain. Unrelated
pending packages don't invalidate a successful single-package update.

Check files with `rpm -V <package-name>`, investigate unexpected differences,
and exercise the application after restarting it. Approved configuration-file
changes can appear in RPM verification output. See
[Check installed files](./logging-and-auditing.md#check-installed-files-against-the-package-database).

Record the installed version, running kernel, restart and application-check
results, and timestamp with the remediation evidence.

## Coordinate with vulnerability scanners

Azure Linux and ACL support common vulnerability scanning tools. For supported
scanners, see [Azure Linux partner solutions](./ecosystem-support.md).

When VEX advisories are available, scanners that consume VEX can report whether
an installed package is actually exposed to a CVE, which reduces false
positives.

## Report a security issue

Report suspected vulnerabilities to the
[Microsoft Security Response Center](https://msrc.microsoft.com/create-report).
Don't disclose a suspected vulnerability through a public GitHub issue.

## Troubleshoot DNF repository, signature, and kernel update issues

### dnf can't reach a repository

Package repository access requires outbound HTTPS connectivity. An NSG rule,
proxy, or host firewall rule can block it. Show the repository configuration
and status:

```bash
dnf repo info azurelinux-base
```

The output is similar to the following partial example:

```output
Repo ID              : azurelinux-base
Name                 : Azure Linux 4.0 - x86_64 - Production - Base
Status               : enabled
Config file          : /etc/yum.repos.d/azurelinux.repo
URLs                 :
  Base URL           : https://packages.microsoft.com/azurelinux/4/prod/base/x86_64
OpenPGP              :
  Keys               : file:///etc/pki/rpm-gpg/RPM-GPG-KEY-azurelinux-4.0-x86_64
  Verify repodata    : false
  Verify packages    : true
...
```

To list every configured repository, run `dnf repolist --all`. Confirm that
the VM can reach the base URL over HTTPS.

If the VM uses a proxy, configure it in `/etc/dnf/dnf.conf` and retry.

### Repository metadata looks stale

Clear the cache and rebuild it:

```bash
sudo dnf clean metadata
sudo dnf makecache
```

### An install fails with a NOKEY error

Confirm the expected repository key and its trusted source before importing
it. Don't bypass verification with `--nogpgcheck`.

### The kernel version didn't change after updating

Kernel packages install alongside the running kernel and take effect at the
next restart:

```bash
rpm -q kernel
uname -r
sudo reboot
```

If the versions still differ after a restart, check which boot entry is
selected by default.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Update an Azure Linux VM](./update-vm-azure-linux.md)
- [Package repositories](./package-repositories.md)
- [Manage packages with DNF](./manage-packages.md)
- [Automatic VM guest patching](/azure/virtual-machines/automatic-vm-guest-patching)
