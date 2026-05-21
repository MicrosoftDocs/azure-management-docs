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

### Regular images

In Azure Linux 4.0 regular images, `ca-certificates` is preinstalled and provides the standard certificate bundle. This package comes from Fedora and distributes the Mozilla-managed certificate set.

For more information, see [Fedora ca-certificates package](https://packages.fedoraproject.org/pkgs/ca-certificates/ca-certificates/).

### Distroless container images

Distroless images don't include the shell and package-management tools required to install regular certificate bundles directly. In Azure Linux distroless images, `prebuilt-ca-certificates` is preinstalled and provides the same trust bundle as `ca-certificates`.

## Install a certificate

### Install a certificate in a regular image

To install a certificate in a regular image, follow the process described in [Fedora's Shared System Certificates](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/).

### Install a certificate in a distroless container

Distroless containers don't provide the standard shell environment required to install certificates directly. Instead, stage the certificate installation in a regular container image and then copy the trusted certificate artifacts into the distroless image.

The following example installs the AMEroot certificate into an Azure Linux ASP.NET XX.0 distroless image:

```dockerfile
# Stage 1: Prepare trust data in a regular Azure Linux image.
FROM <YOUR_CONTAINER_IMAGE> as configure-certs
# Example:
#FROM mcr.microsoft.com/azurelinux/base/core:4.0 as configure-certs

# Download and prepare the certificate.
RUN curl --retry 3 --retry-all-errors --create-dirs --output /app/ameroot.crt http://crl.microsoft.com/pkiinfra/certs/AMEROOT_ameroot.crt
RUN openssl x509 -inform DER -outform PEM -in /app/ameroot.crt -out /etc/pki/ca-trust/source/anchors/ameroot.pem

# Rebuild trust and verify the certificate.
RUN update-ca-trust extract && openssl verify /etc/pki/ca-trust/source/anchors/ameroot.pem

# Optional cleanup.
RUN find /etc/pki/ca-trust/extracted -name 'README' -delete

# Stage 2: Copy trust data into the distroless image.
FROM <YOUR_DISTROLESS_IMAGE> AS base
# Example;
#FROM mcr.microsoft.com/dotnet/aspnet:XX.0-azurelinux4.0-distroless-extra AS base

COPY --from=configure-certs /etc/pki/ca-trust/extracted /etc/pki/ca-trust/extracted

# Configure your application
```

## Related content

To learn more about Azure Linux, see the following resources:

- [Build RPM packages for Azure Linux](./build-rpm-packages.md)
- [Customize Azure Linux images with Image Customizer](./customize-images.md)
