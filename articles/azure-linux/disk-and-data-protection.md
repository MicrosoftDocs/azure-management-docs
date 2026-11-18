---
title: Protect disks and data in Azure Linux
description: Encrypt a data disk in the guest with LUKS2 on Azure Linux, keep it available across restarts, and check which integrity features the kernel supports.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Protect disks and data in Azure Linux

Azure encrypts managed disks at rest. Linux Unified Key Setup version 2 (LUKS2)
adds guest-managed encryption with a separate key-management boundary.
It doesn't protect mounted data from a privileged process in the guest.

This guide covers an empty **data disk**, automatic unlocking, and recovery.
It isn't a procedure for encrypting the operating system disk.

## Default disk encryption and integrity settings

The general-purpose Azure Linux 4 VM image uses an `ext4` root filesystem
without guest LUKS encryption. Azure managed disk encryption operates
separately. Configure a LUKS data volume only if your requirements call for
guest-managed keys.

This guide covers guest LUKS encryption of data disks. File integrity
monitoring and application allowlisting aren't configured by default. The
`aide`, `fapolicyd`, and `ima-evm-utils` packages are available from the Azure
Linux repository but aren't installed. For details, see
[Check integrity feature support](#check-integrity-feature-support).

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.
- An empty data disk attached to the VM, if you plan to create a LUKS2 volume.
- Secure storage for recovery credentials and the LUKS header backup.
- A maintenance window for changes that affect startup or mounted data.

## Verify disk encryption status

Inspect both the guest storage layout and the Azure disk configuration before
deciding whether additional encryption is needed.

### Inspect the storage you have

Identify the target device before making changes:

```bash
lsblk -f
findmnt --real
```

For the default guest configuration, expect an `ext4` root filesystem rather
than a LUKS mapping. A configured LUKS volume appears as `crypto_LUKS` in
`lsblk -f` output.

Inspect platform encryption through Azure:

```azurecli
az disk show \
  --resource-group <resource-group-name> \
  --name <disk-name> \
  --query "encryption"
```

For the available platform options, including customer-managed keys and
encryption at host, see
[Overview of managed disk encryption](/azure/virtual-machines/disk-encryption-overview).

## Configure LUKS2 encryption for a data disk

LUKS configuration is independent of Azure managed disk encryption. Define
key custody and recovery requirements before formatting a data disk.

### Encrypt a data disk with LUKS2

> [!WARNING]
> Formatting and filesystem creation destroy data on the target device.
> Confirm the disk identity, size, filesystem, and mount state. Don't continue
> if it contains data or if you aren't certain which disk it is.

1. Identify the empty data disk by a path that doesn't change when disks are
   added or reordered. Kernel names such as `/dev/sdc` aren't stable across
   restarts:

   ```bash
   lsblk -f
   ls -l /dev/disk/by-id/
   ```

1. Set the persistent path and confirm the disk:

   Replace `<data-disk-id>` with the identifier of the empty data disk you
   confirmed in the previous step.

   ```bash
   DEVICE=/dev/disk/by-id/<data-disk-id>
   lsblk -f "$DEVICE"
   ```

   The output must show the intended disk with no filesystem and no mount
   point.

1. Create the LUKS2 container and open it. `cryptsetup` prompts for the
   passphrase and requires you to type `YES` to confirm the format:

   ```bash
   sudo cryptsetup luksFormat --type luks2 "$DEVICE"
   sudo cryptsetup open "$DEVICE" encrypted_data
   ```

1. Create a filesystem on the mapped device and mount it. Everything written
   through `/dev/mapper/encrypted_data` is encrypted on the underlying disk:

   ```bash
   sudo mkfs.ext4 /dev/mapper/encrypted_data
   sudo mkdir -p /mnt/encrypted-data
   sudo mount /dev/mapper/encrypted_data /mnt/encrypted-data
   ```

#### Verify the volume

```bash
sudo cryptsetup status encrypted_data
```

Relevant lines for an open LUKS2 volume:

```output
/dev/mapper/encrypted_data is active and is in use.
  type:    LUKS2
```

```bash
sudo cryptsetup luksDump "$DEVICE" | head -n 20
lsblk -f "$DEVICE"
```

Confirm three things in the output:

- `cryptsetup status` reports the device as `active` and names the cipher in
  use.
- `luksDump` reports `Version: 2`, along with the key slots that can unlock the
  volume. Each populated slot is a credential that grants access to the data.
- `lsblk -f` shows the disk with type `crypto_LUKS` and your filesystem on the
  mapped device above it.

#### Back up the LUKS header

The header holds the key slots. Losing it can make the data inaccessible even if you know the passphrase. Set `DEVICE` to the persistent path of the LUKS disk from [Encrypt a data disk with LUKS2](#encrypt-a-data-disk-with-luks2), and then back up the header:

```bash
sudo cryptsetup luksHeaderBackup "$DEVICE" \
  --header-backup-file <backup-file-path>
```

> [!WARNING]
> Store the header backup securely and separately from the data disk. A backup
> plus a passphrase valid when it was created can unlock the data, even if that
> passphrase was subsequently removed. Include header backups in your key
> protection and retirement procedures.

### Make the volume available after a restart

This configuration stores an automatic-unlock key on the OS disk. It
protects a separately copied data disk only while the key remains inaccessible
to the recipient. It doesn't protect against access to both disks, their
backups, or a compromised running VM. Choose a different key-management design
if your requirements exclude this arrangement.

Keep the original recovery passphrase in secure storage independent of the VM.

The steps use the `DEVICE` variable, the `encrypted_data` mapping name, and the `/mnt/encrypted-data` mount point from [Encrypt a data disk with LUKS2](#encrypt-a-data-disk-with-luks2). If you started a new shell session, set `DEVICE` again.

1. Create a dedicated key file and add it to a key slot. Don't overwrite a key
   file already used by a volume:

   ```bash
   sudo mkdir -p /etc/luks
   sudo chmod 0700 /etc/luks
   sudo dd if=/dev/urandom of=/etc/luks/encrypted_data.key bs=512 count=8
   sudo chmod 0400 /etc/luks/encrypted_data.key
   sudo cryptsetup luksAddKey "$DEVICE" /etc/luks/encrypted_data.key
   ```

1. Get the volume's UUID, which is stable even if the device path changes:

   ```bash
   sudo cryptsetup luksUUID "$DEVICE"
   ```

1. Add the mapping to `/etc/crypttab`, using the UUID from the previous step:

   ```text
   encrypted_data UUID=<luks-uuid> /etc/luks/encrypted_data.key luks,nofail
   ```

1. Add the mount to `/etc/fstab`. Include `nofail` so a problem with this
   volume doesn't stop the VM from starting:

   ```text
   /dev/mapper/encrypted_data /mnt/encrypted-data ext4 defaults,nofail 0 2
   ```

1. Test automatic unlocking and mounting on a nonproduction VM:

   ```bash
   sudo umount /mnt/encrypted-data
   sudo cryptsetup close encrypted_data
   sudo systemctl daemon-reload
   sudo reboot
   ```

   After the VM restarts, confirm the volume unlocked and mounted on its own:

   ```bash
   sudo cryptsetup status encrypted_data
   findmnt /mnt/encrypted-data
   ```

> [!CAUTION]
> Keep `nofail` on both data-volume entries if the VM must start without the
> disk. This does **not** make applications safe to run without their data.
> Test disk absence and recovery before production, and retain a management
> path such as Azure Run Command or the
> [serial console](/azure/virtual-machines/troubleshooting/serial-console-linux).

#### Require the mount before starting an application

A service can otherwise write to the unmounted directory on the OS disk. For a
systemd service that requires this volume, add a drop-in:

```bash
sudo systemctl edit <service-name>.service
```

```ini
[Unit]
RequiresMountsFor=/mnt/encrypted-data
ConditionPathIsMountPoint=/mnt/encrypted-data
```

The dependency requires the mount; the condition checks that the path is a
mount point. Restart the service after applying the change and exercise its
data access. On a test VM, also confirm that the service doesn't start when
the data disk is absent.

When the disk is available again, confirm the UUID and key path, then start the
service and check `findmnt /mnt/encrypted-data`. A VM that boots successfully
isn't proof that its data-dependent workload recovered.

## Check integrity feature support

Inspect available integrity features separately from encryption:

```bash
grep -E '^CONFIG_(DM_VERITY|FS_VERITY)=' /boot/config-$(uname -r)
```

Example output:

```output
CONFIG_DM_VERITY=m
CONFIG_FS_VERITY=y
```

Confirm the device-mapper target loads, then check whether any verity target is
actually active:

```bash
sudo modprobe dm_verity
sudo dmsetup targets | grep '^verity'
sudo dmsetup table --target verity
```

`dmsetup targets` lists the available target. The final command checks active
verity mappings. The general-purpose VM root isn't protected by dm-verity.
For Azure Container Linux (ACL) on Azure Kubernetes Service (AKS), see
[Azure Container Linux overview](./azure-container-linux-overview.md).

`CONFIG_FS_VERITY=y` shows kernel support only. It doesn't mean files are
protected or that the mounted filesystem is configured for fs-verity.

### Check for integrity monitoring tools

The image doesn't install or configure file integrity monitoring or
application allowlisting. The following packages are available from the Azure
Linux repository:

| Package | Purpose |
| ------- | ------- |
| `aide` | Advanced Intrusion Detection Environment (AIDE) records file attributes and digests in a database and reports later changes. |
| `fapolicyd` | File access policy daemon that allows or denies running files based on a trust policy. |
| `ima-evm-utils` | Utilities to sign files and manage Integrity Measurement Architecture (IMA) and Extended Verification Module (EVM) attributes. |

Check whether they're installed:

```bash
rpm -q aide fapolicyd ima-evm-utils
```

Expected result for the default configuration:

```output
package aide is not installed
package fapolicyd is not installed
package ima-evm-utils is not installed
```

The kernel includes the IMA and EVM security modules, which appear in
`/sys/kernel/security/lsm`. Installing a package doesn't enable monitoring or
enforcement. Each tool needs a policy, a baseline, and a review process that
your organization defines. Test the policy in nonproduction before you enforce
it.

## Troubleshoot LUKS2 data disk issues

### cryptsetup reports the device is busy

Something is still using it. Find what, and release it before retrying. Set `DEVICE` to the persistent path of your LUKS disk. The `encrypted_data` name is the mapping from [Encrypt a data disk with LUKS2](#encrypt-a-data-disk-with-luks2):

```bash
lsblk -f "$DEVICE"
findmnt --source /dev/mapper/encrypted_data
sudo dmsetup ls
```

Unmount the filesystem first, then close the mapping with
`sudo cryptsetup close encrypted_data`.

### No key available with this passphrase

Check the device identity and whether it has a readable LUKS header. Set `DEVICE` to the persistent path of the disk you're trying to unlock:

```bash
sudo cryptsetup luksDump "$DEVICE" | head -n 5
```

An invalid header can indicate the wrong disk or header damage. Don't format
the device to resolve an unlock failure.

### The volume doesn't unlock after a restart

Check the mapper name in `/etc/crypttab` against the name in `/etc/fstab`, and
confirm the key file is readable by root only and referenced by the correct
path. Then read what happened during startup:

```bash
sudo journalctl -b --grep='cryptsetup|crypttab' --no-pager
```

If the automatic key is unavailable, identify the disk again in your current
session. Shell variables don't survive a restart. Confirm the persistent path,
then unlock with the retained recovery passphrase:

```bash
DEVICE=/dev/disk/by-id/<data-disk-id>
lsblk -f "$DEVICE"
sudo cryptsetup open "$DEVICE" encrypted_data
```

After correcting the key path or disk attachment, mount the volume and verify
its data before starting dependent services.

### The device path changed after attaching disks

Kernel device names are assigned in discovery order. Use `/dev/disk/by-id/`
paths for `cryptsetup` commands and UUIDs in `/etc/crypttab`, and don't record
`/dev/sd*` names in configuration files.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Manage storage and filesystems](./storage-file-system-management.md)
- [Overview of managed disk encryption](/azure/virtual-machines/disk-encryption-overview)
- [cryptsetup documentation](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/home)
