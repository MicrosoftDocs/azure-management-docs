---
title: Retirement of the Azure Arc-enabled System Center Virtual Machine Manager
description: This article provides transition guidance following the retirement of Azure Arc-enabled System Center Virtual Machine Manager.
ms.date: 09/24/2026
ms.topic: how-to
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: krkarthik
keywords: "VMM, Arc, Azure, System Center"

# Customer intent: As a system administrator, I want to understand my options as Azure Arc-enabled System Center Virtual Machine Manager retires.
---

# Retirement of Azure Arc-enabled System Center Virtual Machine Manager

Azure Arc-enabled System Center Virtual Machine Manager (SCVMM) enables organizations to extend Azure management capabilities to virtual machines (VMs) managed through System Center Virtual Machine Manager. The service provided inventory visibility in Azure, VM lifecycle operations, self-service VM management through Azure role-based access control, at-scale Arc agent installation, and access to Azure security, governance, monitoring, update management, and automation services for virtual machines. 

As part of Azure Arc product portfolio evolution, Azure Arc-enabled SCVMM is retiring. Transition to services that align with your long-term management and modernization requirements.

>[!IMPORTANT]
> Azure Arc-enabled System Center Virtual Machine Manager (SCVMM) retires in September 2029. Begin planning your transition to the preferred alternatives based on how you use the service today.
> All new onboardings stop by October 2026. For the existing customers i.e., Azure subscriptions with active Azure Arc-enabled SCVMM deployments, Azure Arc-enabled SCVMM will be available for use with all the current capabilities until September 2029.
> Customers should plan and complete the transition as early as possible to have sufficient lead time before retirement. Customers can share their feedback or any concerns that they may have during this transition through email to arc-vmm-feedback@microsoft.com.

### Which migration path should I choose?

| **Current usage** | **Recommended path** |
| --- | --- |
| You **ONLY** use Azure management services such as Update Manager, Defender for Cloud, Azure Monitor, Azure Policy, Guest Configuration, or licensing Extended Security Updates, Pay-as-you-go SQL and Windows Server and **DON'T** require Azure-based VM lifecycle management. | Azure Arc-enabled Servers |
| You **ONLY** use Azure to perform VM lifecycle operations such as create, start, stop, restart, resize, delete, or self-service VM provisioning. | Contact arc-vmm-feedback@microsoft.com |
| You use **BOTH** VM lifecycle operations and Azure management services. | Contact arc-vmm-feedback@microsoft.com |

## Transition to Azure Arc-enabled Servers (Option 1)

Azure Arc-enabled Servers provides guest operating system management regardless of the underlying virtualization platform. Azure Arc-enabled Servers doesn't provide virtualization-layer VM lifecycle operations. It's intended for Azure management services usage and license procurement on the VMs. 

### High-level transition process
1. Identify the VMs currently onboarded to Azure services for patching, monitoring, security, etc. and enrolled for Azure-based licensing like Extended Security Updates (ESUs), Pay-as-you-go licensing through Azure Arc-enabled SCVMM.
2. Execute the Azure CLI command by scoping it to the machines individually or at a resource group or a subscription level. **The Azure CLI command will be updated here by October 2026**.
3. Validate connectivity between the machines and Azure Arc.
4. Verify policies, monitoring, updates, security, license billing, and compliance functionality.
5. Establish plans to Arc-onboard additional machines at-scale in the future, if any. 
6. Remove your Azure Arc-enabled SCVMM resources gracefully from Azure. To deboard your SCVMM managed environment from Azure Arc-enabled SCVMM, follow [these steps](remove-scvmm-from-azure-arc.md). 

## Contact arc-vmm-feedback@microsoft.com (Option 2)

If your organization uses Azure Arc-enabled SCVMM to perform VM lifecycle operations from Azure, Microsoft will work with you to identify an appropriate transition path. Because infrastructure, connectivity, and VM management requirements vary across organizations, there isn't a single replacement solution recommended for every Azure Arc-enabled SCVMM environment. Microsoft has transition options for both connected and disconnected scenarios. 

Contact us at arc-vmm-feedback@microsoft.com to start your transition assessment. The Azure Arc-enabled SCVMM product team will review your current environment and requirements with you and provide guidance on the Microsoft option that best fits your scenario. We encourage you to start this assessment early to allow sufficient time to plan and complete your transition. 

## Frequently asked questions

### What happens in September 2026?
In September **2026**, Microsoft announced that Azure Arc-enabled SCVMM retires in September **2029**. However, customers should transition to the recommended alternatives and avoid planning new deployments. 

### Will I lose any of the existing functionalities of Azure Arc-enabled SCVMM? 
No. The existing capabilities will be supported till September 2029 and if you are an existing customer, you can continue to use the service with no impact. However, Microsoft will not add new capabilities to Azure Arc-enabled SCVMM in the interim period. Customers are recommended to plan and switch to the recommended alternatives as early as possible. 

### I'm planning to deploy Azure Arc-enabled SCVMM. Can I use the service until September 2029?
Microsoft won't add new capabilities to Azure Arc-enabled SCVMM. While the onboarding experience will be available for the existing customers, newer customers won't be able to deploy Azure Arc-enabled SCVMM starting from October 2026. Instead, evaluate Azure Arc-enabled Servers and Azure Local for your cloud-native VM management needs.  

### Can I continue using System Center Virtual Machine Manager?
Yes. The retirement applies only to Azure Arc-enabled SCVMM. Customers can continue using their on-premises SCVMM product subject to the applicable System Center lifecycle and support policies for the versions. 

### Will the Azure Arc agents on my VMs continue to work?
While the Azure Arc agents installed on the VMs continue to work, you need to use Azure CLI command to switch your Azure Arc-enabled SCVMM machines to Azure Arc-enabled Server machines. When done, remove the rest of the Azure Arc-enabled SCVMM resources by following the steps in [this article](remove-scvmm-from-azure-arc.md).

## Related content

### Azure Arc-enabled Servers documentation

- [Azure Arc-enabled Servers overview](/azure/azure-arc/servers/overview)
- [Plan and deploy Azure Arc-enabled Servers](/azure/azure-arc/servers/plan-at-scale-deployment)
- Interactive onboarding options:
  - [Connect machines to Azure using a deployment script](/azure/azure-arc/servers/onboard-portal)
  - [Connect hybrid machines to Azure by using Azure PowerShell](/azure/azure-arc/servers/onboard-powershell)
  - [Connect Windows Server machines to Azure through Azure Arc Setup](/azure/azure-arc/servers/onboard-windows-server)
- At-scale onboarding options:
  - [Connect hybrid machines to Azure at scale](/azure/azure-arc/servers/onboard-service-principal)
  - [Connect machines at scale by running PowerShell scripts with Configuration Manager](/azure/azure-arc/servers/onboard-configuration-manager-powershell)
  - [Connect machines at scale with a Configuration Manager custom task sequence](/azure/azure-arc/servers/onboard-configuration-manager-custom-task)
  - [Connect machines at scale using Group Policy with a PowerShell script](/azure/azure-arc/servers/onboard-group-policy-powershell)
- [VM Extension Management with Azure Arc-enabled Servers](/azure/azure-arc/servers/manage-vm-extensions)

### Azure Local onboarding and management documentation

- [Azure Local documentation](/azure/azure-local)
- [What Is Azure Local? Overview and Key Benefits](/azure/azure-local/overview)
- [Overview of Hyperconverged Deployments for Azure Local](/azure/azure-local/overview/hyperconverged-overview)
- [Overview of Disaggregated Deployments for Azure Local](/azure/azure-local/overview/disaggregated-overview)
- [Prerequisites to Deploy Azure Local Version](/azure/azure-local/deploy/deployment-prerequisites)
- [Azure Local, version 23H2 deployment overview](/azure/azure-local/deploy/deployment-introduction)
- [Deploy Azure Local by using the Azure portal](/azure/azure-local/deploy/deploy-via-portal)
- [Use Azure Migrate to move Hyper-V VMs to Azure Local (preview)](/azure/azure-local/migrate/migration-azure-migrate-overview)
- YouTube walkthrough: [Migrate Hyper-V VMs to Azure Local with Azure Migrate](https://www.youtube.com/watch?v=0kiiuNmT3Dk) 
