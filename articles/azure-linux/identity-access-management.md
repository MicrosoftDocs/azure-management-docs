---
title: Azure Linux Identity and Access Management
description: Learn how Azure Linux manages identity and access, including SSH key-based authentication, passwordless Entra ID login, and hybrid domain join support with SSSD and realmd.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Azure Linux identity and access management

Azure Linux provides secure identity and access management out of the box. SSH key-based authentication is enabled by default, password authentication is disabled, and Microsoft Entra ID integration is available with minimal configuration. For hybrid environments, SSSD and realmd support domain join scenarios including Active Directory and LDAP.

This article explains how Azure Linux manages identity and access, including the default SSH key-based authentication configuration, how to enable passwordless login with Microsoft Entra ID, and how to join hybrid environments using SSSD and realmd.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Verify authentication configuration

Check your current SSH authentication settings to ensure that password authentication is disabled and that the Microsoft Entra ID SSH extension is installed:

```bash
# Verify SSH password authentication is disabled (effective config)
sudo sshd -T | grep -i '^passwordauthentication'

# Check if the Entra ID SSH extension is installed
dnf list --installed | grep -i aad
```

## SSH authentication defaults

Azure Linux images ship with **key-based SSH authentication** enabled and **password authentication disabled**. This reduces the attack surface by preventing brute-force password attacks.

### Default configuration

- SSH key-based authentication is the default for all Azure Linux images.
- Password authentication is disabled. On Azure Linux, cloud-init drops in `/etc/ssh/sshd_config.d/50-cloud-init.conf` with `PasswordAuthentication no` during provisioning, which overrides the upstream default in `/etc/ssh/sshd_config`.
- The Azure virtual machine (VM) deployment parameter `--authentication-type` controls which authentication methods are enabled at provisioning time. Valid values are `ssh`, `password`, or `all`.

### Verify SSH configuration

Confirm that password authentication is disabled by reading the effective sshd configuration (this is the value sshd is actually using, regardless of which file in `/etc/ssh/sshd_config.d/` set it):

```bash
sudo sshd -T | grep -i '^passwordauthentication'
```

Example output:

```text
passwordauthentication no
```

If you need to re-enable password authentication (not recommended for production), set it in a drop-in under `/etc/ssh/sshd_config.d/` and restart the SSH service:

```bash
echo 'PasswordAuthentication yes' | sudo tee /etc/ssh/sshd_config.d/99-passwords.conf
sudo systemctl restart sshd
```

> [!CAUTION]
> Enabling password authentication increases your exposure to brute-force attacks. Use SSH keys or Entra ID authentication whenever possible.

## Microsoft Entra ID SSH

Azure Linux supports **passwordless SSH authentication** through [Microsoft Entra ID](/entra/fundamentals/what-is-entra). This integration works out of the box with minimal configuration and is a key differentiator for Azure Linux deployments.

### Prerequisites for Entra ID SSH

- An Azure VM running Azure Linux with a system-assigned or user-assigned managed identity.
- The `aad-ssh-login` VM extension installed on the target VM.
- The user must have the **Virtual Machine Administrator Login** or **Virtual Machine User Login** role assigned on the VM or its resource group.

### Install the Entra ID SSH extension

If the extension isn't already installed, add it to your VM using the [`az vm extension set`](/cli/azure/vm/extension#az-vm-extension-set) command.

```azurecli-interactive
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory \
    --name AADSSHLoginForLinux \
    --resource-group <resource-group> \
    --vm-name <vm-name>
```

### Sign in with Entra ID

Connect with your Entra ID credentials to SSH into the VM using the [`az ssh vm`](/cli/azure/ssh#az-ssh-vm) command.

```azurecli-interactive
az ssh vm \
    --resource-group <resource-group> \
    --name <vm-name>
```

> [!NOTE]
> Entra ID SSH login eliminates the need to manage and distribute SSH keys across your organization. Access is controlled through Azure role-based access control (RBAC) and Entra ID conditional access policies.

## Hybrid identity with SSSD and realmd

**System Security Services Daemon (SSSD)** and **realmd** are available from the Azure Linux package repositories and enable hybrid identity scenarios such as Active Directory domain join and LDAP integration.

### Supported scenarios

- **Microsoft Entra ID integration**: Authenticate users against Entra ID for centralized identity management.
- **Active Directory domain join**: Join Azure Linux VMs to an on-premises or cloud-hosted Active Directory domain.
- **LDAP integration**: Connect to LDAP directories for user authentication and authorization.

### Join an Active Directory domain

1. Install SSSD and realmd, and then verify they're present:

   ```bash
   sudo dnf install -y sssd realmd
   dnf list --installed | grep -E 'sssd|realmd'
   ```

1. Discover the domain:

   ```bash
   sudo realm discover <domain-name>
   ```

1. Join the domain:

   ```bash
   sudo realm join <domain-name> -U <admin-user>
   ```

1. Verify the domain join:

   ```bash
   realm list
   id <domain-user>@<domain-name>
   ```

### Configure SSSD for Entra ID

> [!NOTE]
> For detailed Entra ID and Active Directory integration guidance, see the [Microsoft Entra ID documentation](/entra/identity/devices/howto-vm-sign-in-azure-ad-linux).

## Related content

- [Overview of Azure Linux network security](./network-security-overview.md)
- [Configure mandatory access control (MAC) in Azure Linux with SELinux and Landlock](./mandatory-access-control.md)
- [Azure Linux security overview](./security-overview.md)
