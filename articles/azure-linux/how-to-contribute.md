---
title: Contribute to Azure Linux
description: Learn how to contribute to Azure Linux by fixing bugs, improving packages, enhancing tooling, or contributing to documentation.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# How to contribute to Azure Linux

Thank you for your interest in contributing to Azure Linux. Whether you're fixing a typo, reporting a bug, or improving a package, every contribution matters and we're glad you're here.

Azure Linux is open source and built on top of the broader Linux ecosystem. This article describes **where** to contribute, **how** to get started, and **what** to expect along the way.

> [!IMPORTANT]
> Azure Linux contributions are scoped to **Azure Linux on Azure scenarios only**. Microsoft support and lifecycle commitments apply only to:
>
> - Azure Linux VM/VMSS, Azure Kubernetes Service (AKS) container host, and container images.
> - Customizations built on top of a prebuilt Azure Linux image (for example, with [Image Customizer](./customize-images.md)).
>
> Contributions targeting bare metal, ISO images, on-premises, other clouds, or images built from scratch from the [Azure Linux sources on GitHub](https://github.com/microsoft/azurelinux) are out of scope.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Ways to contribute

You can contribute to Azure Linux in several ways, depending on your interests and expertise:

- **[Contribute to documentation](./contribute-to-docs.md)**: Fix, improve, or expand guidance for running Azure Linux on Azure, including VMs, AKS, VMSS, and container workloads.
- **[Contribute to code](./contribute-to-code.md)**: Fix bugs, improve packages, or enhance tooling across the Azure Linux distro.

You can also contribute by reporting bugs, requesting new features, or providing feedback. Clear bug reports and well-articulated feature requests help the Azure Linux team identify issues and prioritize improvements effectively. For more information, see [Report issues and request features for Azure Linux](./report-issues-request-features.md) and [Azure Linux official support options](./support-options.md).

## Recommended contribution path

Azure Linux is part of a larger open-source family. We build on and actively participate in the communities around us, most notably the [Fedora Linux Project](https://fedoraproject.org/). When you contribute upstream, your work reaches more people, strengthens the ecosystem, and naturally flows back into Azure Linux as part of our regular update and release process.

We recommend the following approach to determine where your contribution should land:

1. **Start at the source**. Does your change touch a package owned by another project, such as `systemd`, `binutils`, or `rust`? Please bring it to that project's community first. They know the code best, and your improvement benefits every distribution that depends on it.
1. **Then look at Fedora**. Is your change a distribution-level packaging, integration, or policy change that would help multiple distros? The [Fedora Linux Project](https://fedoraproject.org/) is the right place. Azure Linux picks up those changes as part of our regular release process.
1. **Then bring it to Azure Linux**. If the change is truly specific to Azure Linux, such as an Azure integration logic, our tooling, or our documentation, we'd love to see it in an Azure Linux repository.

> [!TIP]
> When you open an Azure Linux PR that originated upstream, include a link to the upstream issue or PR. It speeds up the review by giving context to reviewers and showing the lineage of the change.

## Choose the correct repository for your contribution

Azure Linux spans multiple repositories. Filing in the right place helps your contribution get reviewed faster. The following table shows the primary Azure Linux repositories and the types of changes that belong in each:

| What you want to change | Repository |
| ----------------------- | ---------- |
| Azure Linux | [microsoft/azurelinux](https://github.com/microsoft/azurelinux) |
| Azure Linux image tools | [microsoft/azure-linux-image-tools](https://github.com/microsoft/azure-linux-image-tools) |
| Azure Linux dev tools | [microsoft/azure-linux-dev-tools](https://github.com/microsoft/azure-linux-dev-tools) |
| Azure Linux documentation | [Contribute to Azure Linux documentation](./contribute-to-docs.md) |

## Community expectations

We're building Azure Linux together, and we want this space to be welcoming and productive for everyone.

This project adopts the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any questions or comments.
