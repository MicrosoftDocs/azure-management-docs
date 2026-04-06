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

## Extension path has the noexec flag set

While deploying an extension to your Arc-enabled server, you may encounter the following error: 

```
Extension failed to install. Extension returned non-zero exit code for Install: 64. Extension error 
output: Extension path '<full_path>' has the 'noexec' flag set. Extension exit code: 64
```

This error occurs when the Azure Arc agent attempts to install or run an extension from a file system path that is mounted with the `noexec` flag - in this case, the extension path indicated in the error message. The `noexec` mount option prevents binaries or scripts from being executed from that file system. Extensions must execute installation scripts as part of setup; therefore the installation fails when the extension working directory resides on a `noexec` mount. 

This is most commonly seen in hardened Linux environments where paths such as `/var`, `/var/lib`, `/opt`, or custom mount points are explicitly mounted with `noexec` for security reasons. To unblock extension installation, you should ensure that the file system used by Azure Arc for extension installation allows execution.

**Remount the filesystem with `exec`**
To unblock extension installation, update the mount configuration so that the file system hosting the extension path allows execution. If permitted by your security policy, remount the affected filesystem without the `noexec` flag using the command below. 

```bash
sudo mount -o remount,exec <mount-point>
```

If the mount is defined in `/etc/fstab`, update the entry to remove `noexec` and remount the file system to make the change persistent across reboots; otherwise, you may need to remount whenever an extension update is needed. Only apply this change to filesystems where executing binaries is acceptable under your organization’s security requirements.

After updating the mount configuration, retry the extension installation.


## Known issues

### HandlerManifest.json file does not exist for extension

The extension is stuck in a `Deleting` state. In the extension service log (`gc_ext.log`), you see the following error:

```
HandlerManifest.json file does not exist for extension
```

**Analysis**

The extension is missing the HandlerManifest.json file. This can happen if the extension was not uninstalled properly.

**Solution**

1. To remove the extension, use [`az connectedmachine extension delete`](/cli/azure/connectedmachine/extension#az-connectedmachine-extension-delete) with the `--extension-name`, `--machine-name`, and `--resource-group` parameters.

2. If the extension is still in the same state, attempt to manually remove the extension from the machine. Some extensions may require additional cleanup steps. Refer to extension-specific documentation in the [extensions table](manage-vm-extensions.md#extensions) for further guidance.

    For Windows machines:
    - Navigate to `C:\Packages\Plugins\`
    - Delete the folder corresponding to the extension
    
    For Linux machines:
    - Navigate to `/var/lib/waagent/`
    - Delete the folder corresponding to the extension

3. Uninstall the extension from Azure and then reinstall it.


## Next steps

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Open a support request to get assistance. For more information, see [Create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
