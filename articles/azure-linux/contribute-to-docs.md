---
title: Contribute to Azure Linux Documentation
description: Learn how to contribute to Azure Linux documentation by fixing errors, improving guidance, or adding new content for Azure Linux on Azure scenarios.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Contribute to Azure Linux documentation

This article explains how you can contribute to the Azure Linux documentation by fixing errors, improving guidance, or adding new content that helps users run Azure Linux on Azure and provides guidance for supported Azure Linux scenarios.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## In-scope documentation contributions

The Azure Linux documentation covers **Azure scenarios only**, which is in line with Microsoft's support and lifecycle commitments. We maintain and accept contributions for content that helps people use Azure Linux on Azure, specifically:

- Guidance for Azure Linux VMs and VMSS, AKS, and container images.
- Customizations built on top of a prebuilt Azure Linux image (for example, with [Image Customizer](./customize-images.md)).

If you see a gap, an error, or an opportunity to improve Azure-focused guidance, please share it with us.

## Out-of-scope documentation contributions

To keep the documentation focused and maintainable, the following types of contributions will be closed:

- **Content for scenarios outside Microsoft's support and lifecycle commitments**, including bare metal, ISO images, on-premises, other clouds, or images built from scratch from the [Azure Linux sources on GitHub](https://github.com/microsoft/azurelinux).
- **Duplicate content** that restates what already exists in another page without adding new value.
- **Promotional or vendor-specific content** that highlights a particular product, tool, or service outside the Azure Linux ecosystem.
- **Speculative or unverified guidance** that hasn't been tested on a supported Azure Linux configuration.
- **Large unsolicited rewrites** submitted without a prior issue and maintainer approval.

## Style and formatting guidelines

Azure Linux documentation follows the [Microsoft Learn documentation style guide](/contribute/content/style-quick-start). All contributions should align with these standards. Pull requests (PRs) that don't follow these conventions might be sent back for revision before review begins.

## Submit a documentation change

> [!IMPORTANT]
> **All documentation contributions require an issue first**. Don't open a PR without an associated issue. PRs without a linked issue might be closed without review.

Steps to submit a documentation change:

1. **Open an issue.** Describe the gap, error, or opportunity for improvement. Be as descriptive as possible; include the Azure service, Azure Linux version, workload type, and any relevant context or reproduction steps. **Wait for a maintainer to acknowledge the issue before starting work**.
1. **Search for duplicates**. Before filing, check existing issues and PRs to avoid duplicating work.
1. **Fork and branch**. Once a maintainer confirms the issue, create a personal fork of the repository and a branch with a descriptive name (for example, `fix/aks-quickstart-typo` or `docs/selinux-recipe`).
1. **Submit a PR**. Link the approved issue in your PR description. Keep your changes concise and task-oriented. Include context such as the Azure service, Azure Linux version, and workload type so reviewers can evaluate your contribution.
1. **Wait for review.** A maintainer will review your PR. Be prepared to make changes if requested. Not all PRs are guaranteed to be merged.

## What to expect after you submit a PR

- **Reviews are best effort**. Maintainers review contributions as capacity allows. Response times can vary and there are no guaranteed service-level agreements (SLAs) on community PRs.
- **Not all contributions will be merged**. PRs might be closed if they fall outside the Azure-supported scope, duplicate existing content, or don't align with the documentation direction.
- **You might be asked to revise**. Reviewers might request changes to scope, accuracy, or formatting before a PR can be approved.
- **Issues might be closed without action**. If an issue is incomplete, out of scope, or has no activity after 30 days, it might be closed.
- **Stale PRs will be closed**. PRs with no activity for 30 days might be closed automatically. You can reopen if you want to continue work on the PR.

## Related content

For more information about contributing to Azure Linux, see [How to contribute to Azure Linux](./how-to-contribute.md).
