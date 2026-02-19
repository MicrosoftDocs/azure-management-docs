---
title: Troubleshooting Azure Linux Container Host for AKS package upgrade issues
description: How to troubleshoot Azure Linux Container Host for AKS package upgrade issues.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: troubleshooting
ms.date: 08/18/2024
# Customer intent: "As a system administrator managing Azure Linux Container Hosts for AKS, I want to troubleshoot package upgrade issues, so that I can ensure that all necessary updates are applied and the system remains stable and secure."
---

# Troubleshoot package upgrade issues on the Azure Linux Container Host

The Azure Linux Container Host for Azure Kubernetes Service (AKS) has `dnf-automatic` enabled by default. This systemd service runs daily and automatically installs recently published updated packages. Having this service enabled ensures that packages in the Azure Linux Container Host update automatically when Microsoft publishes a fix. 

Note that for some [Node OS Upgrade Channel](/azure/aks/auto-upgrade-node-image) settings, `dnf-automatic` is disabled by default.

[!INCLUDE [azure-linux-retirement](./includes/azure-linux-retirement.md)]

## Symptoms

Sometimes packages in the Azure Linux Container Host fail to receive automatic upgrades, which can lead to the following symptoms:

- Error messages appear when referencing or using an updated package.
- Packages don't function as expected.
- The Azure Linux Container Host package list shows outdated package versions. You can verify if the packages on your image are synchronized with recently published packages by visiting the repository on [packages.microsoft.com](https://packages.microsoft.com/cbl-mariner/) or checking the release notes in the [Azure Linux GitHub](https://github.com/microsoft/CBL-Mariner/releases) repository.

## Cause

Some packages, such as the Linux kernel, require a reboot for updates to take effect. To facilitate automatic reboots, the Azure Linux virtual machine (VM) runs the check-restart service. This service creates the `/var/run/reboot-required` file when a package update requires a reboot.

## Solution

To ensure that Kubernetes acts on the reboot request, we recommend setting up the [kured daemonset](/azure/aks/node-updates-kured). [Kured](https://github.com/kubereboot/kured) monitors your nodes for the `/var/run/reboot-required` file. When found, Kured drains the workload from the node and reboots it.

## Next steps

If the preceding steps don't resolve the issue, open a [support ticket](https://azure.microsoft.com/support/).
