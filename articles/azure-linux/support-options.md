---
title: Azure Linux Official Support Options
description: Learn about the official support options available for Azure Linux running on Azure, including support, bug reporting, security vulnerability reporting, and community engagement opportunities.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux official support options

This article outlines the official support options available for Azure Linux, including how to get support, report bugs and documentation issues, report security vulnerabilities, and engage with the community through office hours and community calls.

> [!IMPORTANT]
> Azure Linux is open source, but Microsoft support and lifecycle commitments apply only to **Azure scenarios**. Specifically:
>
> **Supported scenarios**
>
> - Azure Linux Virtual Machines (VMs) and Virtual Machine Scale Sets (VMSS)
> - Azure Kubernetes Service (AKS) container host, including the Azure Linux Container Host and Azure Container Linux (ACL)
> - Azure Linux container images published in Microsoft Container Registry (MCR), either under the azurelinux namespace or provided and supported by other Microsoft products or teams (for example, official .NET container based on Azure Linux).
> - ISO images
> - Customized images built on top of a prebuilt Azure Linux image, for example using ./customize-images.md
>
> **Unsupported scenarios and support exceptions**
>
> - Windows Subsystem for Linux (WSL): Azure Linux 4 on WSL receives best-effort support.
> - Bare metal deployments
> - On-premises deployments
>   - Support for Azure Linux in on-premises environments is limited. Microsoft support applies only to specific supported scenarios and platforms, such as Azure Local.
> - Deployments in other clouds
> - Images built from scratch from the https://github.com/microsoft/azurelinux
> - Self-built or third-party packages that aren't provided through official Microsoft Azure Linux repositories aren't covered by Microsoft support.

## Get support through Azure, GitHub, or security reporting

### Open an Azure support request for Azure Linux workloads

Requirement:
- An eligible [Azure support plan](https://azure.microsoft.com/support/plans/) that include technical support.

For technical support for an Azure Linux workload running on Azure, please [open an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request) through the Azure portal.

Use Azure Support when you need help investigating or resolving an issue affecting your Azure Linux workload, such as an outage, regression, configuration problem, deployment issue, or package and repository issue.

### Collect diagnostic information

When troubleshooting an Azure Linux system, you might be asked to collect an sos report. An sos report gathers diagnostic information from the system that can help Microsoft Support investigate and troubleshoot an issue. Before opening or while working on an Azure support request, you can generate an sos report and provide the resulting archive to Microsoft Support when requested.

- Install sos

```bash
sudo dnf install sos
```

- Generate an sos report

Run the following command with elevated privileges:
```bash
sudo sos -a report
```

Follow the prompts displayed by sos. The collection process creates a compressed diagnostic archive

- Locate the sos report archive

When the collection finishes, sos displays the path to the generated archive in its output.
You can also check the standard sos archive location:
```bash
sudo ls -lh /var/tmp/sosreport-*.tar.xz
```

- Provide the archive to Microsoft Support

If you're working with Microsoft Support, provide the generated archive as requested in your support case.

> [!IMPORTANT]
> An sos report can contain system configuration, logs, and other diagnostic information. Review and handle the archive appropriately before sharing it.


### Report bugs and request features on GitHub

**If you don't have an Azure support plan** that includes technical support, or **if you simply want to engage with the Azure Linux project** for bug reports, documentation issues, feature requests, and general questions. For more information, see [Report issues and request features for Azure Linux](./report-issues-request-features.md). All issues require a GitHub issue filed in the correct repository.

> [!IMPORTANT] 
> GitHub is **not monitored for production incidents** and does not provide guaranteed responses times. It should not be used to report issues requiring immediate assistance.

### Security vulnerabilities

**Please don't report security vulnerabilities through public GitHub issues**. Follow the guidance under [Manage CVEs](./manage-common-vulnerabilities-exposures.md#report-a-security-issue).

## Join our Azure Linux community calls

Azure Linux team hosts monthly community calls where you can get help from our product and support teams.

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

Azure Linux publishes a [feature roadmap](https://github.com/orgs/microsoft/projects/970/views/2) that provides visibility into upcoming features and changes. We review them in each community call and welcome your feedback and questions on individual feature items.

## Attributions

The Azure Linux penguin icon is inspired by the Linux mascot ("Tux"), created by Larry Ewing.
