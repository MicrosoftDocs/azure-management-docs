---
title: Deliver Extended Security Updates for Windows Server 2012
description: Learn how to deliver Extended Security Updates for Windows Server 2012.
ms.date: 05/12/2025
ms.topic: concept-article
# Customer intent: "As an IT administrator managing Windows Server 2012, I want to provision and link Extended Security Updates to Arc-enabled servers, so that I can ensure my systems remain compliant and secure beyond their support lifecycle."
---

# Deliver Extended Security Updates for Windows Server 2012

This article provides steps to enable delivery of Extended Security Updates (ESUs) to Windows Server 2012 machines onboarded to Arc-enabled servers. You can enable ESUs to these machines individually or at scale.

## Before you begin

Plan and prepare to onboard your machines to Azure Arc-enabled servers. See [Prepare to deliver Extended Security Updates for Windows Server 2012](prepare-extended-security-updates.md) to learn more.

You also need the [Contributor](/azure/role-based-access-control/built-in-roles#contributor) role in [Azure RBAC](/azure/role-based-access-control/overview) to create and assign ESUs to Arc-enabled servers.

## Manage ESU licenses

1. From your browser, sign in to the [Azure portal](https://portal.azure.com).

1. In the service menu, under **Licenses**, select **Windows Server ESU licenses**.

    :::image type="content" source="media/deliver-extended-security-updates/extended-security-updates-main-window.png" alt-text="Screenshot of main ESU window showing licenses tab and eligible resources tab." lightbox="media/deliver-extended-security-updates/extended-security-updates-main-window.png":::

    From here, you can view and create ESU **Licenses** and view **Eligible resources** for ESUs.

## Create Azure Arc Windows Server 2012 licenses

The first step is to provision Windows Server 2012 and 2012 R2 Extended Security Update licenses from Azure Arc. You link these licenses to one or more Arc-enabled servers that you select in the next section.

After you provision an ESU license, you need to specify the SKU (Standard or Datacenter), type of cores (Physical or vCore), and number of cores to provision an ESU license. You can also provision an Extended Security Update license in a deactivated state so that it won't initiate billing or be functional on creation. Moreover, the cores associated with the license can be modified after provisioning.

> [!NOTE]
> The provisioning of ESU licenses requires you to attest to their SA or SPLA coverage.

The **Licenses** tab displays Azure Arc Windows Server 2012 licenses that are available. From here, you can select an existing license to apply or create a new license.

:::image type="content" source="media/deliver-extended-security-updates/extended-security-updates-licenses.png" alt-text="Screenshot showing existing licenses and the option to create a new one.":::

1. To create a new Windows Server 2012 license, select **Create**, and then provide the information required to configure the license on the page.

    For details on how to complete this step, see [License provisioning guidelines for Extended Security Updates for Windows Server 2012](license-extended-security-updates.md).

1. Review the information provided, and then select **Create**.

    The license you created appears in the list. You can link it to one or more Arc-enabled servers by following the steps in the next section.

## Link ESU licenses to Arc-enabled servers

You can select one or more Arc-enabled servers to link to an Extended Security Update license. Once you link a server to an activated ESU license, the server is eligible to receive Windows Server 2012 and 2012 R2 ESUs.

> [!NOTE]
> You have the flexibility to configure your patching solution of choice to receive these updates – whether that’s [Update Manager](/azure/update-center/overview), [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus), Microsoft Updates, [Microsoft Endpoint Configuration Manager](/mem/configmgr/core/understand/introduction), or a third-party patch management solution.

1. Select the **Eligible resources** tab to view a list of all your Arc-enabled servers running Windows Server 2012 and 2012 R2.

    :::image type="content" source="media/deliver-extended-security-updates/extended-security-updates-eligible-resources.png" alt-text="Screenshot of eligible resources tab showing servers eligible to receive ESUs." lightbox="media/deliver-extended-security-updates/extended-security-updates-eligible-resources.png":::

    The **ESUs status** column indicates whether or not the machine is enabled for ESUs.

1. To enable ESUs for one or more machines, select them in the list, and then select **Enable ESUs**.

1. On the **Enable Extended Security Updates** pane, it shows the number of machines selected to enable ESU and the Windows Server 2012 licenses available to apply. Select a license to link to the selected machine(s) and then select **Enable**.

   :::image type="content" source="media/deliver-extended-security-updates/extended-security-updates-select-license.png" alt-text="Screenshot of options to select the license to apply to previously chosen machines.":::

   > [!NOTE]
   > You can create a license from this page, rather than choosing an existing one, by selecting **Create an ESUs license**.

When you return to the **Eligible resources** tab, you see the status of the selected machines shows **Enabled**.

If problems occur during the enablement process, see [Troubleshoot delivery of Extended Security Updates for Windows Server 2012](troubleshoot-extended-security-updates.md) for assistance.

## Enable ESUs at scale using Azure Policy

For at-scale linking of servers to an Azure Arc Extended Security Update license and locking down license modification or creation, consider using of the following built-in Azure policies:

- [Enable Extended Security Updates (ESUs) license to keep Windows 2012 machines protected after their support lifecycle has ended (preview)](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4864134f-d306-4ff5-94d8-ea4553b18c97)

- [Deny Extended Security Updates (ESUs) license creation or modification (preview)](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4c660f31-eafb-408d-a2b3-6ed2260bd26c)

Azure policies can be specified to a targeted subscription or resource group for both auditing and management scenarios.

## Additional scenarios

There are some scenarios in which you may be eligible to receive Extended Security Updates patches at no additional cost. Two of these scenarios supported by Azure Arc are [Dev/Test (Visual Studio)](license-extended-security-updates.md#visual-studio-subscription-benefit-for-devtest-scenarios) and [Disaster Recovery (Entitled benefit DR instances from Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-by-benefits) or subscription only. Both of these scenarios require that you're already using Windows Server 2012/R2 ESUs enabled by Azure Arc for billable, production machines.

> [!WARNING]
> Don't create a Windows Server 2012/R2 ESU License for only Dev/Test or Disaster Recovery workloads. You shouldn't provision an ESU License only for nonbillable workloads. Moreover, you'll be billed fully for all of the cores provisioned with an ESU license, and any dev/test cores on the license won't be billed as long as they're tagged accordingly based on the following qualifications.

To qualify for these scenarios, you must already have:

- **Billable ESU License.** You must already have provisioned and activated a WS2012 Arc ESU License intended to be linked to regular Azure Arc-enabled servers running in production environments (such as normally billed ESU scenarios). This license should be provisioned only for billable cores, not cores that are eligible for free Extended Security Updates, for example, dev/test cores.

- **Arc-enabled servers.** Onboarded your Windows Server 2012 and Windows Server 2012 R2 machines to Azure Arc-enabled servers for the purpose of Dev/Test with Visual Studio subscriptions or Disaster Recovery.

To enroll Azure Arc-enabled servers eligible for ESUs at no additional cost, follow these steps to tag and link:

1. Tag both the WS2012 Arc ESU License (created for the production environment with cores for only the production environment servers) and the nonproduction Azure Arc-enabled servers with one of the following name-value pairs, corresponding to the appropriate exception:

    1. Name: "ESU Usage"; Value: "WS2012 VISUAL STUDIO DEV TEST"

    1. Name: "ESU Usage"; Value: "WS2012 DISASTER RECOVERY"

    In the case that you're using the ESU License for multiple exception scenarios, mark the license with the tag: Name: "ESU Usage"; Value: "WS2012 MULTIPURPOSE"

1. Link the tagged license (created for the production environment with cores only for the production environment servers) to your tagged nonproduction Azure Arc-enabled Windows Server 2012 and Windows Server 2012 R2 machines. **Do not license cores for these servers or create a new ESU license for only these servers.**

This linking won't trigger a compliance violation or enforcement block, allowing you to extend the application of a license beyond its provisioned cores. The expectation is that the license only includes cores for production and billed servers. Any additional cores will be charged and result in over-billing.

> [!IMPORTANT]
> Adding these tags to your license does NOT make the license free or reduce the number of license cores that are chargeable. These tags allow you to link your Azure machines to existing licenses that are already configured with payable cores without needing to create any new licenses or add additional cores to your free machines.

**Example:**

- You have eight Windows Server 2012 R2 Standard instances, each with eight physical cores. Six of these Windows Server 2012 R2 Standard machines are for production, and 2 of these Windows Server 2012 R2 Standard machines are eligible for free ESUs because the operating system was licensed through a Visual Studio Dev Test subscription.
  - You should first provision and activate a regular ESU License for Windows Server 2012/R2 that's Standard edition and has 48 physical cores to cover the 6 production machines. You should link this regular, production ESU license to your six production servers.
  - Next, you should reuse this existing license, don't add any more cores or provision a separate license, and link this license to your two nonproduction Windows Server 2012 R2 standard machines. You should tag the ESU license and the two nonproduction Windows Server 2012 R2 Standard machines with Name: "ESU Usage" and Value: "WS2012 VISUAL STUDIO DEV TEST".
  - This results in an ESU license for 48 cores, and you'll be billed for those 48 cores. You won't be charged for the additional 16 cores of the dev test servers that you added to this license, as long as the ESU license and the dev test server resources are tagged appropriately.

> [!NOTE]
> You needed a regular production license to start with, and you'll be billed only for the production cores.

## Upgrading from Windows Server 2012/2012 R2

When upgrading a Windows Server 2012/2012R machine to Windows Server 2016 or above, it's not necessary to remove the Connected Machine agent from the machine. The new operating system will be visible for the machine in Azure within a few minutes of upgrade completion. Upgraded machines no longer require ESUs and are no longer eligible for them. Any ESU license associated with the machine isn't automatically unlinked from the machine. See [Unlink a license](api-extended-security-updates.md#unlink-a-license) for instructions on doing so manually.

## Assess WS2012 ESU patch Status

To detect whether your Azure Arc-enabled servers are patched with the most recent Windows Server 2012/R2 Extended Security Updates, you can use the Azure Policy [Extended Security Updates should be installed on Windows Server 2012 Arc machines](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F14b4e776-9fab-44b0-b53f-38d2458ea8be/version~/null/scopes~/%5B%22%2Fsubscriptions%2F4fabcc63-0ec0-4708-8a98-04b990085bf8%22%5D). This policy definition, powered by Machine Configuration, identifies if the server has received the most recent ESU Patches. This is observable from the Guest Assignment and Azure Policy Compliance views built into the Azure portal.
