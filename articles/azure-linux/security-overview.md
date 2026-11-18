---
title: Azure Linux security overview
description: Understand the Azure Linux security model, the split between Azure platform controls and guest operating system controls, and where to find each security task.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Azure Linux security overview

Azure Linux 4 ships with Security-Enhanced Linux (SELinux) enforcing its
targeted policy, and with SSH, kernel, and network hardening defaults applied.
Azure network security groups (NSGs) filter inbound traffic by default; a host
firewall is available when your workload needs one.

These guides cover general-purpose Azure Linux 4 virtual machines (VMs).
Azure Container Linux (ACL) for Azure Kubernetes Service (AKS) is covered in
the [Azure Container Linux documentation](./azure-container-linux-overview.md).

## Default security configuration

These defaults apply to the general-purpose Azure Linux 4 VM image.
Provisioning and image customization can override them.

| Protection | Default configuration | Guide |
| ---------- | --------------------- | ----- |
| Mandatory access control | SELinux runs in enforcing mode with the targeted policy. | [Configure SELinux security](./security-enhanced-linux-security.md) |
| Kernel and eBPF | Address space layout randomization is enabled, kernel log and pointer access are restricted, and unprivileged eBPF is disabled. | [Inspect kernel hardening](./kernel-hardening.md) and [Use eBPF](./extended-berkeley-packet-filter.md) |
| Host network | No host firewall is installed. NSGs filter inbound traffic. IP forwarding, ICMP redirects, and source routing are disabled. | [Secure the host network](./network-security.md) |
| Interactive access | On key-based deployments, SSH password authentication is disabled. Direct root sign-in is denied. SSH uses hardened session limits and SHA-2 message authentication codes. | [Manage identity and access](./identity-and-access.md) |
| Guest disk encryption | The VM root isn't encrypted with Linux Unified Key Setup (LUKS). Guest-managed data-volume encryption requires configuration. Azure platform encryption is separate. | [Protect disks and data](./disk-and-data-protection.md) |
| Cryptography | `DEFAULT` sets cryptographic defaults for applications that consume the system policy. Application overrides can change them. | [Use system cryptographic libraries](./system-crypto.md) |
| Auditing and logs | `auditd` is enabled, but its default rule disables syscall and file-watch auditing. That coverage requires additional rules. The journal is persistent. | [Configure security logging and auditing](./logging-and-auditing.md) |
| Packages | DNF verifies package signatures. Repository metadata signature checking isn't enabled. | [Manage security updates](./manage-common-vulnerabilities-exposures.md) |

## Security defaults reference

Use this reference to find where a default is set and how to check it on a
running VM. Vendor files under `/usr/lib` and package-owned files can be
replaced by updates. Put local changes in the override location that each guide
describes.

The defaults and checks in this table were verified on a newly deployed VM
from the general-purpose Azure Linux 4 image, before any configuration
changes. Provisioning options, extensions, and configuration management can
change the results on your VM.

| Setting | Default | Set in | Check |
| ------- | ------- | ------ | ----- |
| SELinux mode and policy | `enforcing`, `targeted` | `/etc/selinux/config` | `sestatus` |
| Host firewall | Not installed. The `nftables` and `iptables-legacy` tools are installed, but no inbound filtering rules are loaded. | Not applicable | `rpm -q firewalld`, `sudo nft list ruleset`, and `sudo iptables -S` |
| Root SSH sign-in | `PermitRootLogin no` and `DenyUsers root` | `/etc/ssh/sshd_config.d/50-permit-root-login.conf` and `30-azurelinux-cis.conf` | `sudo sshd -T`, then find `permitrootlogin` and `denyusers` |
| SSH password authentication | `PasswordAuthentication no` on key-based deployments | `/etc/ssh/sshd_config.d/50-cloud-init.conf`, written at provisioning | `sudo sshd -T`, then find `passwordauthentication` |
| SSH session limits | `MaxAuthTries 4`, `LoginGraceTime 60`, `MaxStartups 10:30:60`, `ClientAliveInterval 180` | `/etc/ssh/sshd_config.d/30-azurelinux-cis.conf` and `50-client-alive-interval.conf` | `sudo sshd -T` |
| SSH algorithms | System cryptographic policy, with message authentication codes limited to SHA-2 | `/etc/ssh/sshd_config.d/40-azurelinux-crypto-policies.conf` and `30-azurelinux-cis.conf` | `sudo sshd -T`, then find `kexalgorithms`, `ciphers`, and `macs` |
| SSH pre-authentication banner | `/etc/issue.net` | `/etc/ssh/sshd_config.d/30-azurelinux-cis.conf` | `cat /etc/issue.net` |
| `sudo` session handling | Commands run in a pseudo-terminal and are logged to `/var/log/sudo.log` | `/etc/sudoers.d/10-azurelinux-cis` | `sudo cat /etc/sudoers.d/10-azurelinux-cis` |
| Password hashing | `YESCRYPT` | `/etc/login.defs` | `grep ENCRYPT_METHOD /etc/login.defs` |
| Kernel parameters | Full address space layout randomization, restricted kernel pointers and log, restricted `ptrace`, unprivileged eBPF disabled | `/usr/lib/sysctl.d/70-azurelinux-hardening.conf` and the kernel build | `sysctl kernel.randomize_va_space kernel.kptr_restrict kernel.yama.ptrace_scope kernel.unprivileged_bpf_disabled` |
| Network parameters | Forwarding, redirects, source routing, and IPv6 router advertisements disabled; reverse path filtering, SYN cookies, and martian logging enabled | `/usr/lib/sysctl.d/70-azurelinux-hardening.conf` | `sysctl net.ipv4.ip_forward net.ipv4.conf.all.rp_filter net.ipv6.conf.all.accept_ra` |
| Temporary file systems | `/tmp` and `/dev/shm` are `tmpfs` mounted `nosuid,nodev` | systemd mount units | `findmnt /tmp` and `findmnt /dev/shm` |
| Cryptographic policy | `DEFAULT`, including hybrid post-quantum key exchange | `/etc/crypto-policies/config` | `update-crypto-policies --show` |
| Audit daemon | Enabled; the default rule disables syscall auditing | `/etc/audit/rules.d/audit.rules` | `sudo auditctl -l` |
| Journal | Persistent | `/var/log/journal` | `journalctl --disk-usage` |
| Package signatures | `gpgcheck=1`, `repo_gpgcheck=0` | `/etc/yum.repos.d/*.repo` | `grep -h gpgcheck /etc/yum.repos.d/*.repo` |

For CIS Benchmark and Federal Information Processing Standards (FIPS) status,
see the [Azure Linux compliance overview](./compliance-overview.md).

## Configure protection for your workload

Workload configuration determines the required access, audit coverage, and
application controls. Differences from image defaults can reflect provisioning
or configuration management rather than an error.

- Allow only the network access and account privileges the workload needs.
- Configure audit rules and retention for the required security events.
- Confirm that application processes and cryptographic settings use the
  intended protections.
- Apply updates and confirm that affected workloads run the updated code.

Azure network security groups (NSGs), managed disk encryption, and Azure role
assignments remain separate from guest configuration. Review both layers. For
example, allowing a port in the guest doesn't override an NSG, and a managed
identity grants access to Azure resources rather than an operating system
sign-in.

## Change security defaults deliberately

Changes to security defaults require a documented workload requirement or
approved baseline. Weaker settings can expose the workload; stricter settings
can affect compatibility or performance.

Production rollout requires representative nonproduction testing and a rollback
plan. See
[Apply security baselines safely](./compliance-overview.md#apply-security-baselines-safely)
for approval and recovery requirements.

## Related content

- [Azure Linux compliance overview](./compliance-overview.md)
- [Azure Linux architecture](./architecture.md)
- [Build hardened software](./build-hardening.md)
- [Secure language package managers](./language-package-managers.md)
- [Update an Azure Linux VM](./update-vm-azure-linux.md)
- [Package repositories](./package-repositories.md)
- [Report issues and request features](./report-issues-request-features.md)
