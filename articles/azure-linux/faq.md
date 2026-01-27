---
  title: Frequently asked questions about the Azure Linux Container Host for AKS
  description: Find answers to some of the common questions about the Azure Linux Container Host for AKS.
  author: suhuruli
  ms.author: suhuruli
  ms.service: microsoft-linux
  ms.custom: linux-related-content
  ms.topic: faq
  ms.date: 12/12/2023
# Customer intent: "As a cloud engineer, I want to understand how to effectively use Azure Linux Container Host on AKS, so that I can optimize container workloads and manage my Kubernetes environments efficiently."
---

# Frequently asked questions about the Azure Linux Container Host for AKS

> [!CAUTION]
> This article references CentOS, a Linux distribution that is End Of Life (EOL) status. Please consider your use and planning accordingly. For more information, see the [CentOS End Of Life guidance](/azure/virtual-machines/workloads/centos/centos-end-of-life).

[!INCLUDE [azure-linux-retirement](./includes/azure-linux-retirement.md)]

This article answers common questions about the Azure Linux Container Host.

## General FAQs

### What is Azure Linux?

The Azure Linux Container Host is an operating system image that's optimized for running container workloads on [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes). Microsoft maintains the Azure Linux Container Host, which is based on Azure Linux (also known as *Mariner*), an open-source Linux distribution created by Microsoft.

### What are the benefits of using Azure Linux?

For more information, see the [Azure Linux Container Host key benefits](./intro-azure-linux.md#azure-linux-container-host-key-benefits).

### What's the difference between Azure Linux and Mariner?

Azure Linux and Mariner are the same image with different branding. Use the Azure Linux operating system SKU when referring to the image on AKS. To migrate from the Mariner OS SKU to the AzureLinux OS SKU, you can use the [In-Place OS SKU Migration feature](/azure/azure-linux/tutorial-azure-linux-migration). 

### Are Azure Linux container images supported on AKS?

Microsoft officially supports the Microsoft .NET and Open JDK container images based on Azure Linux. Community members provide best-effort support for all other images through the [GitHub issues page](https://github.com/microsoft/CBL-Mariner/issues).

### What's the pricing for Azure Linux?

Azure Linux is available at no additional cost. You only pay for the underlying Azure resources, such as virtual machines (VMs) and storage.

### What GPUs does Azure Linux support?

Azure Linux supports the V100, T4, and NC A100 V4 GPUs.

### What certifications does Azure Linux have?

Azure Linux passes all CIS level 1 benchmarks and offers a FIPS image. For more information, see [Azure Linux Container Host core concepts](./concepts-core.md).

### Is the Microsoft Azure Linux source code released?

Yes. Azure Linux is an open-source project with a thriving community of contributors. You can find the global Azure Linux source code at https://github.com/microsoft/CBL-Mariner.

### What is the Service Level Agreement (SLA) for CVEs?

Microsoft takes high and critical CVEs seriously and may release them out-of-band as a package update before a new AKS node image becomes available. Medium and low CVEs are included in the next scheduled image release.

For more information on CVEs, see [Azure Linux Container Host for AKS core concepts](./concepts-core.md#cve-infrastructure).

### How does Microsoft notify users of new Azure Linux versions?

You can track Azure Linux releases alongside AKS releases on the [AKS release tracker](/azure/aks/release-tracker).

### Does the Azure Linux Container Host support AppArmor?

No, the Azure Linux Container Host doesn't support AppArmor. Instead, it supports SELinux, which you can configure manually.

### How does Azure Linux read time for time synchronization on Azure?

For time synchronization, Azure Linux reads the time from the Azure VM host using [chronyd](/azure/virtual-machines/linux/time-sync#chrony) and the /dev/ptp device.

### How can I get help with Azure Linux?

Submit a [GitHub issue](https://github.com/microsoft/CBL-Mariner/issues/new/choose) to ask a question, provide feedback, or request a feature. For technical issues or bugs, create an [Azure support request](./support-help.md#create-an-azure-support-request).

### How can I stay informed of updates and new releases?

Microsoft hosts public community calls for Azure Linux users to discuss new features, provide feedback, and learn how others use Azure Linux. Each session features a new demo. The schedule for upcoming community calls is as follows:

| Date | Time | Meeting link |
| --- | --- | --- |
| 7/24/2025 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 9/25/2025 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 11/20/2025 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 1/22/2026 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |
| 3/26/2026 | 8-9 AM PST | [Click to join](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ZDcyZjRkYWMtOWQxYS00OTk3LWFhNmMtMTMwY2VhMTA4OTZi%40thread.v2/0?context=%7b%22Tid%22%3a%2272f988bf-86f1-41af-91ab-2d7cd011db47%22%2c%22Oid%22%3a%2271a6ce92-58a5-4ea0-96f4-bd4a0401370a%22%7d). |

## Cluster FAQs

### Is there a migration tool available to switch from a different distro to Azure Linux on Azure Kubernetes Service (AKS)?

Yes, migrating from another distribution to Azure Linux on AKS is straightforward. For more information, see [Tutorial 3 - Migrating to Azure Linux](./tutorial-azure-linux-migration.md).

### Can an existing AKS cluster be updated to use the Azure Linux Container Host, or does a new cluster with the Azure Linux Container Host need to be created?

You can add an Azure Linux node pool to an existing AKS cluster using the `az aks nodepool add` command with the `--os-sku AzureLinux` parameter. Once the node pool starts, it can coexist with another distribution, and Kubernetes schedules workloads across both node pools. For detailed instructions, see [Tutorial 2 - Add an Azure Linux node pool to your existing cluster](./tutorial-azure-linux-add-nodepool.md).

### Can I use a specific Azure Linux version indefinitely?

You can opt out of automatic node image upgrades and manually upgrade your node image to control which version of Azure Linux you use. This approach allows you to use a specific Azure Linux version for as long as needed.

### I added a new node pool on an AKS cluster using the Azure Linux Container Host, but the kernel version isn't the same as the one that booted. Is this expected behavior?

Yes, this is expected. The base image that AKS uses to start clusters typically runs about two weeks behind the latest packages. When AKS builds the image, the cluster boots with the latest kernel available at that time. However, one of the first actions the cluster performs is installing package updates, which includes a newer kernel version. Most updated packages take effect immediately, but a new kernel requires a node reboot to become active.

To enable automatic reboots, run a tool like [Kured](https://github.com/weaveworks/kured). Kured monitors each node and gracefully reboots the cluster one machine at a time to apply kernel updates.

## Update FAQs

### What is Azure Linux's release cycle?

Azure Linux releases major image versions approximately every two years. Each major version uses the Linux Long-Term Support (LTS) kernel and regularly updates stable packages. Microsoft also releases monthly updates with CVE fixes.

### How do upgrades from one major Azure Linux version to another work?

Upgrading between major Azure Linux versions requires a [SKU migration](./tutorial-azure-linux-migration.md). In the next major Azure Linux version release, the operating system SKU will support rolling releases.

### When are the latest Azure Linux Container Host image/node image released?

Microsoft builds new Azure Linux Container Host base images for AKS weekly, but releases occur less frequently. Each image undergoes a week of end-to-end testing before release. After testing completes, the image version takes a few days to roll out to all regions.

### Is it possible to skip multiple Azure Linux minor versions during an upgrade?

If you manually upgrade your node image instead of using automatic node image upgrades, you can skip Azure Linux minor versions. The next manual node image upgrade will update your nodes to the latest Azure Linux Container Host for AKS image.

### Some packages (CNCF, K8s) have a more aggressive release cycle, and I don't want to be up to a year behind. Does the Azure Linux Container Host have any plans for more frequent upgrades?

The Azure Linux Container Host adopts newer Cloud Native Computing Foundation (CNCF) packages like Kubernetes more frequently and doesn't delay them for annual releases. However, Microsoft reserves major compiler upgrades and deprecation of language stacks (such as Python 2.7x) for major releases.
