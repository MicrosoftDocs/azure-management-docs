---
title:  Manage and maintain the Azure Connected Machine agent
description: This article describes the different management tasks that you typically perform during the lifecycle of the Azure Connected Machine agent.
ms.date: 08/13/2025
ms.topic: how-to
# Customer intent: As a system administrator, I want to manage the lifecycle of the Azure Connected Machine agent, including installation, upgrades, and configurations, so that I can ensure optimal performance and reliability of my Azure-connected servers.
---

# Manage and maintain the Connected Machine agent

After you deploy the Azure Connected Machine agent, you may need to reconfigure the agent, upgrade it, remove it, or make other changes. These routine maintenance tasks can be done manually. You can also enable [automatic agent upgrades (preview)](#automatic-agent-upgrade-preview) or look for other places to automate tasks, reducing operational error and expenses.

This article describes how to perform various operations related to the Connected Machine agent and your Arc-enabled servers.

> [!TIP]
> For command line reference information, see the [`azcmagent` CLI documentation](azcmagent.md).

## Install a specific version of the agent

We generally recommend using the [most recent version](agent-release-notes.md) of the Azure Connected Machine agent. However, if you need to run an older version of the agent for any reason, you can follow these instructions to install a specific version of the agent. Only Connected Machine agent versions released within the last year are officially supported by the product group.

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

## Upgrade the agent

The Azure Connected Machine agent is updated regularly to address bug fixes, stability enhancements, and new functionality. [Azure Advisor](/azure/advisor/advisor-overview) identifies resources that aren't using the latest version of the machine agent and recommends that you upgrade to the latest version. It notifies you when you select the Azure Arc-enabled server by presenting a banner on the **Overview** page, or when you access Advisor through the Azure portal.

The Azure Connected Machine agent for Windows and Linux can be upgraded to the latest release manually or automatically, depending on your requirements. Installing, upgrading, or uninstalling the Azure Connected Machine Agent doesn't require you to restart your server.

### Automatic agent upgrade (preview)

Starting with version 1.48 of the Azure Connected Machine agent, you can configure the agent to automatically upgrade itself to the latest version. This feature is currently in public preview and is only available in the Azure public cloud.

To enable automatic upgrades, set the `enableAutomaticUpgrade` property to `true`. Once you do so, your agents will be upgraded within one version of the latest release, with batches rolled out in order to maintain stability across regions.

The following example shows how to configure automatic agent upgrades by using Azure PowerShell.

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

### Additional agent upgrade methods

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

## Uninstall the agent

For servers you no longer want to manage with Azure Arc-enabled servers, follow these steps to remove any VM extensions from the server, disconnect the agent, and uninstall the software from your server. It's important to complete all of these steps to fully remove all related software components from your system.

### Remove VM extensions

If you deployed Azure VM extensions to an Azure Arc-enabled server, you must uninstall all extensions before disconnecting the agent or uninstalling the software. Uninstalling the Azure Connected Machine agent doesn't automatically remove extensions, and these extensions won't be recognized if you reconnect the server to Azure Arc.

For guidance on how to list and remove any extensions on your Azure Arc-enabled server, see the following resources:

* [Manage VM extensions with the Azure portal](manage-vm-extensions-portal.md)
* [Manage VM extensions with Azure PowerShell](manage-vm-extensions-powershell.md#)
* [Manage VM extensions with Azure CLI](manage-vm-extensions-cli.md)

### Disconnect the server from Azure Arc

After you remove all extensions from your server, the next step is to disconnect the agent. Doing so deletes the corresponding Azure resource for the server and clears the local state of the agent.

To disconnect the agent, run the `azcmagent disconnect` command as an administrator on the server. You're prompted to sign in with an Azure account that has permission to delete the resource in your subscription. If the resource has already been deleted in Azure, pass an additional flag to clean up the local state: `azcmagent disconnect --force-local-only`.

If your Administrator and Azure accounts are different, you may encounter issues with the sign-in prompt defaulting to the Administrator account. To resolve these issues, run the `azcmagent disconnect --use-device-code` command. You're prompted to sign in with an Azure account that has permission to delete the resource.

> [!CAUTION]
> When disconnecting the agent from Arc-enabled VMs running on Azure Local, use only the `azcmagent disconnect --force-local-only` command. Using the command without the `–force-local-only` flag can cause your Arc VM on Azure Local to be deleted both from Azure and on-premises.

### Uninstall the agent

Finally, you can remove the Connected Machine agent from the server.

### [Windows](#tab/windows)

Both of the following methods remove the agent, but they don't remove the *C:\Program Files\AzureConnectedMachineAgent* folder on the machine.

#### Uninstall from Control Panel

Follow these steps to uninstall the Windows agent from the machine:

1. Sign in to the computer with an account that has administrator permissions.
1. In **Control panel**, select **Programs and Features**.
1. In **Programs and Features**, select **Azure Connected Machine Agent**, select **Uninstall**, and then select **Yes**.

You can also delete the Windows agent directly from the agent setup wizard. Run the **AzureConnectedMachineAgent.msi** installer package to do so.

#### Uninstall from the command line

You can uninstall the agent manually from the Command Prompt or by using an automated method (such as a script) by following the example below. First you need to retrieve the product code, which is a GUID that is the principal identifier of the application package, from the operating system. The uninstall is performed by using the Msiexec.exe command line - `msiexec /x {Product Code}`.

1. Open the Registry Editor.

2. Under registry key `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall`, look for and copy the product code GUID.

3. Uninstall the agent using Msiexec, as in the following examples:

   * From the command line, enter the following command:

       ```dos
       msiexec.exe /x {product code GUID} /qn
       ```

   * You can perform the same steps using PowerShell:

       ```powershell
       Get-ChildItem -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall | `
       Get-ItemProperty | `
       Where-Object {$_.DisplayName -eq "Azure Connected Machine Agent"} | `
       ForEach-Object {MsiExec.exe /x "$($_.PsChildName)" /qn}
       ```

### [Linux - apt](#tab/linux-apt)

> [!NOTE]
> To uninstall the agent, you must have *root* access permissions or an account that has elevated rights using sudo.

Run the following command to remove the agent:

```bash
sudo apt purge azcmagent
```

### [Linux - yum](#tab/linux-yum)

> [!NOTE]
> To uninstall the agent, you must have *root* access permissions or an account that has elevated rights using sudo.

Run the following command to remove the agent:

```bash
sudo yum remove azcmagent
```

### [Linux - zypper](#tab/linux-zypper)

> [!NOTE]
> To uninstall the agent, you must have *root* access permissions or an account that has elevated rights using sudo.

```bash
sudo zypper remove azcmagent
```
---

## Update or remove proxy settings

To configure the agent to communicate to the service through a proxy server or to remove this configuration after deployment, use one of the methods described here. The agent communicates outbound using the HTTP protocol under this scenario.

Proxy settings can be configured using the `azcmagent config` command or system environment variables. If a proxy server is specified in both the agent configuration and system environment variables, the agent configuration takes precedence and becomes the effective setting. Use `azcmagent show` to view the effective proxy configuration for the agent.

> [!NOTE]
> Azure Arc-enabled servers doesn't support using [Log Analytics gateway](/azure/azure-monitor/agents/gateway) as a proxy for the Connected Machine agent.

### Agent-specific proxy configuration

Agent-specific proxy configuration is the preferred way of configuring proxy server settings. This method is available starting with version 1.13 of the Azure Connected Machine agent. Using agent-specific proxy configuration helps prevent the proxy settings for the Azure Connected Machine agent from interfering with other applications on your system.

> [!NOTE]
> Some extensions deployed to Azure Arc-enabled servers don't inherit the agent-specific proxy configuration. For guidance on configuring proxy settings for extensions, see the documentation for each extension you deploy.

To configure the agent to communicate through a proxy server, run the following command:

```bash
azcmagent config set proxy.url "http://ProxyServerFQDN:port"
```

You can use an IP address or simple hostname in place of the FQDN if your network requires it. If your proxy server runs on port 80, you may omit ":80" at the end.

To check if a proxy server URL is configured in the agent settings, run the following command:

```bash
azcmagent config get proxy.url
```

To stop the agent from communicating through a proxy server, run the following command:

```bash
azcmagent config clear proxy.url
```

You don't need to restart any services when reconfiguring the proxy settings with the `azcmagent config` command.

### Proxy bypass for private endpoints

Starting with agent version 1.15, you can also specify services which should **not** use the specified proxy server. This can help with split-network designs and private endpoint scenarios where you want Microsoft Entra ID and Azure Resource Manager traffic to go through your proxy server to public endpoints, but you want Azure Arc traffic to skip the proxy and communicate with a private IP address on your network.

The proxy bypass feature doesn't require you to enter specific URLs to bypass. Instead, you provide the name of any services that shouldn't use the proxy server. The location parameter refers to the Azure region of the Arc-enabled server.

Proxy bypass value when set to `ArcData` only bypasses the traffic of the Azure extension for SQL Server and not the Arc agent.

| Proxy bypass value | Affected endpoints |
| --------------------- | ------------------ |
| `AAD` | `login.windows.net`</br>`login.microsoftonline.com`</br> `pas.windows.net` |
| `ARM` | `management.azure.com` |
| `AMA` | `global.handler.control.monitor.azure.com`</br>`<virtual-machine-region-name>.handler.control.monitor.azure.com`</br> `<log-analytics-workspace-id>.ods.opinsights.azure.com`</br>`management.azure.com`</br>`<virtual-machine-region-name>.monitoring.azure.com`</br>`<data-collection-endpoint>.<virtual-machine-region-name>.ingest.monitor.azure.com` |
| `Arc` | `his.arc.azure.com`</br>`guestconfiguration.azure.com` |
| `ArcData` <sup>1</sup> | `*.<region>.arcdataservices.com`|

<sup>1</sup> The proxy bypass value `ArcData` is available starting with Azure Connected Machine agent version 1.36 and Azure Extension for SQL Server version 1.1.2504.99. Earlier versions include the SQL Server enabled by Azure Arc endpoints in the "Arc" proxy bypass value.

To send Microsoft Entra ID and Azure Resource Manager traffic through a proxy server, but skip the proxy for Azure Arc traffic, run the following command:

```bash
azcmagent config set proxy.url "http://ProxyServerFQDN:port"
azcmagent config set proxy.bypass "Arc"
```

To provide a list of services, separate the service names by commas:

```bash
azcmagent config set proxy.bypass "ARM,Arc"
```

To clear the proxy bypass, run the following command:

```bash
azcmagent config clear proxy.bypass
```

You can view the effective proxy server and proxy bypass configuration by running `azcmagent show`.

### Windows environment variables

On Windows, the Azure Connected Machine agent first checks the `proxy.url` agent configuration property (starting with agent version 1.13), and then the system-wide `HTTPS_PROXY` environment variable, to determine which proxy server to use. If both are empty, no proxy server is used, even if the default Windows system-wide proxy setting is configured.

We recommend using the agent-specific proxy configuration instead of the system environment variable.

To set the proxy server environment variable, run the following commands:

```powershell
# If a proxy server is needed, execute these commands with the proxy URL and port.
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://ProxyServerFQDN:port", "Machine")
$env:HTTPS_PROXY = [System.Environment]::GetEnvironmentVariable("HTTPS_PROXY", "Machine")
# For the changes to take effect, the agent services need to be restarted after the proxy environment variable is set.
Restart-Service -Name himds, ExtensionService, GCArcService
```

To configure the agent to stop communicating through a proxy server, run the following commands:

```powershell
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", $null, "Machine")
$env:HTTPS_PROXY = [System.Environment]::GetEnvironmentVariable("HTTPS_PROXY", "Machine")
# For the changes to take effect, the agent services need to be restarted after the proxy environment variable removed.
Restart-Service -Name himds, ExtensionService, GCArcService
```

### Linux environment variables

On Linux, the Azure Connected Machine agent first checks the `proxy.url` agent configuration property (starting with agent version 1.13), and then the `HTTPS_PROXY` environment variable set for the himds, GC_Ext, and GCArcService daemons. An included script configures systemd's default proxy settings so that the Azure Connected Machine agent and all other services on the machine use a specified proxy server.

To configure the agent to communicate through a proxy server, run the following command:

```bash
sudo /opt/azcmagent/bin/azcmagent_proxy add "http://ProxyServerFQDN:port"
```

To remove the environment variable, run the following command:

```bash
sudo /opt/azcmagent/bin/azcmagent_proxy remove
```

### Migrate from environment variables to agent-specific proxy configuration

If you're already using environment variables to configure the proxy server for the Azure Connected Machine agent, and you want to migrate to the agent-specific proxy configuration based on local agent settings, follow these steps:

1. [Upgrade the Azure Connected Machine agent](#upgrade-the-agent) to the latest version.

1. Configure the agent with your proxy server information by running `azcmagent config set proxy.url "http://ProxyServerFQDN:port"`.

1. Remove the unused environment variables by following the steps for [Windows](#windows-environment-variables) or [Linux](#linux-environment-variables).

## Rename an Azure Arc-enabled server resource

When you change the name of a Linux or Windows machine connected to Azure Arc-enabled servers, the new name isn't recognized automatically, because the resource name in Azure is immutable. As with other Azure resources, to use the new name, you must delete the resource in Azure and then recreate it.

For Azure Arc-enabled servers, before you rename the machine, you must remove the VM extensions:

1. List the VM extensions installed on the machine and note their configuration using the [Azure portal](manage-vm-extensions-portal.md#list-extensions-installed), [Azure CLI](manage-vm-extensions-cli.md#list-extensions-installed), or [Azure PowerShell](manage-vm-extensions-powershell.md#list-extensions-installed).

2. Remove all VM extensions installed on the machine by using the [Azure portal](manage-vm-extensions-portal.md#remove-extensions), [Azure CLI](manage-vm-extensions-cli.md#remove-extensions), or [Azure PowerShell](manage-vm-extensions-powershell.md#remove-extensions).

3. Use the **azcmagent** tool with the [Disconnect](azcmagent-disconnect.md) parameter to disconnect the machine from Azure Arc and delete the machine resource from Azure. You can run this tool manually while logged on interactively, with a Microsoft identity [access token](/azure/active-directory/develop/access-tokens), or with a [service principal](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).

    Disconnecting the machine from Azure Arc-enabled servers doesn't remove the Connected Machine agent, and you don't need to remove the agent as part of this process.

4. Re-register the Connected Machine agent with Azure Arc-enabled servers. Run the `azcmagent` tool with the [Connect](azcmagent-connect.md) parameter to complete this step. The agent defaults to using the computer's current hostname, but you can choose your own resource name by passing the `--resource-name` parameter to the connect command.

5. Redeploy the VM extensions that were originally deployed to the machine from Azure Arc-enabled servers. If you deployed the Azure Monitor for VMs (insights) agent using an Azure Policy definition, the agents are redeployed after the next [evaluation cycle](/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

## Investigate Azure Arc-enabled server disconnection

The Connected Machine agent [sends a regular heartbeat message](overview.md#agent-status) to Azure every five minutes. If an Arc-enabled server stops sending heartbeats to Azure for longer than 15 minutes, it can mean that the server is offline, the network connection is blocked, or the agent isn't running.

Develop a plan for responding and investigating these incidents, including setting up resource Health alerts to get notified when such incidents occur. For more information, see [Create Resource Health alerts in the Azure portal](/azure/service-health/resource-health-alert-monitor-guide).]

## Related content

* Find troubleshooting information in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md).
* Understand how to [plan and deploy Azure Arc-enabled servers](plan-at-scale-deployment.md) at any scale and implement centralized management and monitoring.
* Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview), for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verifying the machine is reporting to the expected Log Analytics workspace, enable monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.
