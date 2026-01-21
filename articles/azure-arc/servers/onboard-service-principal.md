---
title: Connect hybrid machines to Azure at scale
description: In this article, you learn how to connect machines to Azure using Azure Arc-enabled servers using a service principal.
ms.topic: how-to
ms.custom:
  - devx-track-azurepowershell
  - sfi-image-nochange
ms.date: 01/21/2026
# Customer intent: As an IT administrator, I want to connect multiple hybrid machines to Azure Arc by using a service principal, so that I can efficiently manage and monitor them at scale while ensuring security best practices.
---

# Connect hybrid machines to Azure at scale

You can enable Azure Arc-enabled servers for multiple Windows or Linux machines in your environment with several flexible options depending on your requirements. Using the template script we provide, you can automate every step of the installation, including establishing the connection to Azure Arc. However, you're required to execute this script manually with an account that has elevated permissions on the target machine and in Azure.

One method to connect the machines to Azure Arc-enabled servers is to use a Microsoft Entra [service principal](/azure/active-directory/develop/app-objects-and-service-principals). This service principal method can be used instead of your privileged identity to [interactively connect the machine](onboard-portal.md). This service principal is a special limited management identity that has only the minimum permission necessary to connect machines to Azure using the `azcmagent` command. This method is safer than using a higher privileged account like a Tenant Administrator and follows our access control security best practices.

**The service principal is used only during onboarding, it isn't used for any other purpose**.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Prerequisites

Before you start connecting your machines, review the following requirements:

* Make sure you have administrator permission on the machines you want to onboard. You must have administrator privileges to install the Connected Machine agent: use the root account on Linux, or be a member of the Local Administrators group on Windows.

* Review the [Connected Machine agent prerequisites](prerequisites.md) and verify that your subscription and resources meet the requirements. You must have the **Azure Connected Machine Onboarding** role or **Contributor** role for the resource group of the machine. Make sure to register the below Azure resource providers beforehand in your target subscription.

  * Microsoft.HybridCompute

  * Microsoft.GuestConfiguration

  * Microsoft.HybridConnectivity

  * Microsoft.AzureArcData (if you plan to Arc-enable SQL Server instances)

  For more information, see [Azure resource providers prerequisites](prerequisites.md#azure-resource-providers).

For information about supported regions and other related considerations, see [supported Azure regions](overview.md#supported-regions). Also review our [at-scale planning guide](plan-at-scale-deployment.md) to understand the design and deployment criteria, as well as our management and monitoring recommendations.

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Create a service principal for onboarding at scale

You can create the service principal with Azure CLI ([Windows](/cli/azure/install-azure-cli-windows) or [Linux](/cli/azure/install-azure-cli-linux)), PowerShell or in the Azure portal.

> [!NOTE]
> To create a service principal, your Microsoft Entra tenant needs to allow users to register applications. If it doesn't, your account must be a member of the **Application Administrator** or **Cloud Application Administrator** role.
>
> For more information about tenant-level requirements, see [Delegate app registration permissions in Microsoft Entra ID](/azure/active-directory/roles/delegate-app-roles). To assign Arc-enabled server roles, your account must be a member of the **Owner** or **User Access Administrator** role in the subscription that you want to use for onboarding.

# [Azure portal](#tab/portal)

The Azure Arc service in the Azure portal provides a streamlined way to create a service principal that can be used to connect your hybrid machines to Azure.

1. At the top of the Azure portal, search for and select **Azure Arc**.
1. In the left pane, expand **Additional setup**, then select **Service principals**.
1. In the right pane, select **Add**.
1. In the following fields, provide:
   1. A name for your service principal.
   1. Choose whether the service principal has access to an entire subscription, or only to a specific resource group.
   1. Select the subscription (and resource group, if applicable) to which the service principal has access.
   1. Enter a **Service Tree ID** for the service principal (if applicable).
   1. In the **Client secret** section, you can optionally enter a friendly name of your choice in the **Description** field. Then select the duration for which your generated client secret is in use.
   1. Under **Role assignment** section, select **Azure Connected Machine Onboarding**.
1. Select **Create**.

:::image type="content" source="media/onboard-service-principal/new-azure-arc-service-principal.png" alt-text="Screenshot of the Azure Arc service principal creation screen in the Azure portal.":::

# [Azure CLI](#tab/azurecli)

You can use Azure CLI ([Windows](/cli/azure/install-azure-cli-windows) or [Linux](/cli/azure/install-azure-cli-linux)) to create a service principal with the [az ad sp create-for-rbac](/cli/azure/ad) command.

1. Sign in to Azure by running the following command:

   ```azurecli
   az login
   ```

1. Run the following command to create a service principal and assign it the Azure Connected Machine Onboarding role for the selected subscription. After the service principal is created, it will print the application ID and secret. The secret is valid for one year, after which you need to generate a new secret and update any scripts with the new secret.

   ```azurecli
   az ad sp create-for-rbac --name "Arc server onboarding account" --role "Azure Connected Machine Onboarding" --scopes "/subscriptions/<subscription-id>"
   ```

**Parameters:**

* --name: Display name for the service principal.
* --role: Assigns the Azure Connected Machine Onboarding role.
* --scopes: Scope for the role assignment (subscription level in this case).
* Replace `<subscription-id>` with your subscription ID.

**Output example:**

```azurecli
{
  "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "displayName": "Arc server onboarding account",
  "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

> [!NOTE]
> Take note of these values that are used with parameters passed to the `azcmagent` later in the section: **Install the agent and connect to Azure**.
> - The value from the **appId** property is used for the `--service-principal-id` value.
> - The value from the **password** property is used for the `--service-principal-secret` used to connect the agent.

# [Azure PowerShell](#tab/azure-powershell)

You can use [Azure PowerShell](/powershell/azure/install-azure-powershell) to create a service principal with the [New-AzADServicePrincipal](/powershell/module/Az.Resources/New-AzADServicePrincipal) cmdlet.

1. Ensure the context of your Azure PowerShell session you're working in is the correct subscription. Use [Set-AzContext](/powershell/module/az.accounts/set-azcontext) if you need to change the subscription.

   ```azurepowershell-interactive
   Get-AzContext
   ```

1. Run the following command to create a service principal and assign it the Azure Connected Machine Onboarding role for the selected subscription. After the service principal is created, it will print the application ID and secret. The secret is valid for one year, after which you need to generate a new secret and update any scripts with the new secret.

   ```azurepowershell-interactive
   $sp = New-AzADServicePrincipal -DisplayName "Arc server onboarding account" -Role "Azure Connected Machine Onboarding"
   $sp | Format-Table AppId, @{ Name = "Secret"; Expression = { $_.PasswordCredentials.SecretText }}
   ```

   ```output
   AppId                                Secret
   -----                                ------
   aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee PASSWORD_SHOWN_HERE
   ```

   The values from the following properties are used with parameters passed to the `azcmagent`:

   * The value from the **AppId** property is used for the `--service-principal-id` parameter value
   * The value from the **Secret** property is used for the `--service-principal-secret` parameter used to connect the agent.

---

## Generate the installation script from the Azure portal

Use the Azure portal to create a script that automates the agent download and installation and establishes the connection with Azure Arc. To complete the process, do the following steps:

1. At the top of the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**.
1. In the left pane, expand **Infrastructure**, then select **Machines**.
1. In the right pane, select **Onboard/Create**, then select **Onboard existing machines**.
1. On the **Basics** page, provide the following:
   1. Select the **Subscription** and **Resource group** for the machines.
   1. Under **Region**, select the Azure region to store the servers' metadata.
   1. Under **Operating system**, select the operating system that the script is configured to run on.
   1. Under **Connectivity method**:
      1. Select either **Public endpoint** or **Private endpoint**. If you select **Private endpoint**, you can either select an existing private link scope or create a new one.
      1. If you want to use a **Proxy server URL**, enter the proxy server IP address or the name and port number that the machine uses in the format `http://<proxyURL>:<proxyport>`.
      1. If you selected **Public endpoint** and you want to use [Azure Arc Gateway](arc-gateway.md), select an existing **Gateway resource** or create a new one.
   1. Under **Authentication**, select **Authenticate machines automatically**, then select your service principal.
   1. Select **Next**.
1. Under **Tags**, review the default **Physical location tags** suggested and enter a value, or specify one or more **Custom tags** to support your standards.
1. Select **Next**.
1. Under **Download and run script**, select your deployment method, then review the summary information. If you need to make changes, select **Previous** and make necessary edits.
1. Select **Download**.

For Windows, you're prompted to save `OnboardingScript.ps1`, and for Linux `OnboardingScript.sh` to your computer.

## Install the agent and connect to Azure

Taking the script template created earlier, you can install and configure the Connected Machine agent on multiple hybrid Linux and Windows machines using your organizations preferred automation tool. The script performs similar steps described in the [Connect hybrid machines to Azure from the Azure portal](onboard-portal.md) article. The difference is in the final step, where you establish the connection to Azure Arc using the `azcmagent` command using the service principal.

The following are the settings that you configure the `azcmagent` command to use for the service principal.

* `service-principal-id`: The unique identifier (GUID) that represents the application ID of the service principal.
* `service-principal-secret` | The service principal password.
* `tenant-id`: The unique identifier (GUID) that represents your dedicated instance of Microsoft Entra ID.
* `subscription-id`: The subscription ID (GUID) of your Azure subscription that you want the machines in.
* `resource-group`: The resource group name where you want your connected machines to belong to.
* `location`: See [supported Azure regions](overview.md#supported-regions). This location can be the same or different, as the resource group's location.
* `resource-name`: (*Optional*) Used for the Azure resource representation of your on-premises machine. If you don't specify this value, the machine hostname is used.

To learn more about the `azcmagent` command-line tool, see [Manage and maintain the Connected Machine agent](./manage-agent.md).

> [!NOTE]
> The Windows PowerShell script only supports running from a 64-bit version of Windows PowerShell.

After you install the agent and configure it to connect to Azure Arc-enabled servers, go to the Azure portal to verify that the server successfully connects. View your machines in the [Azure portal](https://aka.ms/hybridmachineportal) and verify your device is connected under **Arc agent status**.

:::image type="content" source="media/onboard-portal/arc-for-servers-successful-onboard.png" alt-text="Screenshot showing a successful server connection in the Azure portal.":::

## Troubleshooting

If you see the following error, then you might need to be added as a member of the **Application Administrator** or **Cloud Application Administrator** role.

```error
ServiceManagementReference field is required for Create, but is missing in the request. Refer to the TSG `https://aka.ms/service-management-reference-error` for resolving the error.
```

## Next steps

* Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
* Learn how to [troubleshoot agent connection issues](troubleshoot-agent-onboard.md).
* You can manage your machines using [Azure Policy](/azure/governance/policy/overview) for tasks such as virtual machine (VM) [guest configuration](/azure/governance/machine-configuration/overview). Azure Policy also helps verify that machines are reporting to the expected Log Analytics workspace. You can also monitor your machines with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy) and other monitoring tools.
