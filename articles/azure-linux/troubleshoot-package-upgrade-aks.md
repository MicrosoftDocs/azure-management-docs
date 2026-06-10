---
title: Troubleshoot Package Upgrade Issues on Azure Linux Container Host for Azure Kubernetes Service (AKS)
description: Guidance on troubleshooting issues where packages on the Azure Linux Container Host for AKS fail to receive automatic upgrades, including identifying symptoms, understanding causes, and applying solutions.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: troubleshooting-general
ms.date: 04/28/2026
---

# Troubleshoot package upgrade issues on Azure Linux Container Host for Azure Kubernetes Service (AKS)

The Azure Linux Container Host for Azure Kubernetes Service (AKS) has `dnf-automatic` enabled by default, a systemd service that runs daily and automatically installs any recently published updated packages. Having this service enabled ensures that packages in the Azure Linux Container Host should automatically update when a fix is published. However, sometimes the packages in the Azure Linux Container Host fail to receive automatic upgrades, which can prevent the system from receiving the latest fixes and security updates in a timely manner.

This article provides guidance on how to troubleshoot issues where packages on the Azure Linux Container Host for AKS fail to receive automatic upgrades, including identifying the symptoms, understanding the underlying causes, and applying recommended solutions to ensure that package updates are applied successfully.

> [!NOTE]
> For some settings of the [AKS node operating system (OS) upgrade channel](/azure/aks/auto-upgrade-node-image), `dnf-automatic` is disabled by default.

## Symptoms

When packages on the Azure Linux Container Host fail to receive automatic upgrades, you might notice one or more of the following symptoms:

- Error messages while referencing or using an updated package.
- Packages not functioning as expected.
- Outdated versions of packages are displayed when checking the Azure Linux Container Host package list. You can verify if the packages on your image are synchronized with the recently published packaged by visiting the repository on [packages.microsoft.com](https://packages.microsoft.com/azurelinux/) or checking the release notes in the [Azure Linux GitHub](https://github.com/microsoft/azurelinux/releases) repository.

## Cause

Some packages, such as the Linux Kernel, require a reboot for the updates to take effect. To facilitate automatic reboots, the Azure Linux virtual machine (VM) runs the check-restart service, which creates the `/var/run/reboot-required` file when a package update requires a reboot.

## Solution

To ensure that Kubernetes acts on the request for a reboot, we recommend setting up the [kured daemonset](/azure/aks/node-updates-kured). [Kured](https://github.com/kubereboot/kured) monitors your nodes for the `/var/run/reboot-required` file and, when it's found, drains the work off the node and reboots it.

## Next steps

If the preceding steps don't resolve the issue, open a [support ticket](https://azure.microsoft.com/support/).
