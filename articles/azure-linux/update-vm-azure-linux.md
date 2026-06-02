---
title: Update an Azure Linux Virtual Machine (VM)
description: This article describes how to keep an Azure Linux virtual machine (VM) up to date using image-based or package-based updates.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Update an Azure Linux virtual machine (VM)

This article explains how to keep an Azure Linux virtual machine (VM) updated using:

- **Image-based updates**: Redeploy the VM from a newer Azure Linux image. This approach aligns naturally with safe deployment practices (SDP).
- **Package-based updates**: Apply package updates in place on a running VM. This approach requires more coordination to roll out safely.

Select the method that best fits your release process. You can also combine both approaches.

## Image-based updates

Azure Linux ships a new image release each month that includes the previous month's package updates. To stay current, redeploy your fleet from the latest Azure Linux image release.

A typical rollout looks like this:

1. Pull the latest Azure Linux image when it's published.
1. Add the latest versions of any extra packages your workload uses.
1. Test the new image.
1. Roll out the image using a safe deployment practice (SDP).

### Automatic OS image upgrades for Virtual Machine Scale Sets

If your workload runs on a Virtual Machine Scale Set (VMSS), you can opt in to [Automatic operating system (OS) image upgrades](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade) to roll out new image versions to your scale set without manual intervention.

## Package-based updates

Package-based updates apply package changes to a VM that's already running. You can orchestrate them to follow a safe deployment framework, but doing so requires more coordination than image-based updates.

### Use dnf-automatic

> [!WARNING]
> `dnf-automatic` doesn't reboot the VM after a kernel update.
> Without staged rollout, a bad package update can be applied to every VM at once and cause an outage. Combine `dnf-automatic` with your own staging and monitoring before enabling it broadly.

Add the `dnf-automatic` package to your image to automate in-VM package updates. After installing the package, enable and start the `dnf5-automatic.timer` systemd timer using the following commands:

```bash
sudo dnf install dnf-automatic
sudo systemctl enable dnf5-automatic.timer
sudo systemctl start dnf5-automatic.timer
```

### Use Azure VM guest patching

For a managed alternative, enable [Azure VM guest patching](/azure/virtual-machines/automatic-vm-guest-patching), which applies package updates following a safe deployment framework managed by Azure.

## Related content

- [Create an Azure Linux VM](./create-vm-azure-linux-4-0.md)
- [Package management on Azure Linux](./package-management-overview.md)
- [Customize images with Image Customizer](./customize-images.md)
