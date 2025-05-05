---
title: Connect Windows Server machines to Azure through Azure Arc Setup
description: In this article, you learn how to connect Windows Server machines to Azure Arc using the built-in Windows Server Azure Arc Setup wizard.
ms.date: 10/31/2024
ms.topic: concept-article
---

# Connect Windows Server machines to Azure through Azure Arc Setup

Windows Server machines can be onboarded directly to [Azure Arc](https://azure.microsoft.com/products/azure-arc/) through a graphical wizard included in Windows Server. The wizard automates the onboarding process by checking the necessary prerequisites for successful Azure Arc onboarding and fetching and installing the latest version of the Azure Connected Machine (AzCM) agent. Once the wizard process completes, you're directed to your Window Server machine in the Azure portal, where it can be viewed and managed like any other Azure Arc-enabled resource.

Onboarding to Azure Arc is not needed if the Windows Server machine is already running in Azure.

For Windows Server 2022, Azure Arc Setup is an optional component that can be removed using the **Remove Roles and Features Wizard**. For Windows Server 2025 and later, Azure Arc Setup is a [Features On Demand](/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities?view=windows-11). Essentially, this means that the procedures for removal and enablement differ between OS versions. See  for more information.

> [!NOTE]
> The Azure Arc Setup feature only applies to Windows Server 2022 and later. It was released in the [Cumulative Update of 10/10/2023](https://support.microsoft.com/en-us/topic/october-10-2023-kb5031364-os-build-20348-2031-7f1d69e7-c468-4566-887a-1902af791bbc).

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Prerequisites

* Azure Arc-enabled servers - Review the [prerequisites](prerequisites.md) and verify that your subscription, your Azure account, and resources meet the requirements.

* An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

* Modern browser (Microsoft Edge) for authentication to Microsoft Azure. Configuration of the Azure Connected Machine agent requires authentication to your Azure account, either through interactive authentication on a modern browser or device code login on a separate device (if the machine doesn't have a modern browser).

## Launch Azure Arc Setup and connect to Azure Arc

The Azure Arc Setup wizard is launched from a system tray icon at the bottom of the Windows Server machine when the Azure Arc Setup feature is enabled. This feature is enabled by default. Alternatively, you can launch the wizard from a pop-up window in the Server Manager or from the Windows Server Start menu.

1. Select the Azure Arc system tray icon, then select **Launch Azure Arc Setup**.

    :::image type="content" source="media/onboard-windows-server/system-tray-icon.png" alt-text="Screenshot showing Azure Arc system tray icon and window to launch Azure Arc setup process.":::
   
1. The introduction window of the Azure Arc Setup wizard explains the benefits of onboarding your machine to Azure Arc. When you're ready to proceed, click **Next**.

    :::image type="content" source="media/onboard-windows-server/get-started-with-arc.png" alt-text="Screenshot of the Getting Started page of the wizard.":::

1. The wizard automatically checks for the prerequisites necessary to install the Azure Connected Machine agent on your Windows Server machine. Once this process completes and the agent is installed, select **Configure**.

1. The configuration window details the steps required to configure the Azure Connected Machine agent. When you're ready to begin configuration, select **Next**.

1. Sign-in to Azure by selecting the applicable Azure cloud, and then selecting **Sign in to Azure**. You'll be asked to provide your sign-in credentials.

1. Provide the resource details of how your machine will work within Azure Arc, such as the **Subscription** and **Resource group**, and then select **Next**.

    :::image type="content" source="media/onboard-windows-server/resource-details.png" alt-text="Screenshot of resource details window with fields.":::

1. Once the configuration completes and your machine is onboarded to Azure Arc, select **Finish**.

1. Go to the Server Manager and select **Local Server** to view the status of the machine in the **Azure Arc Management** field. A successfully onboarded machine has a status of **Enabled**.

    :::image type="content" source="media/onboard-windows-server/server-manager-enabled.png" alt-text="Screenshot of Server Manager local server pane showing machine status is enabled.":::


## Server Manager functions

You can select the **Enabled/Disabled** link in the **Azure Arc Management** field of the Server Manager to launch different functions based on the status of the machine:

- If Azure Arc Setup isn't installed, selecting **Enabled/Disabled** launches the **Add Roles and Features Wizard**.
- If Azure Arc Setup is installed and the Azure Connected Machine agent hasn't been installed, selecting **Disabled** launches `AzureArcSetup.exe`, the executable file for the Azure Arc Setup wizard.
- If Azure Arc Setup is installed and the Azure Connected Machine agent is already installed, selecting **Enabled/Disabled** launches `AzureArcConfiguration.exe`, the executable file for configuring the Azure Connected Machine agent to work with your machine.
    
## Viewing the connected machine

The Azure Arc system tray icon at the bottom of your Windows Server machine indicates if the machine is connected to Azure Arc; a red symbol means the machine does not have the Azure Connected Machine agent installed. To view a connected machine in Azure Arc, select the icon and then select **View Machine in Azure**. You can then view the machine in the [Azure portal](https://portal.azure.com/), just as you would other Azure Arc-enabled resources.

:::image type="content" source="media/onboard-windows-server/view-machine.png" alt-text="Screenshot of the machine connection status window  highlighting the View Machine in Azure button.":::

## Uninstalling Azure Arc Setup

> [!NOTE]
> Uninstalling Azure Arc Setup does not uninstall the Azure Connected Machine agent from the machine. For instructions on uninstalling the agent, see [Managing and maintaining the Connected Machine agent](manage-agent.md).
>
To uninstall Azure Arc Setup from a Windows Server 2022 machine:

1. In the Server Manager, navigate to the **Remove Roles and Features Wizard**. (See [Remove roles, role services, and features by using the Remove Roles and Features Wizard](/windows-server/administration/server-manager/install-or-uninstall-roles-role-services-or-features#remove-roles-role-services-and-features-by-using-the-remove-roles-and-features-wizard) for more information.)

1. On the Features page, uncheck the box for **Azure Arc Setup**.

1. On the confirmation page, select **Restart the destination server automatically if required**, then select **Remove**.

To uninstall Azure Arc Setup through PowerShell, run the following command:

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName AzureArcSetup
```

To uninstall Azure Arc Setup from a Windows Server 2025 machine:

1. Open the Settings app on the machine and select **System**, then select **Optional features**.

1. Select **AzureArcSetup**, and then select **Remove**.

:::image type="content" source="media/onboard-windows-server/arc-setup-remove.png" alt-text="Screenshot of Optional feature menu showing Azure Arc Setup feature with Remove button.":::

To uninstall Azure Arc Setup from a Windows Server 2025 machine from the command line, run the following line of code:

`DISM /online /Remove-Capability /CapabilityName:AzureArcSetup~~~~`

## Next steps

* Troubleshooting information can be found in the [Troubleshoot Azure Connected Machine agent guide](troubleshoot-agent-onboard.md).

* Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.

* Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview), for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verifying the machine is reporting to the expected Log Analytics workspace, enable monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.

* To learn more about Windows Server licensing on a pay-per-usage basis for your Arc-enabled servers, see [Windows Server Pay-as-you-go](/windows-server/get-started/windows-server-pay-as-you-go).
