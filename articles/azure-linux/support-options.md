---
title: Azure Linux Official Support Options
description: Learn about the official support options available for Azure Linux running on Azure, including production support, bug reporting, security vulnerability reporting, and community engagement opportunities.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux official support options

Azure Linux provides official support for customers running workloads on Azure in supported scenarios. This includes support for virtual machines (VMs), virtual machine scale sets (VMSS), Azure Kubernetes Service (AKS) container hosts, and Azure Linux container images.

This article outlines the official support options available for Azure Linux, including how to get production support, report bugs and documentation issues, report security vulnerabilities, and engage with the community through office hours and community calls.

> [!IMPORTANT]
> Azure Linux is open source, but Microsoft support and lifecycle commitments apply only to **Azure scenarios**. Specifically:
>
> - Azure Linux Virtual Machines (VM) / Virtual Machine Scale Sets (VMSS), AKS container host, and container images are supported.
> - Bare metal, ISO images, on-premises, and other clouds aren't supported.
> - Customized images are supported only when built on top of a prebuilt Azure Linux image (for example, with [Image Customizer](./customize-images.md)). Images built from scratch from the [Azure Linux sources on GitHub](https://github.com/microsoft/azurelinux) aren't covered.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Production support

Azure Support is for **production issues only**. If you're running an Azure Linux production workload on Azure and you hit an outage, regression, or configuration problem that's affecting that workload, [open an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request) through the Azure portal.

Use Azure Support for production issues such as:

- Package management and repository issues affecting a running workload.
- System configuration and lifecycle management on production hosts.
- Security baseline guidance for a deployed environment.
- Image and deployment troubleshooting in production.

For bugs, documentation problems, feature requests, and general questions, use GitHub instead. See [Bugs, docs, and feature requests](#bugs-docs-and-feature-requests). GitHub issues aren't monitored for production support.

## Bugs, docs, and feature requests

For bug reports, documentation improvements, and feature requests, see [Report issues and request features for Azure Linux](./report-issues-request-features.md). All issues require a GitHub issue filed in the correct repository.

## Security vulnerabilities

**Please don't report security vulnerabilities through public GitHub issues**. Follow the guidance under [Manage CVEs](./manage-cves.md/#report-a-security-issue).

## Join our Azure Linux community calls

Azure Linux hosts monthly community calls where you can get help from our product and support teams.

The Azure Linux community calls are for Azure Linux users to get together and discuss new features, provide feedback, and learn more about how others use Azure Linux. In each session, we feature a new demo.

The schedule for the upcoming community calls is as follows:

| Date | Time | Meeting link |
| ---- | ---- | ------------ |
| 5/28/2026 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 7/23/2026 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 9/24/2026 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 11/19/2026 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 1/28/2027 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 3/25/2027 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |

## Stay up-to-date

Azure Linux publishes a [feature roadmap](https://github.com/orgs/microsoft/projects/970/views/2) that contains features that are in development and available for general availability (GA) and preview. We review this roadmap in each community call to highlight upcoming features and changes. We welcome you to leave feedback or ask questions on feature items.

## Attributions

The Azure Linux penguin icon is inspired by the Linux mascot ("Tux"), created by Larry Ewing.
