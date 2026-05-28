---
title: Overview of Azure Linux Container Images
description: This article provides an overview of the Azure Linux container images, including standard and distroless images, their use cases, and tagging policy.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/26/2026
---

# Overview of Azure Linux container images

This article provides an overview of the Azure Linux container images, including the standard base images and the distroless images, their intended use cases, and the tagging policy used to manage image versions.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Azure Linux container images

An Azure Linux base container is a minimal container image provided by Microsoft and built on the Azure Linux distribution. It provides a standardized, secure foundation for building custom, partner and golden container images.

```bash
mcr.microsoft.com/azurelinux-beta/base/core:4.0
```

## Azure Linux distroless images

Azure Linux distroless container images are lightweight and provide a minimal attack surface.

Distroless containers contain only the minimal set of packages, with anything extra removed (such as the package manager, libraries, and shell).

Due to their limited set of packages, distroless containers have a minimized security attack surface as well as reduced noise from vulnerability scanners. This generally translates to a reduced overhead of patching vulnerabilities, allowing developers to focus on building their application.

Azure Linux has three distroless images that are intended to support different application types and use cases:

- `minimal`: The smallest distroless image. This image is primarily intended for use with golang or other statically linked applications.

    ```bash
    mcr.microsoft.com/azurelinux-beta/distroless/minimal:4.0
    ```

- `base`: This image is primarily intended for C-based applications.

    ```bash
    mcr.microsoft.com/azurelinux-beta/distroless/base:4.0
    ```

- `debug`: This image adds [busybox](https://busybox.net/about.html) package to the base image and provides a lightweight debugging environment combining minimal versions of many common utilities into a single small executable.

    ```bash
    mcr.microsoft.com/azurelinux-beta/distroless/debug:4.0
    ```

  - To access the debug container and use the busybox utilities included in it, use the following `docker run` command to start a container and launch a shell inside it:

    ```bash
    docker run -it --rm mcr.microsoft.com/azurelinux-beta/distroless/debug:4.0 busybox sh
    ```

## Azure Linux container image tagging policy

The Azure Linux team tags every base container image upload with the `base/core:MajorVersion` and `base/core:FullVersion` tags.

The format of the tags is as follows:

- `Major Version`: X.X, where X.X is the major version number of the release (Example: 4.0).
- `Full Version`: `MajorVersion`.YYYYMMDD, where YYYYMMDD is the date the image was built (Example: 4.0.20260602).

This results in a `base/core:MajorVersion` tag that always represents the latest base container image release for a given major release, and a growing list of `base/core:FullVersion` tags that preserve a tag for each base container image that was pushed. Once a new major version of Azure Linux is released, base container image updates for the prior release continue to be published until the prior release is end of life.

## Related content

To learn more about Azure Linux, see [What is Azure Linux?](./azure-linux-overview.md)
