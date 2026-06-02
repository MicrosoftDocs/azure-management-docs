---
title: Developer Guide for Azure Linux
description: This guide covers common developer tasks on Azure Linux, including building and packaging software, customizing images, and managing certificates for development and testing purposes.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: get-started
ms.date: 04/27/2026
---

# Developer guide for Azure Linux

This article covers common developer tasks on Azure Linux, from building and packaging software to customizing images for your environment.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Developer tasks

| Task | Description |
| ---- | ----------- |
| [Build RPM packages](./build-rpm-packages.md) | Package proprietary software, apply patches, or bundle internal tools using `mock` or `azldev`. |
| [Customize images](./customize-images.md) | Modify Azure Linux base images like adding packages, configuring settings and applying partition layouts, without booting a virtual machine (VM). |
| [Manage certificates](./certificate-management.md) | Add custom certificates to regular and distroless container images. |

## Related content

To learn more about Azure Linux, see [Azure Linux overview](./azure-linux-overview.md).
