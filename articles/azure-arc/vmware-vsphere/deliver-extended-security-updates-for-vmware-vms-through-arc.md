---
title: Deliver ESUs for VMware VMs through Arc
description: Deliver ESUs for VMware VMs through Azure Arc.
ms.date: 10/06/2026
ms.topic: how-to
ms.services: azure-arc
ms.subservice: vmware-vsphere-azure-arc
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
keywords: "VMware, Arc, Azure"
# Customer intent: As an IT administrator managing VMware VMs, I want to enroll Windows Server 2016 VMs in Extended Security Updates through Azure Arc, so that I can ensure they receive critical updates and maintain security compliance efficiently.
---

# Deliver ESUs for VMware VMs through Arc

By using Azure Arc-enabled VMware vSphere, you can enroll all the Windows Server 2016 VMs that your vCenter manages in [Extended Security Updates (ESUs)](/windows-server/get-started/extended-security-updates-overview) at scale. By using Arc ESUs, you get cost flexibility through pay-as-you-go Azure billing and an enhanced delivery experience with built-in inventory and keyless delivery. This article provides the steps to procure and deliver ESUs to WS 2016 VMware VMs that you onboard to Azure Arc-enabled VMware vSphere. 

>[!Note]
> To purchase ESUs, you must have Software Assurance through Volume Licensing Programs such as an Enterprise Agreement (EA), Enterprise Agreement Subscription (EAS), Enrollment for Education Solutions (EES), or Server and Cloud Enrollment (SCE). Alternatively, if your Windows Server 2016 machines are licensed through SPLA or with a Server Subscription, you don't need Software Assurance to purchase ESUs.

## Prerequisites

- The user account must have an Owner or Contributor role in a Resource Group in Azure to create and assign ESUs to VMware VMs.
- The vCenter managing the WS 2016 VMs, for which the ESUs are to be applied, should be [onboarded to Azure Arc](quick-start-connect-vcenter-to-arc-using-script.md).
- The WS 2016 VMs, for which the ESUs are to be applied, should be [onboarded to Azure Arc with the Arc agent installed](enable-vcenter-resources-in-azure.md).

## Manage ESU licenses

1. From your browser, sign in to the [Azure portal](https://portal.azure.com).

1. Go to the **Azure Arc** page, and in the service menu, under **Licenses**, select the **Windows Server 2016 ESU licenses** offering.

    :::image type="content" source="/azure/azure-arc/servers/media/deliver-extended-security-updates/extended-security-updates-2016-main-window.png" alt-text="Screenshot of main ESU window showing licenses tab and eligible resources tab." lightbox="/azure/azure-arc/servers/media/deliver-extended-security-updates/extended-security-updates-2016-main-window.png":::

    From here, you can view and create ESU **Licenses** and view **Eligible resources** for ESUs.

## Create Azure Arc ESUs 

First, provision Extended Security Update licenses from Azure Arc. Link these licenses to one or more Arc-enabled servers that you select in the next section.

> [!NOTE]
> To provision ESU licenses, you must attest to their SA or SPLA coverage.

For Windows Server 2016, specify the SKU (Standard or Datacenter) and the number of cores when you create the license. Unlike Windows Server 2012, you select the core type (physical or virtual) later, when you enable ESUs on your machines. You can also provision the license in a deactivated state so that it doesn't initiate billing or be functional on creation.

1. On the license page, select **Create**.

1. On the license creation page, provide the following information:

    - **Resource group**: Select the resource group for the license.
    - **License name**: Enter a name for the license.
    - **Activation status**: Select the activation status.
    - **Region**: Select your region.
    - **SKU**: Select **Windows Server 2016 Standard** or **Windows Server 2016 Datacenter**.
    - **Number of cores**: Enter the number of cores that the license supports.

    For details on how to complete this step, see [License provisioning guidelines for Extended Security Updates for Windows Server](../servers/license-extended-security-updates.md).

1. Select **Next**, confirm that your Windows Server licenses have Software Assurance, and then select **Create**.

    The license you created appears in the list. You can link it to one or more Arc-enabled VMware vSphere VMs by following the steps in the next section.

## Link ESU licenses to Arc-enabled VMware vSphere VMs

Select one or more Arc-enabled VMware vSphere VMs to link to an ESU license. After you link a VM to an activated ESU license, the VM can receive Windows Server 2016 ESUs.

>[!Note]
> You have the flexibility to configure your patching solution of choice to receive these updates – whether that's [Update Manager](/azure/update-center/overview), [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus), Microsoft Updates, [Microsoft Endpoint Configuration Manager](/mem/configmgr/core/understand/introduction), or a third-party patch management solution.

1.	Select the **Eligible Resources** tab to view a list of all your Arc-enabled machines running Windows Server 2016, including VMware vSphere VMs that have the Arc agent installed. The **ESUs status** column indicates whether the machine is enabled for ESUs.
 
    :::image type="content" source="/azure/azure-arc/servers/media/deliver-extended-security-updates/extended-security-updates-2016-eligible-resources.png" alt-text="Screenshot of eligible resources tab showing servers eligible to receive ESUs." lightbox="/azure/azure-arc/servers/media/deliver-extended-security-updates/extended-security-updates-2016-eligible-resources.png":::

1.	To enable ESUs for one or more machines, select them in the list, and then select **Enable ESUs**.

  	In the **Enable ESUs** dialog:
    - Select the **Core type**: **Physical cores** or **Virtual cores**.
    - Select the ESU license that you created.
    - Select **Enable**.

    When you return to the **Eligible resources** tab, you see the status of the selected machines shows **Enabled**.

To verify enrollment on the server, run `azcmagent show` and confirm that the **Extended Security Updates** section shows a status of **Active**.

If problems occur during the enablement process, see [Troubleshoot delivery of Extended Security Updates for Windows Server](../servers/troubleshoot-extended-security-updates.md) for assistance.

## Next steps

[Programmatically deploy and manage Azure Arc Extended Security Updates licenses](../servers/api-extended-security-updates.md).
