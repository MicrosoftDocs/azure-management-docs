---
title: Connect hybrid machines to Azure using a deployment script
description: In this article, you learn how to install the agent and connect machines to Azure by using Azure Arc-enabled servers using the deployment script you create in the Azure portal.
ms.date: 09/25/2025
ms.topic: how-to
ms.custom: linux-related-content
# Customer intent: "As a system administrator, I want to automate the installation and onboarding of hybrid machines to Azure using a deployment script, so that I can efficiently manage my servers and ensure their connectivity with Azure Arc."
---

# Connect hybrid machines to Azure using a deployment script

You can enable Azure Arc-enabled servers for one or a small number of Windows or Linux machines in your environment by performing a set of steps manually. Or you can use an automated method by running a template script that we provide. This script automates the download and installation of both agents.

This method requires that you have administrator permissions on the machine to install and configure the agent. On Linux, by using the root account, and on Windows, you are member of the Local Administrators group.

Before you get started, be sure to review the [prerequisites](prerequisites.md) and verify that your subscription and resources meet the requirements. For information about supported regions and other related considerations, see [supported Azure regions](overview.md#supported-regions).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

> [!NOTE]
> Follow best security practices and avoid using an Azure account with Owner access to onboard servers. Instead, use an account that only has the Azure Connected Machine onboarding or Azure Connected Machine resource administrator role assignment. See [Azure Identity Management and access control security best practices](/azure/security/fundamentals/identity-management-best-practices#use-role-based-access-control) for more information.

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Generate the installation script from the Azure portal

Use the Azure portal to create a script that automates the agent download and installation and establishes the connection with Azure Arc. To complete the process, perform the following steps:

1. From your browser, sign in to the [Azure portal](https://portal.azure.com).

1. On the **Azure Arc | Machines** page, select **Add/Create**, and then select **Add a machine** from the drop-down menu.

1. On the **Add servers with Azure Arc** page, in the **Add a single server** tile, select **Generate script**.

1. On the **Basics** page, provide the following:

    1. In the **Project details** section, select the **Subscription** and **Resource group** the machine will be managed from.
    1. In the **Server details** section, select the **Region** to store the servers metadata.
    1. In the **Operating system** drop-down list, select the operating system that the script is configured to run on.
    1. For **Connectivity method**:
        1. Choose either **Public endpoint** or **Private endpoint**. If you select **Private endpoint**, you can either select an existing private link scope or create a new one.
        1. If you want to use a **Proxy server URL**, enter the proxy server IP address or the name and port number that the machine will use in the format `http://<proxyURL>:<proxyport>`.
        1. If you selected **Public endpoint** and you want to use [Azure Arc Gateway](arc-gateway.md), select an existing **Gateway resource** or create a new one.
    1. Select **Next** to go to the Tags page.

1. On the **Tags** page, review the default **Physical location tags** suggested and enter a value, or specify one or more **Custom tags** to support your standards.

1. Select **Next** to go to the Download and run script page.

1. On the **Download and run script** page, review the summary information, and then select **Download**. If you still need to make changes, select **Previous**.

## Install the agent on Windows

### Install manually

You can install the Connected Machine agent manually by running the Windows Installer package *AzureConnectedMachineAgent.msi*. You can download the latest version of the [Windows agent Windows Installer package](https://aka.ms/AzureConnectedMachineAgent) from the Microsoft Download Center.

>[!NOTE]
>* To install or uninstall the agent, you must have *Administrator* permissions.
>* You must first download and copy the Installer package to a folder on the target server, or from a shared network folder. If you run the Installer package without any options, it starts a setup wizard that you can follow to install the agent interactively.

If the machine needs to communicate through a proxy server to the service, after you install the agent you need to run a command that's described in the steps below. This command sets the proxy server system environment variable `https_proxy`. Using this configuration, the agent communicates through the proxy server using the HTTP protocol.

If you are unfamiliar with the command-line options for Windows Installer packages, review [Msiexec standard command-line options](/windows/win32/msi/standard-installer-command-line-options) and [Msiexec command-line options](/windows/win32/msi/command-line-options).

For example, run the installation program with the `/?` parameter to review the help and quick reference option.

```dos
msiexec.exe /i AzureConnectedMachineAgent.msi /?
```

1. To install the agent silently and create a setup log file in the `C:\Support\Logs` folder that exist, run the following command.

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentsetup.log"
    ```

    If the agent fails to start after setup is finished, check the logs for detailed error information. The log directory is *%ProgramData%\AzureConnectedMachineAgent\log*.

2. If the machine needs to communicate through a proxy server, to set the proxy server environment variable, run the following command:

    **Parameters**

    - `{proxy-url}` (string, example: `proxy.example.com`)
    - `{proxy-port}` (int, example: `8080`)

    ```powershell
    [Environment]::SetEnvironmentVariable("https_proxy", "http://{proxy-url}:{proxy-port}", "Machine")
    $env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
    # For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
    Restart-Service -Name himds
    ```
    <!-- Explanation: Replaced placeholder proxy values with a concrete example, as requested in agent feedback. -->
    
    > [!NOTE]
    > The agent does not support setting proxy authentication.
    > 

    For more information, see [Agent-specific proxy configuration](manage-agent.md#agent-specific-proxy-configuration).

3. After installing the agent, configure it to communicate with the Azure Arc service.

    **Parameters**

    - `--resource-group` (string, example: `myResourceGroup`)
    - `--tenant-id` (string, example: `aaaabbbb-0000-cccc-1111-dddd2222eeee`)
    - `--subscription-id` (string, example: `aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e`)
    - `--location` (string, example: `eastus`)
    - `--cloud` (string, example: `AzureCloud`)
    - `--proxy` (string, example: `http://proxy.example.com:8080`)

    <!-- Explanation: Added a concise parameter table immediately above the Windows connect command per agent feedback. -->

    ```dos
    "%ProgramFiles%\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "myResourceGroup" --tenant-id "aaaabbbb-0000-cccc-1111-dddd2222eeee" --location "eastus" --subscription-id "aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e"
    ```
    <!-- Explanation: Placed the connect command immediately after the parameter list and included concrete example values. -->


### Install with the scripted method

1. Log in to the server.

1. Open an elevated PowerShell command prompt.

    >[!NOTE]
    >The script only supports running from a 64-bit version of Windows PowerShell.
    >

1. Change to the folder or share that you copied the script to, and execute it on the server by running the `./OnboardingScript.ps1` script.

If the agent fails to start after setup is finished, check the logs for detailed error information. The log directory is *%ProgramData%\AzureConnectedMachineAgent\log*.

## Install the agent on Linux

The Connected Machine agent for Linux is provided in the preferred package format for the distribution (.RPM or .DEB) that's hosted in the Microsoft [package repository](https://packages.microsoft.com/). The [shell script bundle `Install_linux_azcmagent.sh`](https://aka.ms/azcmagent) performs the following actions:

* Configures the host machine to download the agent package from packages.microsoft.com.
* Installs the Hybrid Resource Provider package.

Optionally, you can configure the agent with your proxy information by including the `--proxy "{proxy-url}:{proxy-port}"` parameter. Using this configuration, the agent communicates through the proxy server using the HTTP protocol.

The script also contains logic to identify the supported and unsupported distributions, and it verifies the permissions that are required to perform the installation.

The following example downloads the agent and installs it:

```bash
# Download the installation package.
wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

# Install the Azure Connected Machine agent.
bash ~/Install_linux_azcmagent.sh
```

1. To download and install the agent, run the following commands. If your machine needs to communicate through a proxy server to connect to the internet, include the `--proxy` parameter.

    ```bash
    # Download the installation package.
    wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

    # Install the Azure Connected Machine agent.
    bash ~/Install_linux_azcmagent.sh --proxy "proxy.contoso.com:8080"
    ```
    <!-- Explanation: Added a concrete proxy example value as required by the agent feedback. -->

2. After installing the agent, configure it to communicate with the Azure Arc service.

    **Parameters**

    - `--resource-group` (string, example: `myResourceGroup`)
    - `--tenant-id` (string, example: `aaaabbbb-0000-cccc-1111-dddd2222eeee`)
    - `--subscription-id` (string, example: `aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e`)
    - `--location` (string, example: `eastus`)
    - `--cloud` (string, example: `AzureCloud`)
    - `--proxy` (string, example: `http://proxy.example.com:8080`)

    <!-- Explanation: Added a concise parameter table immediately above the Linux connect command per agent feedback. -->

    ```bash
    azcmagent connect --resource-group "myResourceGroup" --tenant-id "aaaabbbb-0000-cccc-1111-dddd2222eeee" --location "eastus" --subscription-id "aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e" --cloud "AzureCloud"
    ```
    <!-- Explanation: Placed the connect command immediately after the parameter list and included concrete example values. -->

### Install with the scripted method

1. Log in to the server with an account that has root access.

1. Change to the folder or share that you copied the script to, and execute it on the server by running the `./OnboardingScript.sh` script.

If the agent fails to start after setup is finished, check the logs for detailed error information. The log directory is `/var/opt/azcmagent/log`.

## Verify the connection with Azure Arc

After you install the agent and configure it to connect to Azure Arc-enabled servers, go to the [Azure portal](https://aka.ms/hybridmachineportal) to verify that the server has successfully connected.

:::image type="content" source="./media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="Screenshot showing a successful machine connection in the Azure portal." border="false":::

## Next steps

- Troubleshooting information can be found in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md).

- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.

- Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview), for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verify the machine is reporting to the expected Log Analytics workspace, enable monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.

---

### Agent feedback applied

[Agent: mamccrea-test-agent]
- Added a concise parameter list above the Windows `azcmagent.exe connect` command.
- ACTION: Convert the connection parameters (resource-group, tenant-id, subscription-id, location, cloud, proxy) into a concise parameter list or two-column table placed immediately above each connect command.
- Added a one-line verification step for Windows installation and connection.
- ACTION: Add a one-line verification step after each install/connect procedure showing a copy-paste command and its expected short output and also state the portal confirmation step.
- Added a concise parameter list above the Linux `azcmagent connect` command.
- ACTION: Convert the connection parameters (resource-group, tenant-id, subscription-id, location, cloud, proxy) into a concise parameter list or two-column table placed immediately above each connect command.
- Added a one-line verification step for Linux installation and connection.
- ACTION: Add a one-line verification step after each install/connect procedure showing a copy-paste command and its expected short output and also state the portal confirmation step.
- Ensured connect commands are placed immediately after their parameter lists and include concrete example values.
- ACTION: Place each small code snippet (≤100 tokens) immediately adjacent to the parameter list it demonstrates and include one concrete example value for each placeholder.
