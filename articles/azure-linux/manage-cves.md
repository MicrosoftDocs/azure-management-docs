---
title: Manage CVEs on Azure Linux and Azure Container Linux (ACL)
description: Learn how Azure Linux and Azure Container Linux (ACL) handle Common Vulnerabilities and Exposures (CVEs), including the dedicated CVE pipeline, published advisories, and service level agreements (SLAs) for patch delivery.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Manage CVEs on Azure Linux and Azure Container Linux (ACL)

Azure Linux and Azure Container Linux (ACL) share a dedicated Common Vulnerabilities and Exposures (CVE) pipeline, published advisories, and defined service level agreements (SLAs) so you can manage vulnerabilities across your fleet with confidence. This article explains how CVEs are identified, patched, and delivered to your systems depending on which Azure Linux or ACL deployment option you use.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Update and verify CVE fixes

1. Apply the latest security updates for your [deployment option](#cve-delivery-by-deployment-option).
1. Confirm updated package versions, changelogs, or node image versions.

## CVE infrastructure and SLAs

Microsoft is responsible for the full Azure Linux and Azure Container Linux (ACL) stack - from the Linux kernel to the CVE infrastructure, support, and end-to-end validation. This means you don't need to track and patch vulnerabilities from a third-party distribution. Azure Linux handles identifying applicable CVEs, publishing fixes, and meeting SLAs for production package fixes.

The Azure Linux team scans the packages it ships for security vulnerabilities **twice a day** against the [National Vulnerability Database (NVD)](https://nvd.nist.gov/). When a vulnerability is confirmed, the Azure Linux team collaborates with the [Microsoft Security Response Center (MSRC)](https://www.microsoft.com/msrc) to evaluate, patch, and contribute fixes back upstream.

### CVE delivery by deployment option

CVE fixes are delivered differently depending on whether you run general-purpose Azure Linux, Azure Linux Container Host for AKS, or Azure Container Linux (ACL) for AKS. The following table summarizes how CVE fixes are propagated to each deployment option:

| Deployment option | CVE fix delivery mechanism |
| ----------------- | -------------------------- |
| **General-purpose Azure Linux (Virtual Machines (VM), Virtual Machine Scale Sets, custom images)** | CVE fixes are delivered as package updates. Apply them with `dnf update`. |
| **Azure Linux Container Host for AKS** | CVE fixes are delivered as package updates bundled into monthly node image releases. **High and critical CVEs** might be released out-of-band as a package update before the next scheduled node image, so a fix can reach your nodes ahead of a new image. **Medium and low CVEs** are rolled into the next regular node image release. |
| **Azure Container Linux (ACL) for AKS** | ACL is an immutable, image-based operating system (OS); individual packages aren't updated in place. Instead, CVE fixes are delivered through weekly AKS node image releases that include the latest security patches. The `SecurityPatch` node OS upgrade channel isn't supported on ACL, so use the `NodeImage` channel to pick up security updates. For ACL details, see [Azure Container Linux overview](./azure-container-linux-overview.md). |

### Published advisories

> [!NOTE]
> Azure Linux 4.0 CVE SLAs aren't applicable during preview.

Azure Linux and Azure Container Linux (ACL) security advisories are being published in **Vulnerability Exploitability eXchange (VEX)** format through the Microsoft Security Response Center (MSRC). VEX advisories help you determine whether a vulnerability actually affects your specific configuration rather than just whether a package is installed.

Azure Linux and ACL CVEs are also published through the Microsoft [Security Update Guide (SUG) CVRF API](https://github.com/microsoft/MSRC-Microsoft-Security-Updates-API), so you can ingest Microsoft security updates programmatically alongside other Microsoft product advisories.

## Apply security updates

Keep your system updated to receive security fixes. The right mechanism depends on your [deployment option](#cve-delivery-by-deployment-option).

On general-purpose Azure Linux, apply security updates by updating packages with `dnf`:

```bash
sudo dnf update -y
```

On Azure Container Linux, use Azure Kubernetes Service (AKS) node image upgrades on the `NodeImage` channel; don't attempt to update individual packages on the immutable OS.

## Validate fixes

On general-purpose Azure Linux, verify package versions as needed:

```bash
dnf info <PACKAGE_NAME>
```

On Azure Container Linux, verify the running node image version (for example, with `az aks nodepool list --query '[].nodeImageVersion'`) matches the expected release.

## Coordinate with vulnerability scanners

Azure Linux and Azure Container Linux support common vulnerability scanning tools. For a list of supported scanners, see [Azure Linux partner solutions](./ecosystem-support.md).

Once VEX advisories are published for Azure Linux and ACL, scanners that consume VEX can accurately report whether an installed package is actually exposed to a given CVE on these platforms, reducing false positives.

## Report a security issue

Report suspected security vulnerabilities in Azure Linux or Azure Container Linux to the [Microsoft Security Response Center (MSRC)](https://www.microsoft.com/msrc). Don't file security issues on the public issue tracker.

## Related content

- [Overview of Azure Linux security](./security-overview.md)
