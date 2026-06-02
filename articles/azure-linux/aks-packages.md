---
title: Overview of Azure Linux Container Host Packages for Azure Kubernetes Service (AKS)
description: TLearn about the packages included in the Azure Linux Container Host for Azure Kubernetes Service (AKS), how to view the package list for a given node image version, and how to determine the versions of packages installed in a running cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: overview
ms.date: 04/28/2026
---

# Azure Linux Container Host packages for Azure Kubernetes Service (AKS)

The Azure Linux Container Host for Azure Kubernetes Service (AKS) is based on the Microsoft Azure Linux distribution, which supports thousands of packages. The container host contains a subset of those packages based on customer operating system (OS) and Kubernetes needs. This set of curated packages is among the most requested and necessary packages to run container workloads based on feedback from customers and the open-source community.

This article provides an overview of the packages included in the Azure Linux Container Host for AKS, how to view the package list for a given node image version, and how to determine the versions of packages installed in a running cluster.

## Azure Linux Container Host package list

The Azure Linux Container Host package list includes all the needed dependencies to run an Azure Linux virtual machine (VM) and also pulls in any necessary AKS dependencies. To view the full package list for the latest Azure Linux Container Host image released by AKS, see [/release-notes/AKSAzureLinux/gen2/latest.txt](https://github.com/Azure/AgentBaker/blob/main/vhdbuilder/release-notes/AKSAzureLinux/gen2/latest.txt).

Whenever AKS releases a new Azure Linux Container Host image, we update the [AKS Azure Linux release notes folder](https://github.com/Azure/AgentBaker/blob/main/vhdbuilder/release-notes/AKSAzureLinux/gen2/latest.txt) with a new `latest.txt` file, which details the most up-to-date package list. You can also view previous image package lists and the historical versions of each package in the most recent image release in the GitHub repository. For each prior image release, you can find a corresponding `.txt` file with the naming convention `YYYY.MM.DD.txt`, where `YYYY.MM.DD` is the date of each previous image release.

> [!NOTE]
> Packages on a running Azure Linux Container Host cluster might be automatically updated to their latest versions as new packages are released on [packages.microsoft.com](https://packages.microsoft.com/).

### Kernel package

One of the key benefits of the Azure Linux Container Host package set is the kernel package. We patch and update the Linux kernel package for the Azure Linux Container Host at least twice a month. This package is managed and owned by an entire Microsoft team, which ensures it's secure and contains all the latest updates for development.

## Find cluster package versions

If you have direct access to the container host, you can query packages from the host itself using the commands in [List all installed packages](#list-all-installed-packages) and [Check package installation dates](#check-package-installation-dates). If you don't have direct access to the container host, you can [work backwards from the node image version date to determine the package versions in a cluster](#determine-package-versions-from-node-image-version).

### List all installed packages

List all the installed packages and their versions using the following command:

```console
rpm -qa
```

### Check package installation dates

Determine when individual packages were installed using the following command:

```console
cat /var/log/dnf.log
```

### Determine package versions from node image version

1. Get the node image version for your cluster using the [`az aks show`](/cli/azure/aks#az-aks-show) command. Replace the `<resource-group-name>` and `<cluster-name>` placeholders with your own values.

    ```azurecli-interactive
    az aks show --resource-group <resource-group-name> --name <cluster-name> | grep nodeImageVersion
    ```

1. Check the [AKS Azure Linux release notes folder](https://github.com/Azure/AgentBaker/blob/master/vhdbuilder/release-notes/AKSAzureLinux/gen2) for the file that corresponds with the previously determined node image version date. In the file, the _Installed Packages Begin_ section lists all the package versions in your cluster.

## Related content

This article covers some of the core Azure Linux Container Host components such as packages. For more information on the Azure Linux Container Host concepts, see the following articles:

- [Azure Linux Container Host overview](./azure-linux-overview.md)
- [Azure Linux Container Host for AKS core concepts](./aks-core-concepts.md)
