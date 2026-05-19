---
title: Disk and Data Protection in Azure Linux
description: Learn how Azure Linux protects data at rest and verifies filesystem integrity using full disk encryption, fs-verity, dm-verity, and default filesystem configurations.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Disk and data protection in Azure Linux

Azure Linux protects data at rest and verifies filesystem integrity through multiple complementary mechanisms. This article covers full disk encryption, file-level integrity verification with fs-verity, block-level verification with dm-verity, and the default filesystem configuration.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Verify disk protection

Check your current encryption and filesystem verification status:

```bash
# Check if LUKS encryption is active on a device
sudo cryptsetup status /dev/<device>

# Check if fs-verity is supported on the running kernel
ls /proc/sys/fs/verity/
```

## Full disk encryption

Azure Linux supports full disk encryption using LUKS2 with optional TPM 2.0 integration for sealed key management. Disk encryption isn't enabled by default to avoid performance overhead and key management complexity for workloads that don't require it.

### Azure server-side encryption (recommended)

For most customers, we recommend [Azure server-side encryption](/azure/virtual-machines/disk-encryption) for protecting data at rest. Azure manages encryption keys automatically and the protection is transparent to the operating system (OS).

> [!IMPORTANT]
> Azure Disk Encryption (ADE) is deprecated. If you currently use ADE, plan to migrate to Azure server-side encryption. For migration guidance, see [Migrate from Azure Disk Encryption](/azure/virtual-machines/disk-encryption-migrate).

### Enable LUKS2 encryption

Azure Linux ships with `cryptsetup` pre-installed. Use LUKS2 to encrypt data partitions when you need OS-level encryption.

1. Identify the target partition:

   ```bash
   lsblk -f
   ```

1. Format the partition with LUKS2:

   ```bash
   sudo cryptsetup luksFormat --type luks2 /dev/sdX
   ```

1. Open the encrypted partition:

   ```bash
   sudo cryptsetup open /dev/sdX encrypted_data
   ```

1. Create a filesystem on the encrypted device:

   ```bash
   sudo mkfs.ext4 /dev/mapper/encrypted_data
   ```

1. Mount and use the encrypted partition:

   ```bash
   sudo mount /dev/mapper/encrypted_data /mnt/data
   ```

### TPM 2.0 integration

Azure Linux supports binding LUKS2 keys to TPM 2.0 PCR values, so that encrypted volumes unlock automatically when the measured boot chain matches the expected state. TPM-based enrollment requires a virtual machine (VM) provisioned with [Trusted Launch](/azure/virtual-machines/trusted-launch).

Use `systemd-cryptenroll` to enroll a TPM 2.0 key:

```bash
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=7 /dev/sdX
```

## File integrity with fs-verity

fs-verity provides file-level integrity verification using a Merkle tree built into the filesystem. Azure Linux compiles fs-verity support into the kernel by default (`CONFIG_FS_VERITY=y`), and the `fsverity-utils` package is available in the repositories.

### fs-verity use cases

- **Package verification**: Verify that installed packages haven't been modified after installation.
- **Container image integrity**: Ensure container image layers remain unaltered at runtime.
- **Immutable system binaries:** Protect critical system binaries from tampering.

### Enable fs-verity on a file

> [!NOTE]
> fs-verity requires a filesystem that supports it, such as ext4 or btrfs, with the `verity` feature enabled.

1. Install `fsverity-utils`:

   ```bash
   sudo dnf install -y fsverity-utils
   ```

1. Enable fs-verity on a file:

   ```bash
   sudo fsverity enable /path/to/file
   ```

1. Verify the file's integrity measurement:

   ```bash
   fsverity measure /path/to/file
   ```

After you enable fs-verity on a file, any attempt to modify the file results in an I/O error, which prevents silent tampering.

### Verify kernel support

Confirm that fs-verity support is compiled into your kernel:

```bash
grep CONFIG_FS_VERITY /boot/config-$(uname -r)
```

Example output:

```output
CONFIG_FS_VERITY=y
```

## Root filesystem integrity with dm-verity

dm-verity provides cryptographic verification of entire block devices by computing a hash tree over all blocks and checking each block at read time. If any block has been modified, dm-verity returns an I/O error instead of the corrupted data.

### Availability

dm-verity is available on the following Azure Linux images:

| Image type | dm-verity status |
| ---------- | ---------------- |
| Azure Linux general-purpose | Available (opt-in) |
| Azure Container Linux | **Mandatory** (root filesystem integrity) |

### Check dm-verity status

Install `cryptsetup` (which provides `veritysetup`) and list verity targets on the system:

```bash
sudo dnf install -y cryptsetup

# List device-mapper targets and check for verity type
sudo dmsetup table --target verity

# Check the status of a specific verity target
sudo veritysetup status <target-name>
```

### How dm-verity works

dm-verity performs the following steps to ensure root filesystem integrity:

1. At build time, a hash tree is computed over every block of the root filesystem.
1. The root hash is stored in a trusted location (such as the kernel command line or a signed partition).
1. At boot, the kernel verifies each block against the hash tree before returning data to the application.

This approach ensures that any modification to the root filesystem, such as malware, misconfiguration, or disk corruption, is detected immediately.

## Default filesystem

Azure Linux uses **ext4** as the default filesystem. ext4 is a production-hardened filesystem with broad compatibility, well-understood security properties, and support for features like fs-verity.

You can choose alternative filesystems based on your workload requirements. Azure Linux also supports XFS and btrfs in the default kernel configuration.

## Azure Container Linux (ACL) considerations

Azure Container Linux applies stricter integrity enforcement than general-purpose Azure Linux images:

- **Mandatory dm-verity**: The root filesystem is protected by dm-verity. Any modification to root filesystem blocks results in an I/O error.
- **Read-only root filesystem**: The root filesystem is mounted read-only to prevent runtime tampering. Writable state is isolated to designated overlay partitions.
- **fs-verity enforcement**: System binaries are protected by fs-verity, providing an extra integrity verification layer on top of dm-verity.
- **A/B partition scheme**: System updates use an A/B partition layout for atomic, rollback-capable updates. If an update fails verification, the system rolls back to the previous known-good partition automatically.

## Related content

- [Get started with Secure Boot and Trusted Boot in Azure Linux](./secure-boot-trusted-boot.md)
- [Azure Linux certifications and compliance](./certifications-compliance.md)
- [Configure mandatory access control (MAC) in Azure Linux with SELinux and Landlock](./mandatory-access-control.md)
