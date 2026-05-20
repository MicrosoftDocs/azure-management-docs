---
title: Get Started with Secure Boot and Trusted Boot in Azure Linux
description: Learn how to verify and ensure the integrity of the boot chain in Azure Linux using Secure Boot, measured boot, and TPM 2.0 attestation.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: get-started
ms.date: 04/27/2026
---

# Get started with Secure Boot and Trusted Boot in Azure Linux

Azure Linux secures the boot chain from firmware through userspace so that every component that runs has been verified before execution. Secure Boot, measured boot, and TPM 2.0 attestation work together to establish a hardware-rooted chain of trust that protects against bootkits, rootkits, and unauthorized code execution.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Verify boot integrity

Confirm that Secure Boot, kernel lockdown, and TPM 2.0 are active on a running system using the following commands:

1. Check Secure Boot status:

   ```bash
   mokutil --sb-state
   ```

   Expected output: `SecureBoot enabled`.

1. Check kernel lockdown mode:

   ```bash
   cat /sys/kernel/security/lockdown
   ```

   Expected output: `none [integrity] confidentiality` (brackets indicate the active mode).

1. Verify a TPM 2.0 device is present (requires a virtual machine (VM) provisioned with [Trusted Launch](/azure/virtual-machines/trusted-launch)):

   ```bash
   ls /dev/tpm0 && cat /sys/class/tpm/tpm0/tpm_version_major
   ```

   Expected output: the device path and `2`.

## UEFI Secure Boot

UEFI Secure Boot is mandatory for all Azure Linux images. Every component in the boot chain is cryptographically signed, and the firmware verifies each signature before passing control to the next stage.

### Secure Boot sign chain

> [!IMPORTANT]
> Microsoft maintains custody of all signing keys used in the Azure Linux boot chain. Default images trust only Microsoft-signed components.

UEFI Secure Boot signs the following components:

| Component | Description | Signed by |
| --------- | ----------- | --------- |
| SHIM | First-stage bootloader | Microsoft UEFI CA |
| GRUB | Second-stage bootloader (default) | Microsoft |
| systemd-boot | Alternative second-stage bootloader | Microsoft |
| Kernel | Azure Linux kernel | Microsoft |

:::image type="content" source="./media/secure-boot.png" alt-text="Screenshot of a diagram showing the UEFI secure boot chain in Azure Linux." lightbox="./media/secure-boot.png":::

### Default behavior

> [!IMPORTANT]
> Secure Boot and integrity-mode kernel lockdown are only active when the Azure VM is provisioned with Trusted Launch. The [Verify boot integrity](#verify-boot-integrity) section of this guide assumes a Trusted Launch VM. On other VM configurations the same commands return `SecureBoot disabled` and `[none] integrity confidentiality`.

Secure Boot is the default for Azure Linux images that are deployed on Azure VMs provisioned with [Trusted Launch](/azure/virtual-machines/trusted-launch). Trusted Launch is what enables UEFI Secure Boot and the vTPM in the guest. On non-Trusted-Launch VMs, `mokutil --sb-state` returns `SecureBoot disabled` and kernel lockdown is `none`.

You can disable Secure Boot through Azure Virtual Machines (VM) or Virtual Machine Scale Sets (VMSS) provisioning flows, but doing so also disables [kernel lockdown](#kernel-lockdown).

### Custom kernel signing with MOK

> [!NOTE]
> MOK enrollment adds your key alongside the built-in Microsoft keys. It doesn't replace or remove the default trust chain.

If you build a custom kernel or third-party kernel modules, you must enroll a Machine Owner Key (MOK) so SHIM can verify your components.

1. Generate a signing key pair:

   ```bash
   openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER \
     -out MOK.der -nodes -days 36500 -subj "/CN=Custom Kernel Signing Key/"
   ```

1. Enroll the public key:

   ```bash
   sudo mokutil --import MOK.der
   ```

1. Reboot the system. The SHIM MokManager prompts you to confirm enrollment.
1. Sign your kernel or module:

   ```bash
   sudo sbsign --key MOK.priv --cert MOK.der --output /boot/vmlinuz-custom /boot/vmlinuz-custom
   ```

## Kernel lockdown

Kernel lockdown restricts runtime operations that could compromise a verified boot chain. The lockdown mode is tied to the Secure Boot state.

| Secure Boot state | Lockdown mode | Effect |
| ----------------- | ------------- | ------ |
| Enabled | `integrity` | Blocks unsigned module loading, restricts `/dev/mem`, prevents hibernation image tampering |
| Disabled | `none` | No kernel lockdown restrictions |

### Verify lockdown mode

Verify the current kernel lockdown mode by reading the `/sys/kernel/security/lockdown` file:

```bash
cat /sys/kernel/security/lockdown
```

In the output, the active mode appears in brackets. For example:

```output
none [integrity] confidentiality
```

### Integrity mode restrictions

When lockdown is set to `integrity`, the kernel blocks:

- Loading unsigned or improperly signed kernel modules
- Direct access to `/dev/mem` and `/dev/kmem`
- Access to MSRs that could modify kernel behavior
- Writing to ACPI tables
- Using `kexec` to load unsigned kernels

## TPM 2.0

Azure Linux supports TPM 2.0 exclusively. TPM 1.x is deprecated and not supported.

> [!IMPORTANT]
> A TPM device is only exposed to the guest when the Azure VM is provisioned with [Trusted Launch](/azure/virtual-machines/trusted-launch), which enables vTPM and Secure Boot. Azure Linux VMs without Trusted Launch don't have `/dev/tpm0` and can't use TPM-based features.

### Measured boot

During each stage of the boot process, the running component measures (hashes) the next component and extends those measurements into TPM Platform Configuration Registers (PCRs). This creates a tamper-evident log of everything that ran.

:::image type="content" source="./media/measured-boot-new.png" alt-text="Screenshot of a diagram showing the measured boot process in Azure Linux." lightbox="./media/measured-boot-new.png":::

### Remote attestation

TPM 2.0 measured boot enables remote attestation through the Azure Attestation Service. A relying party can request a TPM quote, which is a signed snapshot of the PCR values, to verify that the VM booted an unmodified software stack.

### Required packages

Install the TPM 2.0 userspace tools:

```bash
sudo dnf install -y tpm2-tools tpm2-tss
```

### Verify TPM availability and read PCR values

1. Confirm the TPM device exists:

   ```bash
   ls /dev/tpm0
   ```

1. Read the current PCR values:

   ```bash
   sudo tpm2_pcrread sha256
   ```

   Non-zero values in PCRs 0–9 confirm that measured boot is active.

## Key and certificate management

Microsoft manages the full signing key lifecycle for the following Azure Linux boot components:

- **Signing chain**: Every component from SHIM through the kernel is signed with keys held by Microsoft.
- **Key rotation**: Microsoft rotates signing keys according to an internal policy. Updated images carry the new signatures automatically; no action is required on your part.
- **MOK keys**: If you enrolled custom MOK keys, you manage their lifecycle independently. Re-enrollment is required if you regenerate your key pair.

## Azure Container Linux considerations

Azure Container Linux (the container-optimized variant) enforces a stricter security posture than general-purpose Azure Linux.

| Capability | General-purpose Azure Linux | Azure Container Linux |
| ---------- | --------------------------- | --------------------- |
| Secure Boot | Enabled by default, can disable | **Mandatory**, can't disable |
| dm-verity | Optional | **Mandatory** for root filesystem |
| Measured boot with attestation | Supported | **Required** |

- **dm-verity**: The root filesystem is protected by dm-verity, which verifies every block read against a signed hash tree. Any tampering causes the read to fail.
- **Immutable root**: The root filesystem is read-only. Persistent state is stored on separate writable partitions.

> [!NOTE]
> Because you can't disable Secure Boot on Azure Container Linux, any custom kernel modules must be signed with an enrolled MOK. For more information, see [Custom kernel signing with MOK](#custom-kernel-signing-with-mok).

## Related content

- [Kernel hardening in Azure Linux](./kernel-hardening.md)
- [Configure mandatory access control (MAC) in Azure Linux with SELinux and Landlock](./mandatory-access-control.md)
- [Azure Linux certifications and compliance](./certifications-compliance.md)
