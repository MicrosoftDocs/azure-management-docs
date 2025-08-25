---
title: Azure Linux Container Host for AKS packages
description: Learn about the packages supported by the Azure Linux Container Host for AKS.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.topic: concept-article
ms.date: 08/18/2024
ms.custom: template-concept, linux-related-content
# Customer intent: "As a DevOps engineer, I want to access and manage the package list for the Azure Linux Container Host in AKS, so that I can ensure my Kubernetes workloads are using the latest and most secure versions of required packages."
---

# Packages

The Azure Linux Container Host for AKS is based on the Microsoft Azure Linux distribution, which supports thousands of packages. The container host contains a subset of those packages based on our customers' operating system and Kubernetes needs. This set of curated packages is among the most requested and necessary packages to run container workloads based on feedback from customers and the open-source community.

> [!IMPORTANT]
> Starting on **30 November 2025**, AKS will no longer support or provide security updates for Azure Linux 2.0. Starting on **31 March 2026**, existing node images will be deleted, and you'll be unable to scale your node pools. Migrate to a supported Azure Linux version by [**upgrading your node pools**](/azure/aks/upgrade-aks-cluster) to a supported Kubernetes version or migrating to [`osSku AzureLinux3`](/azure/aks/upgrade-os-version). For more information, see [[Retirement] Azure Linux 2.0 node pools on AKS](https://github.com/Azure/AKS/issues/4988).

## List of Azure Linux Container Host packages

The Azure Linux Container Host package list includes all the needed dependencies to run an Azure Linux VM and also pulls in any necessary Azure Kubernetes Service dependencies. A list of all the packages in the Azure Linux Container Host can be viewed [here](https://github.com/Azure/AgentBaker/blob/master/vhdbuilder/release-notes/AKSCBLMariner/gen2/latest.txt).

Whenever a new image is released by AKS, the [AKS Azure Linux release notes folder](https://github.com/Azure/AgentBaker/blob/master/vhdbuilder/release-notes/AKSAzureLinux/gen2/latest.txt) is updated with a new `latest.txt` file, which details the most up-to-date package list. You can also view previous image package lists and the historical versions of each package in the most recent image release in the GitHub repository. For each prior image release, you can find a corresponding `.txt` file with the naming convention `YYYY.MM.DD.txt`, where `YYYY.MM.DD` is the date of each previous image release. 


> [!NOTE]
> Packages on a running Azure Linux Container Host cluster may have been automatically updated to their latest versions as new packages are released on [packages.microsoft.com](https://packages.microsoft.com/).

One of the key benefits of the Azure Linux Container Host package set is the kernel package. The Linux kernel package for the Azure Linux Container Host is patched and updated at least twice a month. This package is managed and owned by an entire Microsoft team, which ensures it's secure and contains all the latest updates for development.

## Determining package versions in a cluster 

If you have direct access to the container host, you can query packages from the host itself. 

To list all the installed packages and their versions, run the following command: 

```console
rpm -qa
```

To determine when individual packages were installed, run the following command: 

```console
cat /var/log/dnf.log
```

If you don't have direct access to the container host, you can work backwards from the node image version date to determine the package versions in a cluster.

To determine the `nodeImageVersion`, run the following command: 

```azurecli
az aks show -g <groupname> -n <clustername> | grep nodeImageVersion
```

Then, as described above, check the [AKS Azure Linux release notes folder](https://github.com/Azure/AgentBaker/blob/master/vhdbuilder/release-notes/AKSAzureLinux/gen2) for the file that corresponds with the previously determined node image version date. In the file, the *Installed Packages Begin* section lists all the package versions in your cluster.


## Next steps

This article covers some of the core Azure Linux Container Host components such as packages. For more information on the Azure Linux Container Host concepts, see the following articles:

- [Azure Linux Container Host overview](./intro-azure-linux.md)
- [Azure Linux Container Host for AKS core concepts](./concepts-core.md)
