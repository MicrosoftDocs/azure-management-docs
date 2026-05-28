---
title: Certificate Management in Azure Linux
description: This article explains how Azure Linux handles trusted root certificates and how to add custom certificates to regular Azure Linux images and distroless container images.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Certificate management in Azure Linux

This article explains how Azure Linux handles trusted root certificates and how to add custom certificates to regular Azure Linux images and distroless container images.

Azure Linux 4.0 uses Fedora's `ca-certificates` package. Fedora maintains and distributes the package, and the certificate bundle is sourced from [Mozilla's set of trusted CAs](https://wiki.mozilla.org/CA/Included_Certificates).

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Certificate storage and management

Azure Linux stores and manages certificates by using the shared trust store approach described in [Fedora's Shared System Certificates](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/).

## Available certificate bundles

In Azure Linux 4.0 regular images, `ca-certificates` is preinstalled and provides the standard certificate bundle. This package comes from Fedora and distributes the Mozilla-managed certificate set.

For more information, see [Fedora ca-certificates package](https://packages.fedoraproject.org/pkgs/ca-certificates/ca-certificates/).

## Install a certificate

To install a certificate in a regular image, follow the process described in [Fedora's Shared System Certificates](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/).

## Related content

To learn more about Azure Linux, see the following resources:

- [Build RPM packages for Azure Linux](./build-rpm-packages.md)
- [Customize Azure Linux images with Image Customizer](./customize-images.md)
