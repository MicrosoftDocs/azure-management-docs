---
title: Troubleshoot Azure Arc-enabled servers VM extension issues
description: This article tells how to troubleshoot and resolve issues with Azure VM extensions that arise with Azure Arc-enabled servers.
ms.date: 06/19/2025
ms.topic: troubleshooting
# Customer intent: As a system administrator working with Azure Arc-enabled servers, I want to troubleshoot VM extension issues effectively, so that I can ensure successful deployment and maintenance of extensions in my environment.
---

# Troubleshoot Azure Arc-enabled servers VM extension issues

This article provides information on troubleshooting and resolving issues that may occur while attempting to deploy or remove Azure virtual machine (VM) extensions on Azure Arc-enabled servers. For general information, see [Virtual machine extension management with Azure Arc-enabled servers](./manage-vm-extensions.md).

## General troubleshooting

Data about the state of extension deployments can be retrieved from the Azure portal by [selecting the applicable machine and then selecting **Settings > Extensions**](manage-vm-extensions-portal.md#list-extensions-installed).

For general troubleshooting, try the following steps. These steps apply to all VM extensions.

1. Ensure that the Azure Connected Machine agent (`azcmagent`) is connected and that the dependent services are *running/active*.

    Run the [`azcmagent show`](azcmagent-show.md) command and check the output for the status (*Azure Arc Proxy* can be ignored):

    :::image type="content" source="media/troubleshoot-vm-extensions/dependent-services-status.png" alt-text="Screenshot of the table showing the status of dependent services as running or stopped.":::

    If any services other than *Azure Arc Proxy* are stopped, restart them to resume extension operations.

1. Retry extension installation.

    Extensions can get stuck in **Failed** or other states for various reasons. If an extension's status isn't listed as **Succeeded**, remove the extension and install it again. The following Azure PowerShell command can be used to remove an extension:

    ```powershell
    Remove-AzConnectedMachineExtension -Name <Extension Name> -ResourceGroupName <RG Name> -MachineName <Machine Name>
    ```

1. Check the Guest agent log and review activity from when your extension was being provisioned. For Windows, check in `%SystemDrive%\ProgramData\GuestConfig\ext_mgr_logs`, and for Linux check in `/var/lib/GuestConfig/ext_mgr_logs`.

1. Check the extension logs for the specific extension for more details.

    For Windows machines:
    - Logs are stored in `C:\ProgramData\GuestConfig`
    - Extension settings and status files reside in `C:\Packages\Plugins`

    For Linux machines:
    - Logs are stored in `/var/lib/GuestConfig`
    - Extension settings and status files reside in `/var/lib/waagent`

    Extension service logs are written to `…GuestConfig\ext_mgr_logs\gc_ext.log`. Errors regarding downloading or verifying the packages are shown there.  

1. Check extension-specific documentation troubleshooting sections for error codes, known issues, or other details. You can find documentation for many extensions in the [extensions table](manage-vm-extensions.md#extensions).

1. Review the system logs. Check for other operations that could interfere with the extension, such as a long-running installation of another application that requires exclusive package manager access.

## Known issues

### HandlerManifest.json file does not exist for extension

The extension is stuck in a `Deleting` state. In the extension service log (`gc_ext.log`), you see the following error:

```
HandlerManifest.json file does not exist for extension
```

**Analysis**

The extension is missing the HandlerManifest.json file. This can happen if the extension was not uninstalled properly.

**Solution**

1. Manually remove the extension from the machine. Some extensions may require additional cleanup steps. Refer to extension-specific documentation in the [extensions table](manage-vm-extensions.md#extensions) for further guidance.

    For Windows machines:
    - Navigate to `C:\Packages\Plugins\`
    - Delete the folder corresponding to the extension
    
    For Linux machines:
    - Navigate to `/var/lib/waagent/`
    - Delete the folder corresponding to the extension

1. Uninstall the extension from Azure and then reinstall it.

## Next steps

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Open a support request to get assistance. For more information, see [Create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
