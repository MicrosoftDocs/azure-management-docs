---
ms.assetid:
title: What's new in Azure Arc-enabled SCVMM
description: This article describes the new features and enhancements supported in Azure Arc-enabled System Center Virtual Machine Manager.
ms.author: jsuri
author: jyothisuri
ms.date: 06/10/2025
ms.topic: article
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.custom:
  - build-2025
---

# What's new in Azure Arc-enabled SCVMM

Azure Arc-enabled SCVMM receives new features and enhancements on an ongoing basis. This article provides an overview of the recent developments.

## June 2025

- Ability to install Arc agents at-scale on SCVMM VMs using [out-of band methods](/azure/azure-arc/system-center-virtual-machine-manager/enable-guest-management-at-scale?tabs=Out-of-band) such as:
   - Service principal
   - System Center Configuration Manager script
   - System Center Configuration Manager custom task sequence
   - Group policy
   - Ansible playbook

## April 2025 
 
- [Azure PowerShell](/powershell/module/az.scvmm/?view=azps-13.4.0&preserve-view=true) based VM management

## January 2025

- [Windows Server Management](/azure/azure-arc/servers/windows-server-management-overview) for SCVMM VMs with active Windows Server Software Assurance licenses, giving you cost-benefits on Azure Update Manager, Azure Change Tracking and Inventory billing and exclusive capabilities.

## December 2024

- [Terraform templates](/azure/templates/microsoft.scvmm/virtualmachineinstances?pivots=deployment-language-terraform) based VM management (AzAPI provider) 
- [Bicep templates](/azure/templates/microsoft.scvmm/virtualmachineinstances?pivots=deployment-language-bicep) based VM management 
- [ARM templates](/azure/templates/microsoft.scvmm/virtualmachineinstances?pivots=deployment-language-arm-template) based VM management 

## November 2024 

- Installation of the Azure Connected Machine agent [at scale](/azure/azure-arc/system-center-virtual-machine-manager/enable-guest-management-at-scale) on VMs on SCVMM 2025 General Availability or later.

## October 2024

- [Python SDK](/python/api/overview/azure/mgmt-scvmm-readme) based VM management 
- [Java SDK](/java/api/overview/azure/resourcemanager-scvmm-readme) based VM management 
- [JavaScript SDK](/javascript/api/overview/azure/arm-scvmm-readme) based VM management 
- [Go SDK](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/scvmm/armscvmm#section-documentation) based VM management 

## August 2024

- Support to install Azure Connected Machine agent over private endpoints on SCVMM VMs at-scale.

## July 2024

- [Azure-CLI](/cli/azure/scvmm) based VM management 
- [Azure REST APIs](/rest/api/azure-arc-scvmm/operation-groups) based VM management 

## February 2024

- Support for deploying Azure Arc resource bridge with custom IP ranges in addition to SCVMM IP Pools. 

## December 2023

- Manual [upgrade support](/azure/azure-arc/system-center-virtual-machine-manager/upgrade-azure-arc-resource-bridge) for Azure Arc resource bridge.
- Support expansion for Azure Arc resource bridge with non-English resource names.
- Support for deployment of Azure Arc resource bridge on SCVMM Host Groups in addition to SCVMM Clouds.

## November 2023 (General Availability)

- Ability to [link VMs](/azure/azure-arc/system-center-virtual-machine-manager/enable-virtual-hardware-scvmm) with Azure Connected Machine agent installed with Arc-enabled SCVMM servers to leverage self-service lifecycle management.
- Procurement and delivery of [Extended Security Updates](/azure/azure-arc/system-center-virtual-machine-manager/deliver-esus-for-system-center-virtual-machine-manager-vms) for WS 2012 and 2012 R2 Arc-enabled SCVMM VMs.

## October 2023

- Installation of the Azure Connected Machine agent [at scale](/azure/azure-arc/system-center-virtual-machine-manager/enable-guest-management-at-scale) on VMs on SCVMM 2022 UR1 or later and SCVMM 2019 UR5 or later.
- Installation of the Azure Connected Machine agent on SCVMM VMs [using a script](/azure/azure-arc/system-center-virtual-machine-manager/install-arc-agents-using-script).
- Support for Azure services to [govern, protect, configure, and monitor](/azure/azure-arc/servers/overview#supported-cloud-operations) Arc-enabled SCVMM VMs.
- Ability to [recover](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery) the Azure Arc resource bridge in case of accidental deletion.  


## Next steps

- [Connect your System Center Virtual Machine Manager management server to Azure Arc](/azure/azure-arc/system-center-virtual-machine-manager/quickstart-connect-system-center-virtual-machine-manager-to-arc).
