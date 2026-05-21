---
title: Customize Azure Linux images with Image Customizer
description: Learn how to use Image Customizer to modify Azure Linux images for your specific scenarios, including adding packages, configuring users, and applying custom system settings without booting a VM.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Customize Azure Linux images with Image Customizer

Image Customizer is an open-source tool that modifies existing Azure Linux images to suit your specific scenario. It uses `chroot` and loopback block devices to perform customizations without booting a virtual machine (VM), making the process fast, reliable, and easy to integrate into CI/CD workflows. This is the same technology used to build the official Azure Linux images. For the full configuration reference and advanced usage, see the [Image Customizer documentation](https://microsoft.github.io/azure-linux-image-tools/imagecustomizer/README.html).

> [!NOTE]
> Azure Image Builder (AIB) integration with Image Customizer isn't yet available. For now, you can use Image Customizer directly to customize Azure Linux images.

With Image Customizer, you can:

- Add or remove packages.
- Add files or directories.
- Configure users and system settings.
- Apply custom partition layouts.
- Produce output images in multiple formats.

Image Customizer also supports nested customization, so you can further customize an already-customized image. For teams that build multiple images, consider creating a shared custom base image first to reduce maintenance overhead.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- **Docker** installed on your host.
- **A base image**: Any Azure Linux image (from the Azure Marketplace or one you already maintain).
- **A configuration file**: A YAML or JSON file that describes your modifications.

## Supported host systems

You can run Image Customizer on:

- Azure Linux
- Ubuntu 22.04

## Use Image Customizer

Image Customizer runs as a container published to the [Microsoft Artifact Registry (MCR)](https://mcr.microsoft.com/azurelinux/imagecustomizer).

### List available tags

List available tags for the Image Customizer container with the following command:

```bash
curl -s "https://mcr.microsoft.com/v2/azurelinux/imagecustomizer/tags/list" | jq '.tags[]'
```

### Customize an image

Customize an image by running the Image Customizer container with your base image and configuration file mounted into the container. For example:

```bash
docker run --rm \
    --privileged \
    -v "<shared-dir>:z" \
    -v "/dev:/dev" \
    "mcr.microsoft.com/azurelinux/imagecustomizer:latest" \
    imagecustomizer \
        --image-file <base-image.vhdx> \
        --config-file <config-file.yaml> \
        --output-image-format raw \
        --output-image-file <output-image.raw> \
        --build-dir "/tmp"
```

Replace the following values:

| Placeholders | Description |
| ------------ | ----------- |
| `<shared-dir>` | Absolute path to the directory containing your base image and configuration file. The customized image is also written here. |
| `<base-image.vhdx>` | Path to the base image file to modify. |
| `<config-file.yaml>` | Path to the configuration file that describes your modifications. |
| `<output-image.raw>` | Path for the customized output image. |

## Related content

For more information, see the [Image Customizer documentation](https://microsoft.github.io/azure-linux-image-tools/imagecustomizer/README.html).
