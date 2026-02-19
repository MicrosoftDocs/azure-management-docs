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

The Azure Linux Container Host for AKS is based on the Microsoft Azure Linux distribution, which supports thousands of packages. The container host contains a subset of those packages based on operating system and Kubernetes requirements. This curated package set includes the most requested and necessary packages for running container workloads, selected based on feedback from customers and the open-source community.

[!INCLUDE [azure-linux-retirement](./includes/azure-linux-retirement.md)]

## List of Azure Linux Container Host packages

The Azure Linux Container Host package list includes all the needed dependencies to run an Azure Linux VM and also includes any necessary Azure Kubernetes Service dependencies. You can view the complete list of packages in the Azure Linux Container Host [here](https://github.com/Azure/AgentBaker/blob/master/vhdbuilder/release-notes/AKSCBLMariner/gen2/latest.txt).

Whenever AKS releases a new image, the [AKS Azure Linux release notes folder](https://github.com/Azure/AgentBaker/blob/master/vhdbuilder/release-notes/AKSAzureLinux/gen2/latest.txt) updates with a new `latest.txt` file. This file details the most up-to-date package list. You can also view previous image package lists and historical versions of each package in the most recent image release in the GitHub repository. For each prior image release, there's a corresponding `.txt` file with the naming convention `YYYY.MM.DD.txt`, where `YYYY.MM.DD` is the date of that image release. 


> [!NOTE]
> Packages on a running Azure Linux Container Host cluster may have been automatically updated to their latest versions as Microsoft releases new packages on [packages.microsoft.com](https://packages.microsoft.com/).

One of the key benefits of the Azure Linux Container Host package set is the kernel package. Microsoft updates and patches the Linux kernel package for the Azure Linux Container Host at least twice a month. A dedicated Microsoft team manages and owns this package, ensuring it's secure and contains all the latest development updates.

## Determine package versions in a cluster 

If you have direct access to the container host, you can query packages directly from the host. 

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
