---
title: Secure the host network in Azure Linux
description: Understand Azure Linux network filtering, add a host firewall, audit hardened network kernel parameters, and override Azure-generated interface configuration safely.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Secure the host network in Azure Linux

Azure network security groups (NSGs) filter traffic to an Azure Linux 4
virtual machine (VM) at the subnet or network interface. Kernel parameters
harden how the guest handles network traffic. A host firewall inside the guest
is optional, and you add it when your workload needs one.

## Default network filtering and hardening settings

The general-purpose image doesn't include an active host firewall. `firewalld`
isn't installed. The `nftables` and `iptables-legacy` tools are installed, but
the `nftables` service isn't enabled and no inbound filtering rules are loaded.
Inbound filtering depends on the NSG rules for the VM. If your workload needs a
host firewall, install `firewalld` from the Azure Linux repository or manage
`nftables` rules directly.

IP forwarding, ICMP redirects, and source routing are disabled by the network
hardening defaults. At first boot, cloud-init generates the primary interface
configuration for `systemd-networkd` from Azure metadata.

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.
- Access to the NSG rules for the VM's subnet and network interface.
- A running application to test connectivity after adding a firewall rule.

## Verify host firewall, listeners, and network kernel parameters

Connection handling depends on the applicable Azure rules, the host firewall
if you installed one, and the application's listener.

### How the filtering layers interact

- **Azure NSGs** filter traffic in the virtual network before it reaches the
  VM's network interface. Inspect their rules through Azure; guest firewall
  commands don't show or change them.
- **A host firewall**, such as `firewalld`, filters traffic that already
  reached the VM. `firewalld` is a zone-based front end for `nftables`. Named
  services such as `https` define traffic rules; they don't authenticate or
  identify the application process.

For a failed connection, check the listener, the NSG rules, and the host
firewall if one is running. Routing and application configuration also affect
the result.

### Check for a host firewall

Check whether `firewalld` is installed and whether any `nftables` rules are
loaded:

```bash
rpm -q firewalld
systemctl is-enabled nftables
sudo nft list ruleset
```

Expected result for the default configuration:

```output
package firewalld is not installed
disabled
```

An empty `nft list ruleset` result means that no `nftables` rules are loaded.

The Azure guest agent adds `iptables` rules to the `security` table that limit
outbound connections to the Azure host endpoint, `168.63.129.16`. Those rules
protect the agent's communication with the platform. They don't filter inbound
traffic to your workload. To see them, run `sudo iptables -t security -S`.

### See what the host exposes

Firewall rules matter only in relation to what's listening behind them. List
listening sockets and the processes that own them:

```bash
sudo ss -tulnp
```

Read the local address column:

- `127.0.0.1` or `::1` means the service accepts local connections only, and no
  ordinary port-opening rule makes it listen on another interface.
- `0.0.0.0` or `::` means the service accepts connections on every interface,
  for the corresponding address family. Review whether that exposure is
  necessary.

### Audit network kernel parameters

Compare the vendor network settings with the effective runtime values:

```bash
cat /usr/lib/sysctl.d/70-azurelinux-hardening.conf
```

```bash
sysctl \
  net.ipv4.ip_forward \
  net.ipv6.conf.all.forwarding \
  net.ipv4.conf.all.rp_filter \
  net.ipv4.conf.default.rp_filter \
  net.ipv4.conf.all.accept_redirects \
  net.ipv6.conf.all.accept_redirects \
  net.ipv4.conf.all.accept_source_route \
  net.ipv4.conf.all.send_redirects \
  net.ipv4.conf.all.log_martians \
  net.ipv4.tcp_syncookies
```

Expected result for the default configuration:

```output
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.tcp_syncookies = 1
```

| Parameter | Why the value matters |
| --------- | --------------------- |
| `ip_forward`, `forwarding` | The VM doesn't route traffic between interfaces, so it can't be used as an unintended path into another subnet. Routing software, NAT, and container networking turn this on deliberately. |
| `rp_filter` | Strict reverse path filtering drops a packet when the reply would leave through a different interface, which discards the simplest form of source address spoofing. |
| `accept_redirects` | The host ignores ICMP redirects, so a device on the same segment can't rewrite its routing table. |
| `accept_source_route` | Source-routed packets are dropped, so a sender can't dictate the path its traffic takes through your network. |
| `send_redirects` | The host doesn't advertise better routes to other systems. |
| `log_martians` | Packets with impossible source addresses are logged, which turns spoofing attempts into evidence you can query. |
| `tcp_syncookies` | The host keeps completing legitimate handshakes when the SYN backlog fills, which limits the effect of a SYN flood. |

The kernel applies the higher of the `all` and per-interface values for
`rp_filter`. Check the interface values as well as the global setting:

```bash
sudo sysctl -a | grep 'rp_filter'
```

## Add a host firewall and change network settings

Firewall allowances should match application requirements. Changes to network
hardening defaults require a reviewed network-design exception.

### Add a host firewall

Add a host firewall when NSG rules alone don't meet your requirements, for
example when you need per-VM rules managed inside the guest.

> [!WARNING]
> When `firewalld` starts, its default `public` zone allows only the `ssh` and
> `dhcpv6-client` services. Other services that listen on the network become
> unreachable until you allow them. Before you start the firewall, confirm you
> can reach the VM through the
> [serial console](/azure/virtual-machines/troubleshooting/serial-console-linux)
> or [Run command](/azure/virtual-machines/linux/run-command-managed).

1. List the services the VM exposes, so you can allow the ones the workload
   needs:

   ```bash
   sudo ss -tulnp
   ```

1. Install `firewalld`:

   ```bash
   sudo dnf install -y firewalld
   ```

   The package preset enables the service, so it starts at the next restart
   even if you don't start it now.

1. Start the firewall:

   ```bash
   sudo systemctl enable --now firewalld
   ```

1. Confirm the state, backend, default zone, and allowed services:

   ```bash
   sudo firewall-cmd --state
   sudo grep -E '^FirewallBackend=' /etc/firewalld/firewalld.conf
   sudo firewall-cmd --get-default-zone
   sudo firewall-cmd --list-services
   ```

   ```output
   running
   FirewallBackend=nftables
   public
   dhcpv6-client ssh
   ```

1. Add a rule for each workload service, as described in the next section.

Check active zones before changing a rule. A zone can be selected by an
interface or source binding; otherwise, the default zone applies:

```bash
sudo firewall-cmd --get-active-zones
```

### Add a rule without locking yourself out

`firewalld` keeps runtime and permanent configurations separately. This example
adds HTTPS to one zone without saving unrelated runtime changes. Use the zone
that handles the service's traffic; `public` is the `firewalld` default.

> [!WARNING]
> Removing the SSH allowance can prevent new management connections. Before you
> change SSH rules, confirm you can reach the VM through the serial console or
> Run command.

1. Select the zone and add the runtime rule:

   ```bash
   ZONE=public
   sudo firewall-cmd --zone="$ZONE" --add-service=https
   ```

   ```output
   success
   ```

1. Confirm the rule is live:

   ```bash
   sudo firewall-cmd --zone="$ZONE" --query-service=https
   ```

   ```output
   yes
   ```

1. Test the application from a client. Once it works, persist only this rule
   and confirm the permanent state:

   ```bash
   sudo firewall-cmd --permanent --zone="$ZONE" --add-service=https
   sudo firewall-cmd --permanent --zone="$ZONE" --query-service=https
   ```

   The query returns `yes`. A firewall query confirms the rule, not that the
   application is listening or that Azure permits the connection.

1. If you added the rule for this example and no longer need it, remove it
   from both configurations:

   ```bash
   sudo firewall-cmd --zone="$ZONE" --remove-service=https
   sudo firewall-cmd --permanent --zone="$ZONE" --remove-service=https
   ```

Avoid `--runtime-to-permanent` for a single-rule change unless you intend to
save **all** active runtime configuration. It can persist changes unrelated to
your task.

If you edit permanent files directly, reload to apply them. A reload replaces
runtime configuration, including unsaved runtime changes:

```bash
sudo firewall-cmd --reload
```

> [!TIP]
> For a temporary service allowance, use
> `sudo firewall-cmd --zone="$ZONE" --add-service=https --timeout=5m`.
> The allowance expires automatically. This isn't a rollback mechanism for
> other changes, such as removing SSH access.

### Change a network parameter

To change a parameter permanently, write it to a file under `/etc/sysctl.d/`
with a name that sorts after `70-azurelinux-hardening.conf`, such as
`90-network-local.conf`. For the precedence rules and how to apply the change,
see
[Inspect kernel hardening](./kernel-hardening.md#persist-a-parameter-change).

### Override the Azure-generated interface configuration

At first boot, cloud-init reads the network configuration from Azure metadata
and writes a `systemd-networkd` file for the primary interface. It rewrites the
file at each boot, so edits to that file don't persist.

| File | Created by | Purpose |
| ---- | ---------- | ------- |
| `/etc/systemd/network/10-cloud-init-eth0.network` | cloud-init | Configures `eth0` with DHCP for IPv4. Rewritten at each boot. |
| `/usr/lib/systemd/network/01-azure-unmanaged-sriov.network` | `azure-vm-utils` package | Leaves the Accelerated Networking virtual function unmanaged, because it's bonded to the synthetic interface. |

Identify the file that configures the primary interface:

```bash
networkctl status eth0 | grep 'Network File'
```

```output
                Network File: /etc/systemd/network/10-cloud-init-eth0.network
```

With Accelerated Networking, `networkctl list` also shows an interface such as
`ens1` as `enslaved` and `unmanaged`. Don't add configuration for it.

To change a setting for `eth0`, add a drop-in file instead of editing the
generated file. Drop-in files persist across restarts and apply on top of the
generated settings. This example adds a DNS server:

1. Create the drop-in directory and file:

   ```bash
   sudo mkdir -p /etc/systemd/network/10-cloud-init-eth0.network.d
   printf '[Network]\nDNS=<dns-server-ip>\n' | sudo tee \
     /etc/systemd/network/10-cloud-init-eth0.network.d/50-local.conf
   ```

1. Reload `systemd-networkd` and confirm that the drop-in is applied:

   ```bash
   sudo networkctl reload
   networkctl status eth0 | grep -A1 'Network File'
   resolvectl dns eth0
   ```

   The status output lists the drop-in under the generated file:

   ```output
                   Network File: /etc/systemd/network/10-cloud-init-eth0.network
                                 └─/etc/systemd/network/10-cloud-init-eth0.network.d/50-local.conf
   ```

   The `resolvectl` output lists your DNS server and the Azure resolver,
   `168.63.129.16`.

To remove the override, delete the drop-in file and run
`sudo networkctl reload`.

> [!CAUTION]
> A separate `.network` file that sorts before `10-cloud-init-eth0.network` and
> matches `eth0` replaces the generated configuration instead of extending it,
> because `systemd-networkd` uses only the first matching file. Use a drop-in
> unless you intend to own the entire interface configuration. Keep serial
> console access available when you test interface changes.

## Troubleshoot host firewall and network connectivity

### A connection times out instead of being refused

A refusal can indicate no listener or an explicit rejection. A timeout can
result from filtering, packet loss, or a routing problem. Start with the
listener and the zone handling the traffic:

```bash
sudo ss -tulnp
systemctl is-active firewalld
```

If `firewalld` is active, list the zone that handles the traffic:

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=<zone-name> --list-all
```

If a process is listening and no host firewall blocks the port, review the NSG
rules on the subnet and network interface in Azure.

### A service became unreachable after installing firewalld

The `public` zone allows only SSH and DHCPv6 client traffic. List the zone and
add a rule for the service, as described in
[Add a rule without locking yourself out](#add-a-rule-without-locking-yourself-out):

```bash
sudo firewall-cmd --list-all
```

### A change to the generated network file disappeared

cloud-init rewrites `/etc/systemd/network/10-cloud-init-eth0.network` at each
boot. Move the setting into a drop-in file, as described in
[Override the Azure-generated interface configuration](#override-the-azure-generated-interface-configuration).

### A firewall rule disappeared after a restart

Compare the runtime and permanent rules in the same zone. If the rule exists
only at runtime, add that specific rule with `--permanent`. Don't overwrite
the full permanent configuration just to save one service allowance.

### Traffic is dropped after adding a second network interface

Asymmetric routing and strict reverse path filtering don't coexist. Look for
the evidence in the kernel log:

```bash
sudo journalctl -k --grep='martian' --since '1 hour ago'
```

Fix the routing so replies leave through the interface the traffic arrived on.
Loosening `rp_filter` removes a spoofing control, so change it only when the
network design genuinely requires asymmetric paths.

### IP forwarding is enabled and you didn't enable it

Container runtimes and Kubernetes node components enable forwarding by design.
Confirm what set it before you change it back:

```bash
grep -rnsE 'ip_forward|conf\.(all|default)\.forwarding' \
  /etc/sysctl.conf /etc/sysctl.d /usr/lib/sysctl.d
```

Turning forwarding off on a node that runs containers breaks pod networking.

### firewall-cmd output doesn't match the live ruleset

Software that writes `nftables` rules directly bypasses `firewalld`. Compare
the generated ruleset with what `firewalld` reports. The `nft` command ships in
the `nftables` package:

```bash
sudo nft list ruleset
```

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Configure networking on Azure Linux](./configure-networking.md)
- [Manage identity and access](./identity-and-access.md)
- [Network security groups](/azure/virtual-network/network-security-groups-overview)
- [firewalld documentation](https://firewalld.org/documentation/)
