---
title: Manage identity and access in Azure Linux
description: Audit SSH authentication, local accounts, and sudo rules on Azure Linux, and understand which access controls Azure owns and which the guest owns.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Manage identity and access in Azure Linux

Access management for Azure Linux 4 spans Azure resource permissions, SSH
authentication, local accounts, and `sudo` policy.

## Default SSH and root access settings

Direct root SSH sign-in is denied. On a VM provisioned for key-based access,
SSH public-key authentication is enabled and password authentication is
disabled. Provisioning supplies the initial account and key; review those
choices alongside the image settings.

The OpenSSH server applies hardened session limits, shows a banner before
authentication, and uses the system cryptographic policy with message
authentication codes (MACs) limited to SHA-2. For each setting and the file
that sets it, see [Review the OpenSSH defaults](#review-the-openssh-defaults).

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.
- An open SSH session you can keep, so a configuration mistake doesn't lock you
  out.

## Audit SSH, account, and sudo configuration

Inspect Azure access, the effective SSH settings, and guest accounts before
changing authentication.

### Know which layer owns which control

| Control | Owned by | Notes |
| ------- | -------- | ----- |
| Who can reach TCP port 22 | Azure NSG, then a host firewall if you install one | Neither one authenticates. They limit exposure. See [Secure the host network](./network-security.md). |
| The first SSH public key on the VM | Azure, at VM creation | Provisioning writes it to the user's `authorized_keys` file. |
| Resetting or adding a key later | Azure, through the VMAccess extension | Runs with root privileges inside the guest. Restrict who holds the Azure role that can invoke it. |
| Which accounts exist and what they can do | The guest | Inspect local accounts, `sudo` rules, and `sshd` configuration using guest tools. Azure role assignments don't describe these permissions. |
| Tokens for Azure services | Azure managed identity | A workload identity, not a sign-in method. Any code running on the VM can request those tokens, so a managed identity's role assignments are effectively granted to anyone with code execution on the VM. |

Keep managed identity permissions scoped to what the workload needs. For more
information, see
[Managed identities for Azure resources](/entra/identity/managed-identities-azure-resources/overview).

### Audit the effective SSH configuration

`sshd -T` parses `/etc/ssh/sshd_config` and its included files. It shows the
configuration a new daemon would load, not whether the running daemon has
reloaded recent edits:

```bash
sudo sshd -T | grep -E \
  '^(permitrootlogin|pubkeyauthentication|passwordauthentication) '
```

Expected result for a VM provisioned for key authentication:

```output
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
```

When interpreting the output:

- `sshd -T` prints the configuration for a generic connection. It doesn't
  evaluate `Match` blocks, which can relax settings for a specific user,
  group, or source address. To test one, supply the connection details:
  `sudo sshd -T -C user=<user-name>,host=<host-name>,addr=<ip-address>`.
- Inspect provisioning drop-ins before choosing where to make a change:

  ```bash
  sudo ls -1 /etc/ssh/sshd_config.d/
  ```

  Expected result for the default configuration:

  ```output
  30-azurelinux-cis.conf
  40-azurelinux-crypto-policies.conf
  50-azurelinux.conf
  50-client-alive-interval.conf
  50-cloud-init.conf
  50-permit-root-login.conf
  ```

> [!CAUTION]
> Re-enabling password authentication exposes every account with a password to
> credential-guessing traffic. Keys plus NSG restrictions are the safer default.
> If a tool genuinely requires passwords, scope it with a `Match` block instead
> of enabling it globally.

Also inspect `kbdinteractiveauthentication` when reviewing password-based
access. Disabling `PasswordAuthentication` alone doesn't describe every
authentication method:

```bash
sudo sshd -T | grep '^kbdinteractiveauthentication '
```

Expected result for the default configuration:

```output
kbdinteractiveauthentication no
```

### Review the OpenSSH defaults

The image configures the OpenSSH server through drop-in files in
`/etc/ssh/sshd_config.d/`. OpenSSH reads them in lexical order and keeps the
first value it obtains for each keyword.

| File | Source | Settings |
| ---- | ------ | -------- |
| `30-azurelinux-cis.conf` | Azure Linux release package | `DenyUsers root`, `Banner /etc/issue.net`, `LoginGraceTime 60`, `MaxAuthTries 4`, `MaxStartups 10:30:60`, and SHA-2 MACs |
| `40-azurelinux-crypto-policies.conf` | OpenSSH server package | Includes the key exchange, cipher, and signature algorithms from the system cryptographic policy |
| `50-azurelinux.conf` | OpenSSH server package | `KbdInteractiveAuthentication no`, `GSSAPIAuthentication yes`, `UsePAM yes`, `X11Forwarding yes`, and logging to the `AUTHPRIV` facility |
| `50-client-alive-interval.conf` | Azure Linux release package | `ClientAliveInterval 180` |
| `50-cloud-init.conf` | cloud-init, at provisioning | `PasswordAuthentication no` on key-based deployments |
| `50-permit-root-login.conf` | Azure Linux release package | `PermitRootLogin no` |

#### Session and access settings

Read the effective session and access settings:

```bash
sudo sshd -T | grep -E \
  '^(denyusers|maxauthtries|logingracetime|maxstartups|clientaliveinterval|banner|requiredrsasize) '
```

```output
logingracetime 60
maxauthtries 4
clientaliveinterval 180
requiredrsasize 2048
banner /etc/issue.net
denyusers root
maxstartups 10:30:60
```

#### Cryptographic algorithms

Read the effective algorithms:

```bash
sudo sshd -T | grep -E '^(kexalgorithms|ciphers|macs) '
```

With the `DEFAULT` cryptographic policy:

- Key exchange prefers `mlkem768x25519-sha256`, a hybrid post-quantum method,
  and doesn't include SHA-1 methods.
- Ciphers are AES-GCM, ChaCha20-Poly1305, and AES-CTR. CBC-mode ciphers aren't
  offered.
- MACs are SHA-2 only.
- RSA keys must be at least 2,048 bits, and RSA signatures must use SHA-2.

Expected result for the default configuration:

```output
ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes128-gcm@openssh.com,aes128-ctr
macs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
kexalgorithms mlkem768x25519-sha256,curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512
```

#### Forwarding settings

Read the forwarding settings:

```bash
sudo sshd -T | grep -E \
  '^(x11forwarding|allowtcpforwarding|allowagentforwarding) '
```

Expected result for the default configuration:

```output
x11forwarding yes
allowtcpforwarding yes
allowagentforwarding yes
```

X11, TCP, and agent forwarding are enabled by default. Disable the ones your
workload doesn't use, as described in
[Change SSH configuration without losing access](#change-ssh-configuration-without-losing-access).

### Check client compatibility

Current OpenSSH clients connect without changes. To see what a client
negotiates, connect with verbose output and inspect the `kex:` lines:

```bash
ssh -v <user-name>@<host-name> exit 2>&1 | grep -E 'kex: (algorithm|host key|server->client)'
```

Older clients and libraries can fail when they support only algorithms that the
server doesn't offer:

| Client error | Cause | Resolution |
| ------------ | ----- | ---------- |
| `no matching key exchange method found` | The client offers only SHA-1 key exchange, such as `diffie-hellman-group14-sha1`. | Update the client or library. |
| `no matching cipher found` | The client offers only CBC-mode ciphers. | Update the client or enable AES-GCM or AES-CTR in its configuration. |
| `no matching MAC found` | The client offers only SHA-1 MACs, such as `hmac-sha1`. | Update the client or library. |
| `no matching host key type found` | The client accepts only `ssh-rsa` (SHA-1) host key signatures. | Update the client so that it supports `rsa-sha2-256`, `rsa-sha2-512`, or `ssh-ed25519`. |
| `Permission denied (publickey)` with a DSA key, or an RSA key smaller than 2,048 bits | DSA isn't supported, and `RequiredRSASize 2048` rejects small RSA keys. | Generate an Ed25519 key or an RSA key of at least 3,072 bits, and add it to the account. |

Setting the system cryptographic policy to `LEGACY` adds SHA-1 key exchange
methods but doesn't re-enable SHA-1 MACs, because `30-azurelinux-cis.conf`
sets the MACs before the policy file is read. It also weakens TLS and other
applications that use the system policy. Update clients instead of weakening
the server.

### Review accounts and privilege

Review local accounts, accounts with root privileges, and `sudo` rules.

#### List local accounts

Inspect local accounts with user IDs of at least 1000 and their configured
shells. This list doesn't establish that each account can sign in:

```bash
awk -F: '($3 >= 1000) && ($1 != "nobody") {print $1, $3, $7}' /etc/passwd
```

List local accounts with password hashes without printing the hashes:

```bash
sudo awk -F: '($2 ~ /^\$/) {print $1}' /etc/shadow
```

`!` or `*` denotes an unusable password value; an empty field is different and
must be reviewed against the authentication configuration. Password state
alone doesn't determine whether SSH-key access is allowed.

#### Check for root-equivalent accounts

Accounts with root privileges. Only `root` should be listed:

```bash
awk -F: '($3 == 0) {print $1}' /etc/passwd
```

#### Review sudo rules

Privilege escalation rules, including anything provisioning added:

```bash
sudo grep -rnE '^[^#]' /etc/sudoers.d/
getent group wheel
```

The output is similar to the following example for a VM whose initial
administrator is `azureuser`:

```output
/etc/sudoers.d/90-cloud-init-users:4:azureuser ALL=(ALL) NOPASSWD:ALL
/etc/sudoers.d/10-azurelinux-cis:3:Defaults use_pty
/etc/sudoers.d/10-azurelinux-cis:4:Defaults logfile="/var/log/sudo.log"
wheel:x:10:azureuser
```

To include the base rules, also run `sudo grep -nE '^[^#]' /etc/sudoers`. The
base file grants `ALL` to `root` and to members of the `wheel` group, with a
password prompt.

The initial administrator account has passwordless `sudo`. During
provisioning, cloud-init writes `/etc/sudoers.d/90-cloud-init-users` with a
`NOPASSWD:ALL` rule for that account and adds the account to `wheel`. The
`10-azurelinux-cis` file comes from the image and runs `sudo` commands in a
pseudo-terminal with logging to `/var/log/sudo.log`.

This rule is expected on Azure. When you deploy with SSH key authentication,
the account has no password, so a rule that requires one would block `sudo`.
Protect the account by keeping key-based SSH sign-in, and review any other
`NOPASSWD: ALL` entries and groups that grant administrative access. Include directory-backed accounts in the review if you
have configured an identity provider; `/etc/passwd` only lists local entries.

To see who actually used those privileges, query the audit trail and the
journal as described in
[Configure security logging and auditing](./logging-and-auditing.md).

## Change SSH access and connect to a directory service

Keep key-based access unless an approved requirement calls for a change.

### Keep a recovery path

Establish a recovery path before changing authentication settings.

- **Reset or add a key from Azure.** The VMAccess extension writes a public
  key into the account's `authorized_keys` file:

  ```azurecli
  az vm user update \
    --resource-group <resource-group-name> \
    --name <vm-name> \
    --username <user-name> \
    --ssh-key-value "<public-key-contents>"
  ```

  For details, see
  [VMAccess extension](/azure/virtual-machines/extensions/vmaccess).

- **Azure Serial Console.** The
  [Azure Serial Console](/azure/virtual-machines/troubleshooting/serial-console-linux)
  bypasses the network, but it presents a normal login prompt. If no local
  account has a password, you can't sign in through it. Decide deliberately
  whether a break-glass account with a password is part of your recovery plan,
  and if so, protect it accordingly.

### Change SSH configuration without losing access

1. Put your settings in a file under `/etc/ssh/sshd_config.d/`. OpenSSH keeps
   the first value it obtains for each keyword, so a file that sorts earlier
   wins. To override a value from `30-azurelinux-cis.conf`, use a name that
   sorts before it, such as `20-local.conf`. List the directory before choosing
   a name:

   ```bash
   sudo ls -1 /etc/ssh/sshd_config.d/
   ```

1. Validate the configuration syntax before reloading `sshd`:

   ```bash
   sudo sshd -t
   ```

   Success produces no output.

1. Reload the service, which keeps existing sessions alive:

   ```bash
   sudo systemctl reload sshd
   ```

1. Keep the first session open. Establish a second connection using the intended
   account and authentication method, then confirm the parsed settings:

   ```bash
   sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication) '
   ```

If the new session fails, you still have the first one to revert the change.

### Connect to a directory service

The `sssd` and `realmd` packages are available but aren't installed by default.
Check their presence before installing them:

```bash
rpm -q sssd realmd
```

Expected result before installing the directory integration packages:

```output
package sssd is not installed
package realmd is not installed
```

```bash
sudo dnf install -y sssd realmd
```

Installing them prepares the VM for directory integration. It doesn't join a
domain and doesn't change authentication on its own. Domain enrollment depends
on DNS resolution of the directory's service records, time synchronization,
credentials, and directory-side policy. Follow your directory provider's
documentation, and test the result in a nonproduction environment.

## Troubleshoot SSH sign-in failures

### Permission denied (publickey)

Work from the server's view of the failure first:

```bash
sudo journalctl -u sshd -n 50 --no-pager
```

The output is similar to the following example. Look for lines after the
service start that name the account and the reason for the failure:

```output
Oct 30 10:13:01 <host-name> systemd[1]: Starting sshd.service - OpenSSH server daemon...
Oct 30 10:13:02 <host-name> sshd[884]: Server listening on 0.0.0.0 port 22.
Oct 30 10:13:02 <host-name> sshd[884]: Server listening on :: port 22.
Oct 30 10:13:02 <host-name> systemd[1]: Started sshd.service - OpenSSH server daemon.
```

Check the affected account's key-file ownership, permissions, and labels.
Run these commands as that account:

```bash
ls -ld ~/.ssh
ls -l ~/.ssh/authorized_keys
sudo restorecon -Rv ~/.ssh
```

`~/.ssh` should be `700`, `authorized_keys` should be `600`, and both should be
owned by the user. Run `ssh -vvv <user-name>@<host-name>` from the client to
see which key it offered.

### sshd won't start after a configuration change

Run `sudo sshd -t` and fix the reported line. Existing sessions survive a
failed reload, so use the session you kept open. If you've lost all sessions,
use Azure Serial Console or Azure Run Command to revert the file.

### An account can't sign in and the key looks correct

Check whether the account itself is locked or expired:

```bash
sudo passwd -S <user-name>
sudo chage -l <user-name>
```

`L` in the `passwd -S` output means the password is locked, which also blocks
console sign-in even when SSH keys work.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Secure the host network](./network-security.md)
- [Configure security logging and auditing](./logging-and-auditing.md)
- [Configure SELinux security](./security-enhanced-linux-security.md)
