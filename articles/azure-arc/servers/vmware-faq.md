---
title: Azure Arc-enabled servers VMware vSphere FAQ
description: Learn how to use Azure Arc-enabled servers on virtual machines running in VMware vSphere environments.
ms.date: 02/12/2026
ms.topic: faq
# Customer intent: As a cloud architect, I want to evaluate the management capabilities of Azure Arc-enabled servers and Azure Arc-enabled VMware vSphere, so that I can choose the best solution for integrating on-premises VMware environments with Azure management tools.
---

# Azure Arc-enabled servers and VMware vSphere FAQ

This article helps you understand how [Azure Arc-enabled servers](cloud-native/overview.md) and [Azure Arc-enabled VMware vSphere](../vmware-vsphere/overview.md) can be used together or separately.

## What is Azure Arc?

Azure Arc is the overarching brand for a suite of Azure hybrid products that extend specific Azure public cloud services and management capabilities beyond Azure to on-premises environments and third-party clouds. Azure Arc-enabled servers, for example, allows you to use the same Azure management tools you use with a VM running in Azure with a VM running on-premises.

## What's the difference between Azure Arc-enabled servers and Azure Arc-enabled VMware vSphere?

The best way to understand the distinction is as follows:

- [Azure Arc-enabled servers](cloud-native/overview.md) interacts on the guest operating system level, with no awareness of the underlying infrastructure fabric and the virtualization platform that it runs on. Since Arc-enabled servers also support bare-metal machines, a host hypervisor might not exist in some cases.

- [Azure Arc-enabled VMware vSphere](../vmware-vsphere/overview.md) is a superset of Arc-enabled servers that extends management capabilities beyond the guest operating system to the VM itself. Azure Arc-enabled VMware vSphere provides lifecycle management and CRUD (Create, Read, Update, and Delete) operations on a VMware vSphere VM. You can perform these lifecycle management capabilities in the Azure portal or through APIs, just as you would with a native Azure VM. For more information, see [What is Azure Arc-enabled VMware vSphere?](../vmware-vsphere/overview.md)

> [!NOTE]
> [Azure Arc-enabled VMware vSphere](../vmware-vsphere/overview.md) supports vSphere environments anywhere, either on-premises as well as [Azure VMware Solution (AVS)](/azure/azure-vmware/deploy-arc-for-azure-vmware-solution), VMware Cloud on AWS, and Google Cloud VMware Engine.

## Can I use Azure Arc-enabled servers on VMs running in VMware environments?

Yes. Azure Arc-enabled servers work with VMs running in an on-premises VMware vSphere environment, as well as Azure VMware Solution (AVS). Arc-enabled servers supports the full breadth of guest management capabilities across security, monitoring, and governance.

## Which operating systems does Azure Arc-enabled servers work with?

Azure Arc-enabled servers and Azure Arc-enabled VMware vSphere work with [all supported versions](./prerequisites.md) of Windows Server and major distributions of Linux. As mentioned, even though Arc-enabled servers work with VMware vSphere virtual machines, the [Connected Machine agent](agent-overview.md) that provides the connection to Azure has no familiarity with the underlying infrastructure fabric and virtualization layer.

## Should I use Arc-enabled servers or Arc-enabled VMware vSphere for my VMware VMs?

Each option has its own unique benefits, and you can combine them as needed. By using Arc-enabled servers, you can manage the guest OS of your VMs by using the Azure Connected Machine agent. By using Arc-enabled VMware vSphere, you can onboard your VMware environment at scale to Azure Arc with automatic discovery. Arc-enabled VMware VSphere also lets you perform full VM lifecycle and virtual hardware operations directly on the machine. You have the flexibility to start with either option and incorporate the other one later without any disruption.
