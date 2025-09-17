---
ms.assetid:
title: What's new in Azure Arc-enabled VMware vSphere
description: This article describes the new features and enhancements supported in Azure Arc-enabled VMware vSphere.
ms.author: jsuri
author: jyothisuri
ms.date: 08/07/2025
ms.topic: article
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.custom: references_regions
---

# What's new in Azure Arc-enabled VMware vSphere

Azure Arc-enabled VMware vSphere receives new features and enhancements on an ongoing basis. This article provides an overview of the recent developments, which are common to both on-premises VMware and Azure VMware Solution private clouds. 

## August 2025 

- Retirement notice: Azure Kubernetes Service on VMware (Preview) is retiring on 16th March, 2026.

## August 2025 

- Rollout to Italy North region.

## June 2025 

- Ability to customize guest operating system settings while creating Windows virtual machines. 

## May 2025 

- Installation of the Arc agent on Linux machines using SSH-based authentication. 

## January 2025 

- [Windows Server Management](/azure/azure-arc/servers/windows-server-management-overview) for VMware VMs with active Windows Server Software Assurance licenses, giving you cost benefits on Azure Update Manager, Azure Change Tracking, and Inventory billing and exclusive capabilities. 

## October 2024

- Ability to install Arc agents at-scale on VMware VMs using out-of-band methods such as:
   - Service principal
   - System Center Configuration Manager script
   - System Center Configuration Manager custom task sequence
   - Group policy
   - Ansible playbook

## September 2024 

- [Terraform templates](/azure/templates/microsoft.connectedvmwarevsphere/clusters?pivots=deployment-language-terraform) based VM management (AzAPI provider) 
- [Bicep templates](/azure/templates/microsoft.connectedvmwarevsphere/clusters?pivots=deployment-language-bicep) based VM management 
- [ARM templates](/azure/templates/microsoft.connectedvmwarevsphere/clusters?pivots=deployment-language-arm-template) based VM management 

## August 2024 

- [Python SDK](/python/api/overview/azure/mgmt-connectedvmware-readme) based VM management 
- [Java SDK](/java/api/overview/azure/resourcemanager-connectedvmware-readme) based VM management 
- [JavaScript SDK](/javascript/api/overview/azure/arm-connectedvmware-readme) based VM management 
- [.NET SDK](/dotnet/api/overview/azure/resourcemanager.connectedvmwarevsphere-readme) based VM management 
- [Go SDK](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/connectedvmware/armconnectedvmware#section-documentation) based VM management 

## June 2024 

- [Azure-CLI](/cli/azure/connectedvmware) based VM management 
- [Azure REST APIs](/rest/api/azure-arc-vmware/operation-groups) based VM management 
- [Azure PowerShell](/powershell/module/az.connectedvmware) based VM management 

## April 2024 

- Support to install Arc agent over private endpoints on VMware VMs at-scale. 

## March 2024 

- Public preview of [Azure Kubernetes Service on VMware](/azure/aks/hybrid/aks-vmware-overview). 

## February 2024 

- Rollout to five new regions: 
     - East Asia 
     - Japan East 
     - North Central US 
     - UK West 
     - Central India   

## November 2023 (General Availability) 

- Ability to [link VMs](/azure/azure-arc/vmware-vsphere/enable-virtual-hardware) with Arc agent installed to leverage self-service lifecycle and power cycle management. 
- Procurement and delivery of [Extended Security Updates](/azure/azure-arc/vmware-vsphere/deliver-extended-security-updates-for-vmware-vms-through-arc) for Windows Server and SQL server Arc-enabled VMware/AVS VMs. 
- Microsoft-managed upgrade support for Azure Arc resource bridge.

## September 2023 

- Installation of the Arc agent [at scale](/azure/azure-arc/vmware-vsphere/enable-guest-management-at-scale) on VMware on-premises and Azure VMware Solution private cloud VMs.
- Support for Azure services to [govern, protect, configure, and monitor](/azure/azure-arc/servers/overview#supported-cloud-operations) Arc-enabled VMware/AVS VMs. 
- Ability to [recover](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion) the Azure Arc resource bridge in case of accidental deletion. 

## August 2023 

- Manual [upgrade support](/azure/azure-arc/vmware-vsphere/administer-arc-vmware#upgrade-the-arc-resource-bridge-manually) for Azure Arc resource bridge.

## Next steps 

- Plan your resource bridge deployment by reviewing the [support matrix for Arc-enabled VMware vSphere](support-matrix-for-arc-enabled-vmware-vsphere.md). 
- Once ready, [connect VMware vCenter Server to Azure Arc by using the helper script](quick-start-connect-vcenter-to-arc-using-script.md).
