---
title: Connect machines at scale by running PowerShell scripts with Configuration Manager
description: You can use Configuration Manager to run a PowerShell script that automates at-scale onboarding to Azure Arc-enabled servers.
ms.date: 03/12/2025
ms.topic: how-to
# Customer intent: As an IT administrator, I want to automate the onboarding of Azure Arc-enabled servers using PowerShell scripts in Configuration Manager, so that I can efficiently manage servers at scale and ensure secure, centralized deployment of applications and updates.
---

# Connect machines at scale by running PowerShell scripts with Configuration Manager

Microsoft Configuration Manager facilitates comprehensive management of servers supporting the secure and scalable deployment of applications, software updates, and operating systems. Configuration Manager has an integrated ability to run PowerShell scripts.

You can use Configuration Manager to run a PowerShell script that automates at-scale onboarding to Azure Arc-enabled servers.

Before you get started, be sure to review the [prerequisites](prerequisites.md) and verify that your subscription and resources meet the requirements. For information about supported regions and other related considerations, see [supported Azure regions](overview.md#supported-regions). Also review our [at-scale planning guide](plan-at-scale-deployment.md) to understand the design and deployment criteria, as well as our management and monitoring recommendations.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Prerequisites for Configuration Manager to run PowerShell scripts

The following prerequisites must be met to use PowerShell scripts in Configuration Manager:

- The Configuration Manager version must be 1706 or higher.
- To import and author scripts, your Configuration Manager account must have **Create** permissions for **SMS Scripts**.
- To approve or deny scripts, your Configuration Manager account must have **Approve** permissions for **SMS Scripts**.
- To run scripts, your Configuration Manager account must have **Run Script** permissions for **Collections**.

## Generate a service principal and prepare the installation script

Before you can run the script to connect your machines, you must:

1. Follow the steps to [create a service principal for onboarding at scale](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale). Assign the **Azure Connected Machine Onboarding** role to your service principal, and limit the scope of the role to the target Azure landing zone. Make a note of the Service Principal Secret, as you'll need this value later.

2. Follow the steps to [generate the installation script from the Azure portal](onboard-service-principal.md#generate-the-installation-script-from-the-azure-portal). While you'll use this installation script later, don't run the script in PowerShell.

## Create the script in Configuration Manager

Before you begin, check in **Configuration Manager Default Settings** that the PowerShell execution policy under **Computer Agent** is set to **Bypass**.

1. In the Configuration Manager console, select **Software Library**.
1. In the **Software Library** workspace, select **Scripts**.
1. On the **Home** tab, in the **Create** group, select **Create Script**.
1. On the **Script** page of the **Create Script** wizard, configure the following settings:
   1. **Script Name** – Onboard Azure Arc
   1. **Script language** - PowerShell
   1. **Import** – Import the installation script that you generated in the Azure portal.
      :::image type="content" source="media/onboard-configuration-manager-powershell/configuration-manager-create-script.png" alt-text="Screenshot of the Create Script screen in Configuration Manager.":::
1. In the Script Wizard, paste the script generated from Azure portal. Edit this pasted script with the Service Principal Secret for the service principal you generated.
1. Complete the wizard. The new script is displayed in the **Script** list with a status of **Waiting for approval**.

## Approve the script in Configuration Manager

With an account that has **Approve** permissions for **SMS Scripts**, do the following:

1. In the Configuration Manager console, select **Software Library**.
1. In the **Software Library** workspace, select **Scripts**.
1. In the **Script** list, choose the script you want to approve or deny. Then, on the Home tab, in the Script group, select **Approve/Deny**.
1. In the **Approve or deny script** dialog box, select **Approve** for the script.
   :::image type="content" source="media/onboard-configuration-manager-powershell/configuration-manager-approve-script.png" alt-text="Screenshot of the Approve or deny script screen in Configuration Manager.":::
1. Complete the wizard, then confirm that the new script is shown as **Approved** in the **Script** list.

## Run the script in Configuration Manager

Select a collection of targets for your script by doing the following:

1. In the Configuration Manager console, select **Assets and Compliance**.
1. In the **Assets and Compliance** workspace, select **Device Collections**.
1. In the **Device Collections** list, select the collection of devices on which you want to run the script.
1. Select a collection of your choice, and then select **Run Script**.
1. On the **Script** page of the **Run Script** wizard, choose the script you authored and approved.
1. Select **Next**, and then complete the wizard.

## Verify successful connection to Azure Arc

The script status monitoring indicates whether the script has successfully installed the Connected Machine Agent to the collection of devices. Successfully onboarded Azure Arc-enabled servers will also be visible in the [Azure portal](https://aka.ms/hybridmachineportal).

:::image type="content" source="media/onboard-portal/arc-for-servers-successful-onboard.png" alt-text="Screenshot of the Azure portal showing successful onboarding of Azure Arc-enabled servers.":::

## Next steps

- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
- Review connection troubleshooting information in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md).
- Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview) for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verifying that the machine is reporting to the expected Log Analytics workspace, enabling monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.