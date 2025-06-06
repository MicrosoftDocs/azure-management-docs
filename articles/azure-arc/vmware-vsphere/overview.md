---
title: What is Azure Arc-enabled VMware vSphere?
description: Azure Arc-enabled VMware vSphere extends Azure governance and management capabilities to VMware vSphere infrastructure and delivers a consistent management experience across both platforms. 
ms.topic: overview
ms.date: 04/17/2025
ms.custom:
  - references_regions
  - build-2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
---

# What is Azure Arc-enabled VMware vSphere?

Azure Arc-enabled VMware vSphere is an [Azure Arc](../overview.md) service that helps you simplify management of hybrid IT estate distributed across VMware vSphere and Azure. It does so by extending the Azure control plane to VMware vSphere infrastructure and enabling the use of Azure security, governance, and management capabilities consistently across VMware vSphere and Azure. 

Arc-enabled VMware vSphere allows you to:

- Discover your VMware vSphere estate (VMs, templates, networks, datastores, clusters/hosts/resource pools) and register resources with Arc at scale. 

- Perform various virtual machine (VM) operations directly from Azure, such as create, resize, delete, and power cycle operations such as start/stop/restart on VMware VMs consistently with Azure. 

- Empower developers and application teams to self-serve VM operations on-demand using [Azure role-based access control](/azure/role-based-access-control/overview) (RBAC).

- Install the Azure connected machine agent at scale on VMware VMs to [govern, protect, configure, and monitor](../servers/overview.md#supported-cloud-operations) them.

- Browse your VMware vSphere resources (VMs, templates, networks, and storage) in Azure, providing you with a single pane view for your infrastructure across both environments.

> [!NOTE]
>For guidance on choosing the right Azure Arc service for your virtual machines, see [Choose the right Azure Arc service for machines](../choose-service.md).

## Onboard resources to Azure management at scale

Azure services such as Microsoft Defender for Cloud, Azure Monitor, Azure Update Manager, and Azure Policy provide a rich set of capabilities to secure, monitor, patch, and govern off-Azure resources via Arc. 

By using Arc-enabled VMware vSphere's capabilities to discover your VMware estate and install the Arc agent at scale, you can simplify onboarding your entire VMware vSphere estate to these services. 

## Set up self-service access for your teams to use vSphere resources using Azure Arc

Arc-enabled VMware vSphere extends Azure's control plane (Azure Resource Manager) to VMware vSphere infrastructure. This enables you to use Microsoft Entra ID-based identity management, granular Azure RBAC, and Azure Resource Manager (ARM) templates to help your app teams and developers get self-service access to provision and manage VMs on VMware vSphere environment, providing greater agility. 

1. Virtualized Infrastructure Administrators/Cloud Administrators can connect a vCenter instance to Azure. 

2. Administrators can then use the Azure portal to browse VMware vSphere inventory and register virtual machines resource pools, networks, and templates into Azure. 

3. Administrators can provide app teams/developers fine-grained permissions on those VMware resources through Azure RBAC.

4. App teams can use Azure interfaces (portal, CLI, or REST API) to manage the lifecycle of on-premises VMs they use for deploying their applications (CRUD, Start/Stop/Restart).

5. App teams can use Azure Resource Manager (ARM) templates/Bicep (Infrastructure as Code) to deploy VMs as part of CI/CD pipelines. 

## How does it work?

Arc-enabled VMware vSphere provides these capabilities by integrating with your VMware vCenter Server. To connect your VMware vCenter Server to Azure Arc, you need to deploy the [Azure Arc resource bridge](../resource-bridge/overview.md) in your vSphere environment. Azure Arc resource bridge is a virtual appliance that hosts the components that communicate with your vCenter Server and Azure. 

When a VMware vCenter Server is connected to Azure, an automatic discovery of the inventory of vSphere resources is performed. This inventory data is continuously kept in sync with the vCenter Server. 

All guest OS-based capabilities are provided by enabling guest management (installing the Arc agent) on the VMs. Once guest management is enabled, VM extensions can be installed to use the Azure management capabilities. You can perform virtual hardware operations such as resizing, deleting, adding disks, and power cycling without guest management enabled. 

## How is Arc-enabled VMware vSphere different from Arc-enabled Servers

The easiest way to think of this is as follows:

- Azure Arc-enabled servers interact on the guest operating system level, with no awareness of the underlying infrastructure fabric and the virtualization platform that they're running on. Since Arc-enabled servers also support bare-metal machines, there can, in fact, not even be a host hypervisor in some cases.

- Azure Arc-enabled VMware vSphere is a superset of Arc-enabled servers that extends management capabilities beyond the guest operating system to the VM itself. This provides lifecycle management and CRUD (Create, Read, Update, and Delete) operations on a VMware vSphere VM. These lifecycle management capabilities are exposed in the Azure portal and look and feel just like a regular Azure VM. Azure Arc-enabled VMware vSphere also provides guest operating system management—in fact, it uses the same components as Azure Arc-enabled servers. 

You have the flexibility to start with either option, and incorporate the other one later without any disruption. With both the options, you enjoy the same consistent experience.

## Supported VMware vSphere versions

Azure Arc-enabled VMware vSphere currently works with vCenter Server versions 7 and 8.

> [!NOTE]
> Azure Arc-enabled VMware vSphere supports vCenters with a maximum of 9500 VMs. If your vCenter has more than 9500 VMs, we don't recommend you to use Arc-enabled VMware vSphere with it at this point.

If you're trying to enable Arc for Azure VMware Solution (AVS) private cloud, see [Deploy Arc-enabled VMware vSphere for Azure VMware Solution private cloud](/azure/azure-vmware/deploy-arc-for-azure-vmware-solution).

## Supported regions

For the most up-to-date information about region availability of Azure Arc-enabled VMware vSphere, see [Azure Products by Region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table) page.

## Data Residency

Azure Arc-enabled VMware vSphere doesn't store/process customer data outside the region the customer deploys the service instance in.  

## Azure Kubernetes Service (AKS) Arc on VMware (preview)

Starting March 2024, Azure Kubernetes Service (AKS) enabled by Azure Arc on VMware is available for preview. AKS Arc on VMware enables you to use Azure Arc to create new Kubernetes clusters on VMware vSphere. For more information, see [What is AKS enabled by Arc on VMware?](/azure/aks/hybrid/aks-vmware-overview).

The following capabilities are available in the AKS Arc on VMware preview:

- **Simplified infrastructure deployment on Arc-enabled VMware vSphere**: Onboard VMware vSphere to Azure using a single-step process with the AKS Arc extension installed.
- **Azure CLI**: A consistent command-line experience, with [AKS Arc on Azure Local 23H2](/azure/aks/hybrid/aks-create-clusters-cli), for creating and managing Kubernetes clusters. Note that the preview only supports a limited set of commands.
- **Cloud-based management**: Use familiar tools such as Azure CLI to create and manage Kubernetes clusters on VMware.
- **Support for managing and scaling node pools and clusters**.

## Next steps

- Plan your resource bridge deployment by reviewing the [support matrix for Arc-enabled VMware vSphere](support-matrix-for-arc-enabled-vmware-vsphere.md).
- Once ready, [connect VMware vCenter to Azure Arc using the helper script](quick-start-connect-vcenter-to-arc-using-script.md).
- Try out Arc-enabled VMware vSphere by using the [Azure Arc Jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_vsphere).
- [Consider unified operations and plan for hybrid and multicloud environments with the Cloud Adoption Framework](/azure/cloud-adoption-framework/scenarios/hybrid/plan).
- [Choose the Azure Hybrid solution that meets your business requirements with guidance from the Azure Architecture Center](/azure/architecture/guide/technology-choices/hybrid-considerations).