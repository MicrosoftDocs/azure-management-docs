---
title: Overview of Azure Linux Network Security
description: Learn about the network security features and hardened defaults in Azure Linux that help protect workloads from network-based attacks and enforce a secure network posture.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux network security overview

Azure Linux ships with a hardened network configuration out of the box. A host firewall (`firewalld`) denies unsolicited inbound traffic by default, and kernel-level `sysctl` settings disable common network attack vectors. These host-level controls complement Azure Network Security Groups (NSGs) to provide defense-in-depth for your workloads.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Verify network security

Use the following commands to confirm that the default network security posture is active:

```bash
# Verify firewalld is running
sudo systemctl status firewalld

# Check the default firewall zone and allowed services
sudo firewall-cmd --list-all

# Spot-check key sysctl hardening values
sysctl net.ipv4.ip_forward net.ipv4.conf.all.rp_filter net.ipv4.tcp_syncookies
```

If `firewalld` is active and IP forwarding is disabled (`0`), your system is running with the expected defaults.

## Host firewall (firewalld)

Azure Linux enables `firewalld` with an `nftables` backend by default. The default zone enforces a **deny-all inbound, allow-all outbound** posture.

### Default inbound allow rules

Only two inbound services are permitted in the default zone:

| Service | Port / Protocol | Reason |
| ------- | --------------- | ------ |
| SSH | TCP 22 | Remote management access |
| DHCPv6-client | UDP 546 | IPv6 Router Advertisement and address configuration |

All other inbound traffic is denied.

### Manage firewall rules

Use `firewall-cmd` to inspect and modify the running configuration:

```bash
# List all rules in the active zone
sudo firewall-cmd --list-all

# Add a port (example: HTTPS)
sudo firewall-cmd --add-port=443/tcp

# Make the change persistent across reboots
sudo firewall-cmd --runtime-to-permanent

# Remove a port
sudo firewall-cmd --remove-port=443/tcp
sudo firewall-cmd --runtime-to-permanent

# Reload the firewall to apply persistent changes
sudo firewall-cmd --reload
```

> [!NOTE]
> `firewalld` provides host-level defense-in-depth. Azure NSGs remain the primary network control plane for Azure virtual machines (VMs). Always configure both layers for production workloads.

## Network hardening sysctl defaults

Azure Linux applies the following kernel network parameters by default to reduce the attack surface at the operating system (OS) level:

| Setting | Default value | Purpose |
| ------- | ------------- | ------- |
| `net.ipv4.ip_forward` | `0` | Disable IPv4 packet forwarding between interfaces |
| `net.ipv6.conf.all.forwarding` | `0` | Disable IPv6 packet forwarding between interfaces |
| `net.ipv4.conf.all.rp_filter` | `2` | Strict-mode Reverse Path Filtering - drop packets arriving on unexpected interfaces |
| `net.ipv4.conf.all.accept_redirects` | `0` | Ignore ICMP redirect messages (IPv4) |
| `net.ipv6.conf.all.accept_redirects` | `0` | Ignore ICMP redirect messages (IPv6) |
| `net.ipv4.conf.all.accept_source_route` | `0` | Reject source-routed packets |
| `net.ipv4.conf.all.send_redirects` | `0` | Don't send ICMP redirects |
| `net.ipv4.conf.all.log_martians` | `1` | Log packets with impossible source addresses |
| `net.ipv4.tcp_syncookies` | `1` | Enable TCP SYN cookies to mitigate SYN flood attacks |

### Verify sysctl settings

Check individual values with the `sysctl` command:

```bash
# Verify all network hardening defaults at once
sysctl \
  net.ipv4.ip_forward \
  net.ipv6.conf.all.forwarding \
  net.ipv4.conf.all.rp_filter \
  net.ipv4.conf.all.accept_redirects \
  net.ipv6.conf.all.accept_redirects \
  net.ipv4.conf.all.accept_source_route \
  net.ipv4.conf.all.send_redirects \
  net.ipv4.conf.all.log_martians \
  net.ipv4.tcp_syncookies
```

Expected output shows each value matching the defaults in the [Network hardening sysctl defaults table](#network-hardening-sysctl-defaults). If any value differs, check whether a custom configuration in `/etc/sysctl.d/` overrides the default.

## IPv6 security

IPv6 is enabled by default on Azure Linux, consistent with Azure networking defaults.

Azure Linux applies the following IPv6-specific hardening settings to reduce exposure:

| Setting | Default value | Purpose |
| ------- | ------------- | ------- |
| `net.ipv6.conf.all.accept_redirects` | `0` | Ignore ICMPv6 redirect messages |
| `net.ipv6.conf.all.forwarding` | `0` | Disable IPv6 forwarding |

### Verify IPv6 hardening

Check the current values of the IPv6 hardening settings with `sysctl`:

```bash
sysctl net.ipv6.conf.all.accept_redirects net.ipv6.conf.all.forwarding
```

Both values should be `0`. If your workload doesn't use IPv6, you can optionally disable it entirely. While it's possible to disable IPv6 at the OS level, it's not required or recommended on Azure because Azure infrastructure relies on IPv6 for certain platform functions.

## Azure NSGs and firewalld

Azure Network Security Groups (NSGs) and `firewalld` serve complementary security roles:

- **Azure NSG** is the primary network control plane. NSG rules are evaluated at the Azure networking layer before traffic reaches the VM's network interface. Use NSGs to define broad allow/deny policies for your virtual network.
- **firewalld** provides a host-level firewall inside the VM. It acts as a second enforcement point after traffic passes through the NSG.

**Best practice**: Configure both layers for defense-in-depth. An NSG rule that allows traffic doesn't bypass `firewalld`, and a `firewalld` rule that opens a port has no effect if the NSG blocks the traffic first.

:::image type="content" source="./media/network-security-overview.png" alt-text="Screenshot of a diagram illustrating Azure Network Security Groups (NSGs) and firewalld in Azure Linux." lightbox="./media/network-security-overview.png":::

## Related content

- [Kernel hardening in Azure Linux](./kernel-hardening.md)
- [Logging and auditing in Azure Linux](./logging-auditing.md)
- [Overview of Azure Linux security](./security-overview.md)
