---
title:  Overview of the Azure Arc-enabled System Center Virtual Machine Manager 
description: This article provides a detailed overview of the Azure Arc-enabled System Center Virtual Machine Manager.
ms.date: 03/25/2025
ms.topic: overview
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
author: PriskeyJeronika-MS
ms.author: v-gjeronika
manager: jsuri
keywords: "VMM, Arc, Azure, System Center"
ms.custom: references_regions
---

# Overview of Azure Arc-enabled System Center Virtual Machine Manager

Azure Arc-enabled System Center Virtual Machine Manager (SCVMM) empowers System Center customers to connect their VMM environment to Azure and perform VM self-service operations from Azure portal. Azure Arc-enabled SCVMM extends the Azure control plane to SCVMM managed infrastructure, enabling the use of Azure security, governance, and management capabilities consistently across System Center managed estate and Azure.

Azure Arc-enabled SCVMM also allows you to manage your hybrid environment consistently and perform self-service VM operations through Azure portal. For Microsoft Azure Pack customers, this solution is intended as an alternative to perform VM self-service operations.

Azure Arc-enabled SCVMM allows you to:

- Perform various VM lifecycle operations such as start, stop, pause, and delete VMs on SCVMM managed VMs directly from Azure.
- Empower developers and application teams to self-serve VM operations on demand using [Azure role-based access control (RBAC)](/azure/role-based-access-control/overview).
- Browse your VMM resources (VMs, templates, VM networks, and storage) in Azure, providing you with a single pane view for your infrastructure across both environments.
- Discover and onboard existing SCVMM managed VMs to Azure.
- Install the Azure Connected Machine agent at scale on SCVMM VMs to [govern, protect, configure, and monitor them](../servers/overview.md#supported-cloud-operations).
- Build automation and self-service pipelines using Python, Java, JavaScript, Go, and .NET SDKs; Terraform, ARM, and Bicep templates; Azure REST APIs, CLI, and PowerShell.
- Leverage Azure Arc benefits such as [Windows Server management](/azure/azure-arc/servers/windows-server-management-overview?tabs=portal) for VMs with Software Assurance licenses, and pay-as-you-go billing for [Extended Security Updates](/azure/azure-arc/system-center-virtual-machine-manager/deliver-esus-for-system-center-virtual-machine-manager-vms) for Windows Server 2012/R2 VMs. 

> [!NOTE]
> For more information regarding the different services Azure Arc offers, see [Choosing the right Azure Arc service for machines](../choose-service.md).

## Onboard resources to Azure management at scale

Azure services such as Microsoft Defender for Cloud, Azure Monitor, Azure Update Manager, and Azure Policy provide a rich set of capabilities to secure, monitor, patch, and govern off-Azure resources via Arc.

By using Azure Arc-enabled SCVMM's capabilities to discover your SCVMM managed estate and install the Azure Connected Machine agent at scale, you can simplify onboarding your entire System Center estate to these services.

## How does it work?

To Arc-enable an SCVMM management server, deploy [Azure Arc resource bridge](../resource-bridge/overview.md) in the VMM environment. Azure Arc resource bridge is a virtual appliance that connects VMM management server to Azure. Azure Arc resource bridge enables you to represent the SCVMM resources (clouds, VMs, templates etc.) in Azure and do various operations on them.

## Architecture

The following image shows the architecture for the Azure Arc-enabled SCVMM:

:::image type="content" source="media/architecture/azure-arc-scvmm-architecture.png" alt-text="Screenshot of Arc enabled SCVMM - architecture." lightbox="media/architecture/azure-arc-scvmm-architecture.png":::

>[!NOTE]
> The above architecture diagram was created as part of Arc Jumpstart. To download it's source file in a high-resolution format, visit [Jumpstart Gems](https://aka.ms/jumpstartgems).

## How is Azure Arc-enabled SCVMM different from Azure Arc-enabled servers

- Azure Arc-enabled servers interact on the guest operating system level, with no awareness of the underlying infrastructure fabric and the virtualization platform that they're running on. Since Azure Arc-enabled servers also support bare-metal machines, there might, in fact, not even be a host hypervisor in some cases.
- Azure Arc-enabled SCVMM is a superset of Azure Arc-enabled servers that extends management capabilities beyond the guest operating system to the VM itself. This provides lifecycle management and CRUD (Create, Read, Update, and Delete) operations on an SCVMM VM. These lifecycle management capabilities are exposed in the Azure portal and look and feel just like a regular Azure VM. Azure Arc-enabled SCVMM also provides guest operating system management, in fact, it uses the same components as Azure Arc-enabled servers.

You have the flexibility to start with either option, and incorporate the other one later without any disruption. With both options, you'll enjoy the same consistent experience.

### Supported scenarios

The following scenarios are supported in Azure Arc-enabled SCVMM:

- SCVMM administrators can connect a VMM instance to Azure and browse the SCVMM virtual machine inventory in Azure.
- Administrators can use the Azure portal to browse SCVMM inventory and register SCVMM cloud, virtual machines, VM networks, and VM templates into Azure.
- Administrators can provide app teams/developers fine-grained permissions on those SCVMM resources through Azure RBAC.
- App teams can use Azure interfaces (portal, CLI, or REST API) to manage the lifecycle of on-premises VMs they use for deploying their applications (CRUD, Start/Stop/Restart).
- Administrators can install Azure Connected Machine agents on SCVMM VMs at-scale and install corresponding extensions to use Azure management services like Microsoft Defender for Cloud, Azure Update Manager, Azure Monitor, etc.

### Unsupported scenarios

Azure Arc-enabled SCVMM doesn't support:

- Azure-based management of VMware vCenter VMs managed by SCVMM. To onboard VMware VMs to Azure Arc, we recommend you to use [Azure Arc-enabled VMware vSphere](/azure/azure-arc/vmware-vsphere/overview).
- Azure-based management of Azure Local VMs managed by SCVMM. To onboard Azure Local VMs to Azure Arc, we recommend you to use [Azure Arc VM management capabilities of Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview).

### Supported VMM versions

Azure Arc-enabled SCVMM works with VMM 2025, 2022, and 2019 versions and supports SCVMM management servers with a maximum of 15,000 VMs.

### Supported regions

For the most up-to-date information about regional availability of Azure Arc-enabled SCVMM, see [Product Availability by Region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table) page.

## Data Residency

Azure Arc-enabled SCVMM doesn't store/process customer data outside the region the customer deploys the service instance in.

## Next steps

- Plan your Azure Arc-enabled SCVMM deployment by reviewing the [support matrix](support-matrix-for-system-center-virtual-machine-manager.md).
- Once ready, [connect your SCVMM management server to Azure Arc using the onboarding script](quickstart-connect-system-center-virtual-machine-manager-to-arc.md).

### CAF

- [Hybrid and multicloud migration](/azure/cloud-adoption-framework/scenarios/hybrid/migrate).

### WAF

- [Operational Excellence design principles](/azure/well-architected/operational-excellence/principles).
