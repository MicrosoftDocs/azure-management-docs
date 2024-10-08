---
title: Troubleshoot Azure Arc-enabled servers VM extension issues
description: This article tells how to troubleshoot and resolve issues with Azure VM extensions that arise with Azure Arc-enabled servers.
ms.date: 10/02/2024
ms.topic: troubleshooting
---

# Troubleshoot Azure Arc-enabled servers VM extension issues

This article provides information on troubleshooting and resolving issues that may occur while attempting to deploy or remove Azure VM extensions on Azure Arc-enabled servers. For general information, see [Manage and use Azure VM extensions](./manage-vm-extensions.md).

## General troubleshooting

Data about the state of extension deployments can be retrieved from the Azure portal by [selecting the applicable machine and then selecting **Settings>Extensions**](manage-vm-extensions-portal.md#list-extensions-installed).

The following troubleshooting steps apply to all VM extensions.

1. Ensure that the Azure Connected Machine agent (azcmagent) is connected and that the dependent services are *running/active*.

    Run the [**azcmagent show**](azcmagent-show.md) command and check the output for the status (*Azure Arc Proxy* can be ignored):
    
    :::image type="content" source="media/troubleshoot-vm-extensions/dependent-services-status.png" alt-text="Screenshot of the table showing the status of dependent services as running or stopped.":::

    If the services are stopped, restart the services to resume extension operations.

1. Retry extension installation.

    Extensions can stall in Creating/Updating or Fail status for various reasons. In this case, remove the extension and install it again. To remove an extension, use the following command:

    ```powershell
    Remove-AzConnectedMachineExtension -Name <Extension Name> -ResourceGroupName <RG Name> -MachineName <Machine Name>
    ```

1. Check the Guest agent log for the activity when your extension was being provisioned. For Windows, check in `%SystemDrive%\ProgramData\GuestConfig\ext_mgr_logs`, and for Linux check in `/var/lib/GuestConfig/ext_mgr_logs`.

1. Check the extension logs for the specific extension for more details.

    For Windows machines:
    - Logs reside in `C:\ProgramData\GuestConfig`
    - Extension settings and status files reside in `C:\Packages\Plugins`

    For Linux machines:
    - Logs reside in `/var/lib/GuestConfig`
    - Extension settings and status files reside in `/var/lib/waagent`

    Extension service logs are written to `…GuestConfig\ext_mgr_logs\gc_ext.log`. Errors regarding downloading or verifying the packages are shown there.  

1. Check extension-specific documentation troubleshooting sections for error codes, known issues etc. More troubleshooting information for each extension can be found in the **Troubleshoot and support** section in the overview for the extension. This includes the description of error codes written to the log. The extension articles are linked in the [extensions table](manage-vm-extensions.md#extensions).

1. Look at the system logs. Check for other operations that may have interfered with the extension, such as a long running installation of another application that required exclusive package manager access.


<!--
## Troubleshooting specific extension scenarios

### VM Insights

- Enabling VM Insights for an Azure Arc-enabled server installs the Dependency and Log Analytics agent. On a slow machine or one with a slow network connection, it is possible to see timeouts during the installation process. Microsoft is taking steps to address this in the Connected Machine agent to help improve this condition. In the interim, a retry of the installation may succeed.

### Log Analytics agent for Linux

- The Log Analytics agent version 1.13.9 (corresponding extension version is 1.13.15) is not correctly marking uploaded data with the resource ID of the Azure Arc-enabled server. Although logs are being sent to the service, when you try to view the data from the selected enabled server after selecting **Logs** or **Insights**, no data is returned. You can view its data by running queries from Azure Monitor Logs or from Azure Monitor for VMs, which are scoped to the workspace.

- Some distributions are not currently supported by the Log Analytics agent for Linux. The agent requires additional dependencies to be installed, including Python 2. Review the support matrix and prerequisites [here](/azure/azure-monitor/agents/agents-overview#supported-operating-systems).

- Error code 52 in the status message indicates a missing dependency. Check the output and logs for more information about which dependency is missing.

- If an installation fails, review the **Troubleshoot and support** section in the overview for the extension. In most cases, there is an error code included in the status message. For the Log Analytics agent for Linux, status messages are explained [here](/azure/virtual-machines/extensions/oms-linux#troubleshoot-and-support), along with general troubleshooting information for this VM extension.

-->

## Next steps

If you don't see your problem here or you can't resolve your issue, try one of the following channels for additional support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).

- Connect with [@AzureSupport](https://x.com/azuresupport), the official Microsoft Azure account for improving customer experience. Azure Support connects the Azure community to answers, support, and experts.

- File an Azure support incident. Go to the [Azure support site](https://azure.microsoft.com/support/options/), and select **Get Support**.
