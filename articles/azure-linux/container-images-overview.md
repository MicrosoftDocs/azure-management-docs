---
title: Overview of Azure Linux Container Images
description: This article provides an overview of the Azure Linux container images, including standard and distroless images, their use cases, and tagging policy.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/26/2026
ai-usage: ai-assisted
---

# Overview of Azure Linux container images

This article provides an overview of the Azure Linux container images, including the standard base images and the distroless images, their intended use cases, and the tagging policy used to manage image versions.

## Azure Linux container images

An Azure Linux base container is a minimal container image provided by Microsoft and built on the Azure Linux distribution. It provides a standardized, secure foundation for building custom, partner and golden container images.

```bash
mcr.microsoft.com/azurelinux/core:4
```

For available image tags, see [Azure Linux core tags in Microsoft Artifact Registry](https://mcr.microsoft.com/en-us/artifact/mar/azurelinux/core/tags).

## Azure Linux distroless images

Azure Linux distroless container images are lightweight and provide a minimal attack surface.

Distroless containers contain only the minimal set of packages, with anything extra removed (such as the package manager, libraries, and shell).

Due to their limited set of packages, distroless containers have a minimized security attack surface as well as reduced noise from vulnerability scanners. This generally translates to a reduced overhead of patching vulnerabilities, allowing developers to focus on building their application.

Azure Linux has three distroless images that are intended to support different application types and use cases:

- `minimal`: The smallest distroless image. This image is primarily intended for use with golang or other statically linked applications.

    ```bash
    mcr.microsoft.com/azurelinux/core:4-distroless-minimal
    ```

- `base`: This image is primarily intended for C-based applications.

    ```bash
    mcr.microsoft.com/azurelinux/core:4-distroless
    ```

- `debug`: This image adds [busybox](https://busybox.net/about.html) package to the base image and provides a lightweight debugging environment combining minimal versions of many common utilities into a single small executable.

    ```bash
    mcr.microsoft.com/azurelinux/core:4-distroless-debug
    ```

  - To access the debug container and use the busybox utilities included in it, use the following `docker run` command to start a container and launch a shell inside it:

    ```bash
    docker run -it --rm mcr.microsoft.com/azurelinux/core:4-distroless-debug /bin/sh
    ```

## Azure Linux golden container images

Azure Linux golden container images are prebuilt container images based on the Azure Linux distribution. Each image includes the packages and dependencies needed to support a specific application or workload.

Golden containers provide a standardized foundation for running common applications and services, reducing the need to build and maintain custom container images.

The following golden container images are available:

- BusyBox
- NGINX
- Node.js
- PostgreSQL
- Python
- PyTorch
- Telegraf

## Azure Linux container image tagging policy

The Azure Linux team tags every base container image upload with the `core:MajorVersion` and `core:FullVersion` tags.

The format of the tags is as follows:

- `Major Version`: The major version number of the release (for example, `4` for Azure Linux 4).
- `Full Version`: `YYYYMMDDRR`, where `YYYYMMDD` is the date the image was built and `RR` is the revision for that day, usually `00`.

This results in a `core:MajorVersion` tag that always represents the latest base container image release for a given major release, and a growing list of `core:FullVersion` tags that preserve a tag for each base container image that was pushed. Once a new major version of Azure Linux is released, base container image updates for the prior release continue to be published until the prior release is end of life.

## Related content

To learn more about Azure Linux, see [What is Azure Linux?](./azure-linux-overview.md)
