---
title: Remove your SCVMM environment from Azure Arc
description: This article explains the steps to cleanly remove your SCVMM environment from Azure Arc and delete related Azure Arc resources from Azure.
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.topic: how-to
ms.date: 02/09/2026
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
# Customer intent: As an infrastructure admin, I want to cleanly remove my SCVMM environment from Azure Arc.
ms.custom:
  - build-2025
---

# Remove your SCVMM environment from Azure Arc

This article describes how to cleanly remove your SCVMM managed environment from Azure Arc-enabled SCVMM. For SCVMM environments that you no longer want to manage by using Azure Arc-enabled SCVMM, follow these steps:

1. [Remove the Azure Connected Machine agent from SCVMM virtual machines](#remove-the-azure-connected-machine-agent-from-scvmm-virtual-machines).
1. [Remove your SCVMM environment from Azure Arc](#remove-your-scvmm-environment-from-azure-arc).
1. [Remove Azure Arc resource bridge related items in your SCVMM management server](#remove-azure-arc-resource-bridge-related-items-in-your-scvmm-management-server).



## Remove the Azure Connected Machine agent from SCVMM virtual machines

To prevent continued billing of Azure management services after you remove the SCVMM environment from Azure Arc, first cleanly remove the Azure Connected Machine agent from all the Arc-enabled SCVMM virtual machines where you installed it. When you enable guest management on Arc-enabled SCVMM virtual machines, the Azure Connected Machine agent is installed on them.

After you enable guest management, you can install VM extensions on the virtual machines and use Azure management services like the Log Analytics on them. To cleanly remove guest management, follow the steps in the next section to remove any VM extensions from the virtual machine, disconnect the agent, and uninstall the software from your virtual machine. 

>[!NOTE]
> It's important to complete each of these three steps to fully remove all the related software components from your virtual machines.

### Remove VM extensions

If you deployed Azure VM extensions to an Azure Arc-enabled SCVMM VM, uninstall the extensions before disconnecting the agent or uninstalling the software. Uninstalling the Azure Connected Machine agent doesn't automatically remove extensions. If you later connect the VM to Azure Arc again, the extensions aren't recognized.

To uninstall extensions, follow these steps:

1. Sign in to the [Azure Arc center in Azure portal](https://portal.azure.com/#blade/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/overview).

1. Select **SCVMM management servers**.

1. Search and select the VMM management server you want to remove from Azure Arc.

1. Select **Virtual machines** under **SCVMM inventory**.

1. Search and select the virtual machine where you enabled Guest Management.

1. Select **Extensions**.

1. Select the extensions and select **Uninstall**.

### Disconnect the agent from Azure Arc

When you disconnect the agent, it clears the local state of the agent and removes agent information from Microsoft's systems. To disconnect the agent, sign in and run the following command as an administrator or root account on the virtual machine.

```powershell
    azcmagent disconnect --force-local-only
```

### Uninstall the agent

#### [For Windows virtual machines](#tab/for-windows-virtual-machines)

To uninstall the Windows agent from the machine, follow these steps:

1. Sign in to the computer by using an account that has administrator permissions.
1. In Control Panel, select **Programs and Features**.
1. In **Programs and Features**, select **Azure Connected Machine Agent**, select **Uninstall**, and select **Yes**.
1. Delete the `C:\Program Files\AzureConnectedMachineAgent` folder.

#### [For Linux virtual machines](#tab/for-linux-virtual-machines)

To uninstall the Linux agent, the command to use depends on the Linux operating system. You must have `root` access permissions or your account must have elevated rights by using sudo.

- For Ubuntu, run the following command:

  ```bash
  sudo apt purge azcmagent
  ```

- For RHEL and Oracle Linux, run the following command: 

    ```bash
    sudo yum remove azcmagent
    ```

- For SLES, run the following command:

     ```bash
    sudo zypper remove azcmagent
    ```

---

## Remove your SCVMM environment from Azure Arc

You can remove your SCVMM resources from Azure Arc by using either the deboarding script or manual steps.

### Remove SCVMM managed resources from Azure Arc by using deboarding script

Download the [deboarding script](https://download.microsoft.com/download/a/d/b/adb5650c-5c90-4e94-8a93-2a4707c2020a/arcscvmm-deboard-windows.ps1) to clean up all the Arc-enabled SCVMM resources. The script removes all the Azure resources, including SCVMM management server, custom location, virtual machines, virtual templates, hosts, clusters, resource pools, datastores, virtual networks, Azure Resource Manager (ARM) resource of Appliance, and the appliance VM running on the SCVMM management server.

#### Run the script

To run the deboarding script, follow these steps:

##### Windows
1.	Open a PowerShell window as an Administrator and go to the folder where you downloaded the PowerShell script.

1.	Run the following command to allow the script to run because it's an unsigned script. If you close the session before you complete all the steps, run this command again for the new session.

    ```powershell-interactive
    Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
    ```
1. Run the script.

    ```powershell-interactive
    ./arcvmm-deboard-windows.ps1
    ```

#### Inputs for the script

The essential inputs required for the script are:

- **vmmServerId**: The Azure resource ID of the SCVMM management server resource. 

    For example: */subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/Synthetics/providers/Microsoft.ScVmm/VMMServers/scvmmserverresource*

- **ApplianceConfigFilePath (optional)**: Path to kubeconfig, output from the deploy command. Providing applianceconfigfilepath also deletes the appliance VM running on the SCVMM management server.

- **Force**: Using the Force flag deletes all the Azure resources without reaching resource bridge. Use this option if resource bridge VM isn't in running state.

### Remove SCVMM managed resources from Azure manually

If you aren't using the deboarding script, follow these steps to remove the SCVMM resources manually:

>[!NOTE]
>When you enable SCVMM resources in Azure, you create an Azure resource that represents them. Before you can delete the SCVMM management server resource in Azure, you must delete all the Azure resources that represent your related SCVMM resources.

1. Sign in to the [Azure portal](https://portal.azure.com/) and go to [Azure Arc center](https://portal.azure.com/#blade/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/overview).

1. Select **SCVMM management servers**.

1. Search and select the SCVMM management server you want to remove from Azure Arc.

1. Select **Virtual machines** under **SCVMM inventory**.

1. Select all the VMs that have **Virtual hardware management** value as **Enabled**.

1. Select **Remove from Azure**.

    This action only removes these resource representations from Azure. The resources continue to remain in your SCVMM management server.

1. Perform steps 4, 5, and 6 for **Clouds**, **VM networks**, and **VM templates** by performing **Remove from Azure** operation for resources with **Azure Enabled** value as **Yes**.

1. Once the deletion is complete, select **Overview**.

1. Note the **Custom location** and the **Azure Arc Resource bridge** resource in the **Essentials** section.

1. Select **Remove from Azure** to remove the SCVMM management server resource from Azure.

1. Go to the noted **Custom location** resource and select **Delete**.

1. Go to the noted **Azure Arc Resource bridge** resource and select **Delete**.

At this point, you remove all your Arc-enabled SCVMM resources from Azure.

## Remove Azure Arc resource bridge related items in your SCVMM management server

During onboarding, to create a connection between your SCVMM management server and Azure, you deploy an Azure Arc resource bridge in your SCVMM managed environment. As the last step, delete the resource bridge VM and the VM template you created during onboarding.

You can find both the virtual machine and the template on the *resource pool/cluster/host/cloud* that you provided during [Azure Arc-enabled SCVMM onboarding](./quickstart-connect-system-center-virtual-machine-manager-to-arc.md).

## Next step

[Connect your System Center Virtual Machine Manager management server to Azure Arc again](./quickstart-connect-system-center-virtual-machine-manager-to-arc.md).
