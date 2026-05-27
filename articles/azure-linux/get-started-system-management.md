---
title: Get Started with Azure Linux System Management
description: Get started with the core system management tasks for Azure Linux, including kernel configuration, network setup, storage management, logging and monitoring, and package management.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: get-started
ms.date: 04/27/2026
---

# Get started with Azure Linux system management

This article provides an overview of the core operational tasks for running and maintaining Azure Linux deployments, including kernel configuration, network setup, package updates, storage management, and system monitoring. It also points to the detailed guidance in each area, which you can use to effectively configure and manage your Azure Linux environment.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Core system management topics

| Topic | Description |
| ----- | ----------- |
| [Kernel](./kernel-management.md) | Kernel selection, configuration, boot parameters, and LTS/HWE kernel tracks to match your hardware and workload needs |
| [Network configuration](./configure-networking.md) | Managing network interfaces, DHCP, static IPs, and advanced features such as VLANs, bridges, and bonded interfaces with systemd-networkd |
| [Storage and filesystems](./storage-file-system-management.md) | Disk management, partition configuration, filesystem selection, and mount options for Azure Linux deployments |
| [Logging and monitoring](./logging-monitoring.md) | System logging with journald, Azure Monitor integration, and monitoring tools for visibility and troubleshooting |
| [Package management](./package-management-overview.md) | DNF 5 package manager, repository configuration, RPM packages, and dependency resolution for keeping your system current and secure |

## Related content

- **New to Azure Linux system management?** Start with [Manage the Azure Linux kernel](./kernel-management.md) to understand the foundation, then work through [Configure networking on Azure Linux](./configure-networking.md) for connectivity basics.
- **Deploying at scale?** [Logging and monitoring on Azure Linux](./logging-monitoring.md) covers integration with Azure Monitor and observability best practices.
- **Need to manage packages?** [Manage Azure Linux packages with DNF 5](./manage-packages.md) covers DNF 5 commands and repository configuration.
