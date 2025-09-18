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

# Troubleshoot issues with package upgrades on the Azure Linux Container Host

The Azure Linux Container Host for Azure Kubernetes Service (AKS) has `dnf-automatic` enabled by default, a systemd service that runs daily and automatically installs any recently published updated packages. Having this service enabled ensures that packages in the Azure Linux Container Host should automatically update when a fix is published. Note that for some settings of [Node OS Upgrade Channel](/azure/aks/auto-upgrade-node-image), `dnf-automatic` is disabled by default.

> [!IMPORTANT]
> Starting on **30 November 2025**, AKS will no longer support or provide security updates for Azure Linux 2.0. Starting on **31 March 2026**, node images will be removed, and you'll be unable to scale your node pools. Migrate to a supported Azure Linux version by [**upgrading your node pools**](/azure/aks/upgrade-aks-cluster) to a supported Kubernetes version or migrating to [`osSku AzureLinux3`](/azure/aks/upgrade-os-version). For more information, see [[Retirement] Azure Linux 2.0 node pools on AKS](https://github.com/Azure/AKS/issues/4988).

## Symptoms

However, sometimes the packages in the Azure Linux Container Host fail to receive automatic upgrades, which can lead to the following symptoms:

- Error messages while referencing or using an updated package.
- Packages not functioning as expected.
- Outdated versions of packages are displayed when checking the Azure Linux Container Host package list. You can verify if the packages on your image are synchronized with the recently published packaged by visiting the repository on [packages.microsoft.com](https://packages.microsoft.com/cbl-mariner/) or checking the release notes in the [Azure Linux GitHub](https://github.com/microsoft/CBL-Mariner/releases) repository.

## Cause

Some packages, such as the Linux Kernel, require a reboot for the updates to take effect. To facilitate automatic reboots, the Azure Linux virtual machine (VM) runs the check-restart service, which creates the `/var/run/reboot-required` file when a package update requires a reboot.

## Solution

To ensure that Kubernetes acts on the request for a reboot, we recommend setting up the [kured daemonset](/azure/aks/node-updates-kured). [Kured](https://github.com/kubereboot/kured) monitors your nodes for the `/var/run/reboot-required` file and, when it's found, drains the work off the node and reboots it.

## Next steps

If the preceding steps don't resolve the issue, open a [support ticket](https://azure.microsoft.com/support/).
