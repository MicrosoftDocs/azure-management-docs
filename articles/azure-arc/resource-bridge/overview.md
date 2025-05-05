---
title: Azure Arc resource bridge overview
description: Learn how to use Azure Arc resource bridge to support VM self-servicing on Azure Local, VMware, and System Center Virtual Machine Manager.
ms.date: 04/22/2025
ms.topic: overview
ms.custom: references_regions
---

# What is Azure Arc resource bridge?

Azure Arc resource bridge is a Microsoft managed product that is part of the core Azure Arc platform. It is designed to host other Azure Arc services. In this release, the resource bridge supports VM self-servicing and management from Azure, for virtualized Windows and Linux virtual machines hosted in an on-premises environment on Azure Local ([Azure Arc VM management](/azure/azure-local/manage/azure-arc-vm-management-overview)), VMware ([Arc-enabled VMware vSphere](../vmware-vsphere/overview.md)), and System Center Virtual Machine Manager ([Arc-enabled SCVMM](../system-center-virtual-machine-manager/overview.md)).

Azure Arc resource bridge is a Kubernetes management cluster installed on the customer’s on-premises infrastructure as an appliance VM (also known as the Arc appliance). The resource bridge is provided credentials to the infrastructure control plane that allows it to apply guest management services on the on-premises resources. Arc resource bridge enables projection of on-premises resources as ARM resources and management from ARM as "Arc-enabled" Azure resources.

Arc resource bridge delivers the following benefits:

* Enables VM self-servicing from Azure without having to create and manage a Kubernetes cluster.
* Fully supported by Microsoft, including updates to core components.
* Supports deployment to any private cloud hosted on Hyper-V or VMware from the Azure portal or using the Azure Command-Line Interface (CLI).

## Overview

Azure Arc resource bridge hosts other components such as [custom locations](..\platform\conceptual-custom-locations.md), cluster extensions, and other Azure Arc agents in order to deliver the level of functionality with the private cloud infrastructures it supports. This complex system is composed of three layers:

* The base layer that represents the resource bridge and the Arc agents.
* The platform layer that includes the custom location and cluster extension.
* The solution layer for each service supported by Arc resource bridge (that is, the different type of VMs).

:::image type="content" source="media/overview/architecture-overview.png" alt-text="Azure Arc resource bridge architecture diagram." border="false" lightbox="media/overview/architecture-overview.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

Azure Arc resource bridge can host other Azure services or solutions running on-premises. There are two objects hosted on the Arc resource bridge:

* Cluster extension: The Azure service deployed to run on-premises. Currently, it supports three services:

  * Azure Arc VM management on Azure Local
  * Azure Arc-enabled VMware
  * Azure Arc-enabled System Center Virtual Machine Manager (SCVMM)

* Custom locations: A deployment target where you can create Azure resources. It maps to different resource for different Azure services. For example, for Arc-enabled VMware, the custom locations resource maps to an instance of vCenter, and for Azure Arc VM management on Azure Local, it maps to an Azure Local instance.

Custom locations and cluster extension are both Azure resources, which are linked to the Azure Arc resource bridge resource in Azure Resource Manager. When you create an on-premises VM from Azure, you can select the custom location, and that routes that *create action* to the mapped vCenter, Azure Local instance, or SCVMM.

Some resources are unique to the infrastructure. For example, vCenter has a resource pool, network, and template resources. During VM creation, these resources need to be specified. With Azure Local, you just need to select the custom location, network, and template to create a VM.

To summarize, the Azure resources are projections of the resources running in your on-premises private cloud. If the on-premises resource isn't healthy, it can impact the health of the related resources that are projected in Azure. For example, if the resource bridge is deleted by accident, all the resources projected in Azure by the resource bridge are impacted. The on-premises VMs in your on-premises private cloud aren't impacted, as they're running on vCenter, but you won't be able to start or stop the VMs from Azure. Directly managing or modifying the resource bridge using on-premises applications isn't recommended.

## Benefits of Azure Arc resource bridge

Through Azure Arc resource bridge, you can accomplish the following tasks for each private cloud infrastructure from Azure:

### Azure Local

You can provision and manage on-premises Windows and Linux virtual machines (VMs) running on Azure Local instances.

### VMware vSphere

By registering resource pools, networks, and VM templates, you can represent a subset of your vCenter resources in Azure to enable self-service. Integration with Azure allows you to manage access to your vCenter resources in Azure to maintain a secure environment. You can also perform various operations on the VMware virtual machines that are enabled by Arc-enabled VMware vSphere:

* Start, stop, and restart a virtual machine
* Control access and add Azure tags
* Add, remove, and update network interfaces
* Add, remove, and update disks and update VM size (CPU cores and memory)
* Enable guest management
* Install extensions

### System Center Virtual Machine Manager (SCVMM)

You can connect an SCVMM management server to Azure by deploying Azure Arc resource bridge in the VMM environment. Azure Arc resource bridge enables you to represent the SCVMM resources (clouds, VMs, templates etc.) in Azure and perform various operations on them:

* Start, stop, and restart a virtual machine
* Control access and add Azure tags
* Add, remove, and update network interfaces
* Add, remove, and update disks and update VM size (CPU cores and memory)
* Enable guest management
* Install extensions

## Example scenarios

The following are just two examples of the many scenarios that can be enabled by using Arc resource bridge in a hybrid environment.

### Apply Azure Policy and other Azure services to on-premises VMware VMs

A customer deploys Arc Resource Bridge onto their on-premises VMware environment. They sign into the Azure portal and select the VMware VMs that they'd like to connect to Azure. Now they can manage these on-premises VMware VMs in Azure Resource Manager (ARM) as Arc-enabled machines, alongside their native Azure machines, achieving a single pane of glass to view their resources in a VMware/Azure hybrid environment. This includes deploying Azure services, such as Defender for Cloud and Azure Policy, to keep updated on the security and compliance posture of their on-premises VMware VMs in Azure.

:::image type="content" source="../media/overview/resource-bridge-vmware.png" alt-text="Diagram showing VMware VMs connected to Azure through Arc resource bridge." lightbox="../media/overview/resource-bridge-vmware.png":::

### Create physical Azure Local VMs on-premises from Azure

A customer has multiple datacenter locations in Canada and New York. They install an Arc resource bridge in each datacenter and connect their Azure Local VMs to Azure in each location. They can then sign into Azure portal and see all their Arc-enabled VMs from the two physical locations together in one central cloud location. From the portal, the customer can choose to create a new VM; that VM is also created on-premises at the selected datacenter, allowing the customer to manage VMs in different physical locations centrally through Azure.

:::image type="content" source="../media/overview/resource-bridge-multi-datacenter.png" alt-text="Diagram showing Azure Local VMs in two datacenters connected to Azure through Arc resource bridge." lightbox="../media/overview/resource-bridge-multi-datacenter.png":::

## Version and region support

### Supported regions

In order to use Arc resource bridge in a region, Arc resource bridge and the Arc-enabled feature for a private cloud must be supported in the region. For example, to use Arc resource bridge with Azure Local in East US, Arc resource bridge and the Arc VM management feature for Azure Local must be supported in East US. To confirm feature availability across regions for each private cloud provider, review their deployment guide and other documentation. There could be instances where Arc resource bridge is available in a region where the private cloud feature isn't yet available.

Arc resource bridge supports the following Azure regions:

* East US
* East US 2
* West US 2
* West US 3
* Central US
* North Central US
* South Central US
* Canada Central
* Australia East
* Australia SouthEast
* West Europe
* North Europe
* UK South
* UK West
* Sweden Central
* Japan East
* Southeast Asia
* East Asia
* Central India

### Regional resiliency

While Azure has redundancy features at every level of failure, if a service impacting event occurs, Azure Arc resource bridge currently doesn't support cross-region failover or other resiliency capabilities. If the service becomes unavailable, on-premises VMs continue to operate unaffected. Management from Azure is unavailable during that service outage.

### Private cloud environments

The following private cloud environments and their versions are officially supported for Arc resource bridge:

* VMware vSphere version 7.0, 8.0
* Azure Local
* SCVMM

### Supported versions

Generally, the latest released version and the previous three versions (n-3) of Arc resource bridge are supported. For example, if the current version is 1.0.18, then the typical n-3 supported versions are:

* Current version: 1.0.18
* n-1 version: 1.0.17
* n-2 version: 1.0.16
* n-3 version: 1.0.15

There could be instances where supported versions aren't sequential. For example, version 1.0.18 is released and later found to contain a bug; a hot fix is released in version 1.0.19 and version 1.0.18 is removed. In this scenario, n-3 supported versions become 1.0.19, 1.0.17, 1.0.16, 1.0.15.

Arc resource bridge typically releases a new version on a monthly cadence, at the end of the month. Delays might occur that could push the release date further out. Regardless of when a new release comes out, if you are within n-3 supported versions, then your Arc resource bridge version is supported. To stay updated on releases, visit the [Arc resource bridge release notes](release-notes.md). To learn more about upgrade options, visit [Upgrade Arc resource bridge](upgrade.md).

### Private link support

Arc resource bridge currently doesn't support private link.

## Next steps

* Learn how [Azure Arc-enabled VMware vSphere extends Azure's governance and management capabilities to VMware vSphere infrastructure](../vmware-vsphere/overview.md).
* Learn how [Azure Arc-enabled SCVMM extends Azure's governance and management capabilities to System Center managed infrastructure](../system-center-virtual-machine-manager/overview.md).
* Learn about [provisioning and managing on-premises Windows and Linux VMs running on Azure Local instances](/azure/azure-local/manage/azure-arc-vm-management-overview).
* Review the [system requirements](system-requirements.md) for deploying and managing Arc resource bridge.
