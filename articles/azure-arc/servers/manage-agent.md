---
title:  Manage Azure Connected Machine agent versions
description: This article describes how to install a specific Azure Connected Machine agent version and manage automatic agent upgrades (preview) for Azure Arc-enabled servers.
ms.date: 03/24/2026
ms.topic: how-to
# Customer intent: As a system administrator, I want to manage versions of the Azure Connected Machine agent, including managing automatic upgrades, so that I can ensure optimal performance and reliability of my Azure-connected servers.
---

# Manage Azure Connected Machine agent versions for Azure Arc-enabled servers

After you deploy the Azure Connected Machine agent, you may need to reconfigure the agent, upgrade it, remove it, or make other changes. These routine maintenance tasks can be done manually. You can also enable [automatic agent upgrades (preview)](#enable-automatic-agent-upgrade-preview) or look for other places to automate tasks, reducing operational error and expenses.

The Azure Connected Machine agent is updated regularly to address bug fixes, stability enhancements, and new functionality. [Azure Advisor](/azure/advisor/advisor-overview) identifies resources that aren't using the latest version of the machine agent and recommends that you upgrade to the latest version. It notifies you when you select the Azure Arc-enabled server by presenting a banner on the **Overview** page, or when you access Advisor through the Azure portal.

The Azure Connected Machine agent for Windows and Linux can be upgraded to the latest release manually or automatically, depending on your requirements. Installing, upgrading, or uninstalling the Azure Connected Machine Agent doesn't require you to restart your server.

This article describes how to perform various operations related to the Connected Machine agent and your Arc-enabled servers.

> [!TIP]
> For command line reference information, see the [`azcmagent` CLI documentation](azcmagent.md).

## Install a specific version of the agent

We generally recommend using the [most recent version](agent-release-notes.md) of the Azure Connected Machine agent. However, if you need to run an older version of the agent for any reason, uninstall the current version and then install the target version. If your machine is already connected to Azure Arc, you do not need to disconnect the machine. Only Connected Machine agent versions released within the last year are officially supported by the product group.

Follow these instructions to install a specific version of the Azure Connected Machine agent.

### [Windows](#tab/windows)

You can find links to releases of the Windows agents below the heading of each [release note](agent-release-notes.md). If you're looking for an agent version that's more than six months old, see the [release notes archive](agent-release-notes-archive.md).

### [Linux - apt](#tab/linux-apt)

1. If you haven't already, configure your package manager with the [Linux Software Repository for Microsoft Products](/linux/packages).
1. Search for available agent versions with `apt-cache`:

   ```bash
   sudo apt-cache madison azcmagent
   ```

1. Find the version you want to install, replace `VERSION` in the following command with the full (4-part) version number, and run the command to install the agent:

   ```bash
   sudo apt install azcmagent=VERSION
   ```

   For example, to install version 1.28, the install command is:

   ```bash
   sudo apt install azcmagent=1.28.02260.736
   ```

### [Linux - yum](#tab/linux-yum)

1. If you haven't already, configure your package manager with the [Linux Software Repository for Microsoft Products](/linux/packages).
1. Search for available agent versions with `yum list`:

   ```bash
   sudo yum list azcmagent --showduplicates
   ```

1. Find the version you want to install, replace `VERSION` in the following command with the full (4-part) version number, and run the command to install the agent:

   ```bash
   sudo yum install azcmagent-VERSION
   ```

   For example, to install version 1.28, the install command would look like:

   ```bash
   sudo yum install azcmagent-1.28.02260-755
   ```

### [Linux - zypper](#tab/linux-zypper)

1. If you haven't already, configure your package manager with the [Linux Software Repository for Microsoft Products](/linux/packages).
1. Search for available agent versions with `zypper search`:

   ```bash
   sudo zypper search -s azcmagent
   ```

1. Find the version you want to install, replace `VERSION` in the following command with the full (4-part) version number, and run the command to install the agent:

   ```bash
   sudo zypper install -f azcmagent-VERSION
   ```

   For example, to install version 1.28, the install command would look like:

   ```bash
   sudo zypper install -f azcmagent-1.28.02260-755
   ```

---

## Enable automatic agent upgrade (preview)

Starting with version 1.57 of the Azure Connected Machine agent, you can configure the agent to automatically upgrade itself to the latest version. This feature is currently in public preview and is only available in the Azure public cloud.

When you enable automatic upgrades, your agent is scheduled to be upgraded within one version of the latest release. To maintain stability across regions and minimize disruptions, upgrades are rolled out across batches, with all upgrades initiated during off-peak hours. If the upgrade doesn't complete successfully, the agent will reattempt the automatic upgrade periodically until it succeeds.

To enable automatic upgrades when connecting your machine to Azure Arc, use the `--enable-automatic-upgrade` flag in the `azcmagent connect` command. For example:

```bash
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "eastus" --enable-automatic-upgrade
```

You can also use [Azure Policy](/azure/governance/policy/overview) to assign the [Configure Azure Arc-enabled Servers to enable automatic upgrades](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9dfba6f-7430-4214-a666-342b3d3d0d62) policy. This policy enables automatic agent upgrade to servers at scale across your environment.

To enable automatic upgrades on an existing Arc-enabled server, set the `enableAutomaticUpgrade` property to `true` by using Azure CLI ([Windows](/cli/azure/install-azure-cli-windows) or [Linux](/cli/azure/install-azure-cli-linux)) or Azure PowerShell.

The following example shows how to configure automatic agent upgrades with Azure CLI.

```azurecli
# Set your target subscription
az account set --subscription "YOUR SUBSCRIPTION"

# Enable automatic upgrades on a single Arc-enabled server
az rest 
--method PATCH 
--url "https://management.azure.com/subscriptions/<SUB_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.HybridCompute/machines/<MACHINE_NAME>?api-version=2024-05-20-preview" 
--headers "Content-Type=application/json" 
--body '{"properties":{"agentUpgrade":{"enableAutomaticUpgrade":true}}}'
```

The following example shows how to configure automatic agent upgrades by using PowerShell.

```azurepowershell
Set-AzContext -Subscription "YOUR SUBSCRIPTION"

$params = @{
  ResourceGroupName = "YOUR RESOURCE GROUP"
  ResourceProviderName = "Microsoft.HybridCompute"
  ResourceType = "Machines"
  ApiVersion = "2024-05-20-preview"
  Name = "YOUR MACHINE NAME"
  Method = "PATCH"
  Payload = '{"properties":{"agentUpgrade":{ "enableAutomaticUpgrade":true}}}'
}
Invoke-AzRestMethod @params
```

## Additional agent upgrade methods

The following table describes additional methods supported to perform agent upgrades:

| Operating system | Upgrade method |
|------------------|----------------|
| Windows | Manually<br> Microsoft Update |
| Ubuntu | [apt](https://help.ubuntu.com/lts/serverguide/apt.html) |
| Red Hat/Oracle Linux/Amazon Linux | [yum](https://access.redhat.com/articles/yum-cheat-sheet) |
| SUSE Linux Enterprise Server | [zypper](https://en.opensuse.org/SDB:Zypper_usage_11.3) |

### [Windows](#tab/windows)

The latest version of the Azure Connected Machine agent for Windows-based machines can be obtained from:

* Microsoft Update
* [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=AzureConnectedMachineAgent)
* [Microsoft Download Center](https://aka.ms/AzureConnectedMachineAgent)

#### Microsoft Update configuration

The recommended way to keep the Windows agent version up to date is to automatically obtain the latest version through Microsoft Update. This lets you use your existing update infrastructure (such as Microsoft Configuration Manager or Windows Server Update Services) and include Azure Connected Machine agent updates with your regular OS update schedule.

Windows Server doesn't check for updates in Microsoft Update by default. To receive automatic updates for the Azure Connected Machine Agent, you must configure the Windows Update client on the machine to check for other Microsoft products.

For Windows Servers that belong to a workgroup and connect to the Internet to check for updates, you can enable Microsoft Update by running the following commands in PowerShell as an administrator:

```powershell
$ServiceManager = (New-Object -com "Microsoft.Update.ServiceManager")
$ServiceID = "7971f918-a847-4430-9279-4a52d1efe18d"
$ServiceManager.AddService2($ServiceId,7,"")
```

For Windows Servers that belong to a domain and connect to the Internet to check for updates, you can configure this setting at-scale using Group Policy:

1. Sign into a computer used for server administration with an account that can manage Group Policy Objects (GPO) for your organization.
1. Open the [**Group Policy Management Console**](/windows-server/identity/ad-ds/manage/group-policy/group-policy-management-console).
1. Expand the forest, domain, and organizational unit to select the appropriate scope for your new GPO. If you already have a GPO you wish to modify, skip to step 6.
1. Right-click the container and select **Create a GPO in this domain, and Link it here...**.
1. Provide a name for your policy such as "Enable Microsoft Update".
1. Right-click the policy and select **Edit**.
1. Navigate to **Computer Configuration > Administrative Templates > Windows Components > Windows Update**.
1. Select the **Configure Automatic Updates** setting to edit it.
1. Select the **Enabled** radio button to allow the policy to take effect.
1. At the bottom of the **Options** section, check the box for **Install updates for other Microsoft products** at the bottom.
1. Select **OK**.

The next time computers in your selected scope refresh their policy, they'll start to check for updates in both Windows Update and Microsoft Update.

For organizations that use Microsoft Configuration Manager or Windows Server Update Services (WSUS) to deliver updates to their servers, you need to configure WSUS to synchronize the Azure Connected Machine Agent packages and approve them for installation on your servers. Follow the guidance for [Windows Server Update Services](/windows-server/administration/windows-server-update-services/manage/setting-up-update-synchronizations#to-specify-update-products-and-classifications-for-synchronization) or [Configuration Manager](/mem/configmgr/sum/get-started/configure-classifications-and-products#to-configure-classifications-and-products-to-synchronize) to add the following products and classifications to your configuration:

* **Product Name**: Azure Connected Machine Agent (select all suboptions)
* **Classifications**: Critical Updates, Updates

Once the updates are being synchronized, you can optionally add the Azure Connected Machine Agent product to your autoapproval rules, so that your servers automatically stay up to date with the latest agent software.

#### To manually upgrade using the Setup Wizard

1. Sign in to the computer with an account that has administrative rights.
1. Download the latest agent installer from https://aka.ms/AzureConnectedMachineAgent
1. Run **AzureConnectedMachineAgent.msi** to start the Setup Wizard.

If the Setup Wizard discovers a previous version of the agent, it upgrades it automatically. When the upgrade completes, the Setup Wizard closes automatically.

#### To upgrade from the command line

If you're unfamiliar with the command-line options for Windows Installer packages, review [Msiexec standard command-line options](/windows/win32/msi/standard-installer-command-line-options) and [Msiexec command-line options](/windows/win32/msi/command-line-options).

1. Sign on to the computer with an account that has administrative rights.
1. Download the latest agent installer from https://aka.ms/AzureConnectedMachineAgent
1. To upgrade the agent silently and create a setup log file in the `C:\Support\Logs` folder, run the following command:

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\azcmagentupgradesetup.log"
    ```

### [Linux - apt](#tab/linux-apt)

Updating the agent on a Linux machine involves two commands; one command to update the local package index with the list of latest available packages from the repositories, and another command to upgrade the local package.

You can download the latest agent package from Microsoft's [package repository](https://packages.microsoft.com/).

> [!NOTE]
> To upgrade the agent, you must have *root* access permissions or an account that has elevated rights using sudo.

1. To update the local package index with the latest changes made in the repositories, run the following command:

    ```bash
    sudo apt update
    ```

2. To upgrade your system, run the following command:

    ```bash
    sudo apt upgrade azcmagent
    ```

Actions of the [apt](https://help.ubuntu.com/lts/serverguide/apt.html) command, such as installation and removal of packages, are logged in the `/var/log/dpkg.log` log file.

### [Linux - yum](#tab/linux-yum)

Updating the agent on a Linux machine involves two commands; one command to update the local package index with the list of latest available packages from the repositories, and another command to upgrade the local package.

You can download the latest agent package from Microsoft's [package repository](https://packages.microsoft.com/).

> [!NOTE]
> To upgrade the agent, you must have *root* access permissions or an account that has elevated rights using sudo.

1. To update the local package index with the latest changes made in the repositories, run the following command:

    ```bash
    sudo yum check-update
    ```

2. To upgrade your system, run the following command:

    ```bash
    sudo yum update azcmagent
    ```

Actions of the [yum](https://access.redhat.com/articles/yum-cheat-sheet) command, such as installation and removal of packages, are logged in the `/var/log/yum.log` log file.

### [Linux - zypper](#tab/linux-zypper)

Updating the agent on a Linux machine involves two commands; one command to update the local package index with the list of latest available packages from the repositories, and another command to upgrade the local package.

You can download the latest agent package from Microsoft's [package repository](https://packages.microsoft.com/).

> [!NOTE]
> To upgrade the agent, you must have *root* access permissions or an account that has elevated rights using sudo.

1. To update the local package index with the latest changes made in the repositories, run the following command:

    ```bash
    sudo zypper refresh
    ```

2. To upgrade your system, run the following command:

    ```bash
    sudo zypper update azcmagent
    ```

Actions of the [zypper](https://en.opensuse.org/Portal:Zypper) command, such as installation and removal of packages, are logged in the `/var/log/zypper.log` log file.

---
