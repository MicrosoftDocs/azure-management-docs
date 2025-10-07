---
title:  Deliver ESUs for SCVMM VMs through Arc
description: Deliver ESUs for SCVMM VMs through Azure Arc. 
ms.date: 03/05/2025
ms.topic: how-to
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
author: Jeronika-MS
keywords: "VMM, Arc, Azure, System Center"
ms.custom:
  - build-2025
# Customer intent: "As an IT administrator managing SCVMM VMs, I want to deliver Extended Security Updates through Azure Arc, so that I can ensure compliance and security for my Windows Server 2012/2012 R2 environments without needing key activation."
---

# Deliver Extended Security Updates for SCVMM VMs through Arc

This article provides the steps to procure and deliver Extended Security Updates (ESUs) to Windows Server 2012 and 2012 R2 System Center Virtual Machine Manager (SCVMM) VMs onboarded to Azure Arc-enabled SCVMM.

Azure Arc-enabled System Center Virtual Machine Manager allows you to enroll all the Windows Server 2012/2012 R2 VMs managed by your SCVMM server in [Extended Security Updates](/windows-server/get-started/extended-security-updates-overview) at scale.

## Key benefits

Delivering ESUs for SCVMM VMs through Arc offers the following benefits:

- **Pay-as-you-go:** Flexibility to sign up for a monthly subscription service with the ability to migrate mid-year.

- **Azure billed:** You can draw down from your existing [Microsoft Azure Consumption Commitment (MACC)](/marketplace/azure-consumption-commitment-benefit) and analyze your costs using [Microsoft Cost Management and Billing](/azure/cost-management-billing/cost-management-billing-overview).

- **Built-in inventory:** The coverage and enrollment status of Windows Server 2012/2012 R2 ESUs on eligible Azure Arc-enabled SCVMM VMs are identified in the Azure portal, highlighting gaps and status changes.

- **Keyless delivery:** The enrollment of ESUs on Azure Arc-enabled SCVMM Windows Server 2012/2012 R2 VMs won't require the acquisition or activation of keys. 

- **Access to Azure management services:** ESUs enabled by Azure Arc give access to Azure management services such as [Azure Update Manager](/azure/update-manager/overview?tabs=azure-vms), [Azure Automation Change Tracking and Inventory](/azure/automation/change-tracking/overview?tabs=python-2), and [Azure Policy Guest Configuration](/azure/cloud-adoption-framework/manage/azure-server-management/guest-configuration-policy) at no additional cost.  

>[!NOTE]
> - Through Azure Arc-enabled SCVMM, you can procure and deliver ESUs only for SCVMM managed VMs and not for your hosts. 
> - To purchase ESUs, you must have Software Assurance through Volume Licensing Programs such as an Enterprise Agreement (EA), Enterprise Agreement Subscription (EAS), Enrollment for Education Solutions (EES), or Server and Cloud Enrollment (SCE). Alternatively, if your Windows Server 2012/2012 R2 machines are licensed through Services Provider License Agreement (SPLA) or with a Server Subscription, Software Assurance isn't required to purchase ESUs.

## Prerequisites

Before you procure and deliver ESUs for SCVMM VMs through Arc, ensure you meet these prerequisites:

- The user account must have an Owner/Contributor role in a Resource Group in Azure to create and assign ESUs to SCVMM VMs. 
- The SCVMM server managing the Windows Server 2012 and 2012 R2 VMs, for which the ESUs are to be applied, should be [onboarded to Azure Arc](./quickstart-connect-system-center-virtual-machine-manager-to-arc.md). After onboarding, the Windows Server 2012 and 2012 R2 VMs, for which the ESUs are to be applied, should be [Azure-enabled](enable-scvmm-inventory-resources.md) and [guest management enabled](./enable-guest-management-at-scale.md). 

## Create Azure Arc ESUs 

To create Azure Arc ESUs, follow these steps:

1.	Sign in to the [Azure portal](https://portal.azure.com/).

2.	On the **Azure Arc** page, select **Extended Security Updates** in the left pane. From here, you can view and create ESU **Licenses** and view **Eligible resources** for ESUs.

3.	The **Licenses** tab displays Azure Arc Windows Server 2012 licenses that are available. Select an existing license to apply or create a new license.

4.	To create a new Windows Server 2012 license, select **Create**, and then provide the information required to configure the license on the page. For detailed information on how to complete this step, see [License provisioning guidelines for Extended Security Updates for Windows Server 2012](../servers/license-extended-security-updates.md).

    :::image type="content" source="media/deliver-esus-for-scvmm-vms/select-or-create-license.png" alt-text="Screenshot of how to create a new license." lightbox="media/deliver-esus-for-scvmm-vms/select-or-create-license.png":::

5.	Review the information provided and select **Create**. The license you created appears in the list, and you can link it to one or more Azure Arc-enabled SCVMM VMs by following the steps in the next section.

    :::image type="content" source="media/deliver-esus-for-scvmm-vms/new-license-created.png" alt-text="Screenshot showing the successful creation of a new license." lightbox="media/deliver-esus-for-scvmm-vms/new-license-created.png":::

## Link ESU licenses to Azure Arc-enabled SCVMM VMs

You can select one or more Azure Arc-enabled SCVMM VMs to link to an ESU license. Once you've linked a VM to an activated ESU license, the VM is eligible to receive Windows Server 2012 and 2012 R2 ESUs.

>[!NOTE]
> You have the flexibility to configure your patching solution of choice to receive these updates – whether it's [Azure Update Manager](/azure/update-center/overview), [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus), Microsoft Updates, [Microsoft Endpoint Configuration Manager](/mem/configmgr/core/understand/introduction), or a non-Microsoft patch management solution.

To link ESU licenses, follow these steps:

1.	Select the **Eligible resources** tab to view a list of all your Azure Arc-enabled server machines running Windows Server 2012 and 2012 R2, including SCVMM machines that have the Azure Connected Machine agent installed. The **ESUs status** column indicates whether the machine is ESUs enabled.
 
    :::image type="content" source="media/deliver-esus-for-scvmm-vms/view-arc-enabled-machines.png" alt-text="Screenshot of Azure Arc-enabled server machines running Windows Server 2012 and 2012 R2 under the eligible resources tab." lightbox="media/deliver-esus-for-scvmm-vms/view-arc-enabled-machines.png":::

2.	To enable ESUs for one or more machines, select them in the list, and then select **Enable ESUs**.

3.	On the **Enable Extended Security Updates** page, you can see the number of machines selected to enable ESUs and the Windows Server 2012 licenses available to apply. Select a license to link to the selected machine(s) and select **Enable**.

    :::image type="content" source="media/deliver-esus-for-scvmm-vms/enable-license.png" alt-text="Screenshot of how to select and enable license." lightbox="media/deliver-esus-for-scvmm-vms/enable-license.png":::

4.	The **ESUs status** column value of the selected machines changes to **Enabled**.

    :::image type="content" source="media/deliver-esus-for-scvmm-vms/extended-security-updates-enabled-resources.png" alt-text="Screenshot of eligible resources tab showing status of enabled for previously selected servers." lightbox="media/deliver-esus-for-scvmm-vms/extended-security-updates-enabled-resources.png":::

## Access to Azure services

For Azure Arc-enabled SCVMM VMs enrolled in WS2012/2012 R2 ESUs enabled by Azure Arc, free access is provided to these Azure services from **October 10, 2023**.

- [Azure Update Manager](/azure/update-manager/overview): Unified management and governance of update compliance that includes not only Azure and hybrid machines, but also ESU update compliance for all your Windows Server 2012/2012 R2 SCVMM VMs. Enrollment in ESUs doesn't have an impact on Azure Update Manager. After enrollment in ESUs through Azure Arc, the server becomes eligible for ESU patches. These patches can be delivered through Azure Update Manager or any other patching solution. You'll still need to configure updates from Microsoft Updates or [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus). 

- [Azure Automation Change Tracking and Inventory](/azure/automation/change-tracking/overview): Track changes in Azure Arc-enabled SCVMM VMs. 

- [Azure Policy Guest Configuration](/azure/cloud-adoption-framework/manage/azure-server-management/guest-configuration-policy): Audit the configuration settings in an Azure Arc-enabled SCVMM VM. 

## Upgrade from Windows Server 2012/2012 R2  

You can select one or more Azure Arc-enabled SCVMM VMs to link to an ESU license. Once you've linked a VM to an activated ESU license, the VM is eligible to receive Windows Server 2012 and 2012 R2 ESUs. When upgrading a Windows Server 2012/2012 R2 machine to Windows Server 2016 or above, it's not necessary to remove the Azure Connected Machine agent from the machine. 

The new operating system will be visible for the machine in Azure within a few minutes of upgrade completion. Upgraded machines no longer require ESUs and are no longer eligible for them. Any ESU license associated with the machine isn't automatically unlinked from the machine. See [Unlink a license](/azure/azure-arc/servers/api-extended-security-updates#unlink-a-license) for instructions on doing so manually.

>[!NOTE]
> - See [Troubleshoot delivery of Extended Security Updates for Windows Server 2012](../servers/troubleshoot-extended-security-updates.md) to troubleshoot any problems that occur during the enablement process.<br>
> - Review the [additional scenarios](../servers/deliver-extended-security-updates.md#additional-scenarios) in which you may be eligible to receive ESU patches at no additional cost.<br>
> - See the [billing principles](/azure/azure-arc/servers/deliver-extended-security-updates) for ESUs enabled by Azure Arc.

## Next step

[Programmatically deploy and manage Azure Arc Extended Security Updates licenses](../servers/api-extended-security-updates.md).
