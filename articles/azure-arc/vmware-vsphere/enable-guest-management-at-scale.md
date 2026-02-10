---
title: Install Arc agent at scale for your VMware VMs
description: Learn how to enable guest management at scale for Arc enabled VMware vSphere VMs.
ms.topic: how-to
ms.date: 02/10/2026
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.custom:
  - build-2025
  - sfi-ropc-nochange
# Customer intent: As an IT infrastructure administrator, I want to install Arc agents at scale on VMware VMs, so that I can leverage Azure management capabilities for efficient resource management and operations.
---

# Install Arc agents at scale for your VMware VMs

In this article, you learn how to install Azure connected machine agents for VMware VMs. Installing these agents is a prerequisite to use Azure services for securing, patching, and monitoring your VMs. By using these agents, you can leverage Azure Arc benefits such as Extended Security Updates, pay-as-you-go licensing for Windows Server and SQL servers, and Software Attestation benefits. 

You can install Arc agents on VMware VMs through multiple methods. Choose the method that fits your deployment preferences: 

- Azure portal
- Programmatic methods such as Azure CLI, Azure PowerShell, Azure REST APIs, Azure SDKs, Terraform, Bicep, and ARM templates. The reference section of this documentation repository has information on the exact syntax.
- Out-of-band methods such as using a Service Principal, System Center Configuration Manager script, System Center Configuration Manager custom task sequence, Group policy, and Ansible playbook. 

## Prerequisites

Before you install Arc agents at scale for VMware VMs, ensure the following conditions are met:

- The resource bridge is running.
- The vCenter is in *Connected* state and its associated Azure Arc resource bridge is in a *Running* state.
- You have the *Azure Arc VMware VM Contributor* role or a custom Azure role with permissions to install Arc agents on the target machines.
- All the target machines are:
    - Powered on.
    - Running a [supported operating system](../servers/prerequisites.md#supported-operating-systems).
    - VMware tools are installed on the machines. If you don't install VMware tools, the portal disables the option to enable guest management operation.  
        >[!NOTE]
        >Use the out-of-band methods to install Arc agents if VMware tools aren't installed.  
    - Able to connect through the firewall to communicate over the internet, and [these URLs](../servers/network-requirements.md#urls) aren't blocked.

   > [!NOTE]
      > If you're using a Linux VM, the account must not prompt for login on sudo commands. To override the prompt, from a terminal, run `sudo visudo`, and add `<username> ALL=(ALL) NOPASSWD:ALL` at the end of the file. Ensure you replace `<username>`. <br> <br>If your VM template has these changes incorporated, you don't need to make this change for the VM created from that template.

## Install Arc agents 

# [Azure portal](#tab/azure-portal)

This method works only if VMware tools are installed on the target machines. If VMware tools aren't installed, the portal grays out the **Enable guest management** option. You can install Arc agents by using out-of-band methods.

An administrator can install agents for multiple machines from the Azure portal if the machines share the same administrator credentials.

1. Go to **Azure Arc center** and select **vCenter resource**.

1. Select all the target machines and choose **Enable in Azure** option. 

1. Select **Enable guest management** checkbox to install Arc agents on the selected machines. By using this option, you can use Azure services such as Azure Update Manager, Azure Monitor, Microsoft Defender for Cloud, Azure Policy, Azure Automation, Change Tracking and Inventory, and more to secure, govern, patch, and monitor your virtual machines.

1. If you enable guest management on any of your machines, based on your organization's network policies, choose the connectivity method for the Arc agents that runs in your VMware VMs to connect to Azure. The available options are Public endpoint, Proxy server, and Private endpoint. 
     - To connect the Arc agent through a proxy, provide the proxy server details.
     - To connect the Arc agent through a private endpoint, follow these [steps](../servers/private-link-security.md) to set up Azure private link. 

      >[!NOTE]
      > Private endpoint connectivity is only available for Arc agent to Azure communications. For Arc resource bridge to Azure connectivity, Azure private link isn't supported.

1. Enter the administrator username and password for the machine. For Windows VMs, the account must be part of the local administrators group. For Linux VMs, it must be a root account.

6. Select **Enable** to start the installation of the Arc agent in the specified machines. Once installation is complete, the Guest management column will switch to Enabled for the machines with Arc agent running. You can start using Azure services for these machines. These credentials won't be persisted in Azure. They're used to install the Azure Arc agent and then discarded.

# [Auto Arc-enablement script](#tab/ercenablement-script)

This method works only if VMware tools are installed on the target machines. If VMware tools aren't installed, you can install Arc agents by using out-of-band methods. 

You can automate Arc agent installation by using a helper script that uses the AzCLI command. To enable VMs and install Arc agents at scale, download this [helper script](https://aka.ms/arcvmwarebatchenable). In a single ARM deployment, the helper script can enable and install Arc agents on 200 VMs.  

### Features of the script

- Creates a log file (vmware-batch.log) for tracking its operations.

- Generates a list of Azure portal links to all the deployments created (`all-deployments-<timestamp>.txt`). 

- Creates ARM deployment files (`vmw-dep-<timestamp>-<batch>.json`).

- Can enable up to 200 VMs in a single ARM deployment if guest management is enabled, otherwise enables 400 VMs. 

- Supports running as a cron job to enable all the VMs in a vCenter. 

- Allows for service principal authentication to Azure for automation. 

Before running this script, install Azure CLI and the `connectedvmware` extension. 

### Prerequisites 

Before running this script, install: 

- Azure CLI from [here](/cli/azure/install-azure-cli).

- The `connectedvmware` extension for Azure CLI: Install it by running `az extension add --name connectedvmware`. 

### Usage 

1. Download the script to your local machine. 

1. Open a PowerShell terminal and go to the directory containing the script. 

1. Run the following command to allow the script to run, as it's an unsigned script. If you close the session before you complete all the steps, run this command again for the new session: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`.

1. Run the script with the required parameters. For example, `.\arcvmware-batch-enablement.ps1 -VCenterId "<vCenterId>" -EnableGuestManagement -VMCountPerDeployment 3 -DryRun`. Replace `<vCenterId>` with the ARM ID of your vCenter. 

### Parameters

- `VCenterId`: The ARM ID of the vCenter where the VMs are located. 

- `EnableGuestManagement`: If you specify this switch, the script enables guest management on the VMs. 

- `VMCountPerDeployment`: The number of VMs to enable per ARM deployment. The maximum value is 200 if guest management is enabled, otherwise it's 400. 

- `DryRun`: If you specify this switch, the script only creates the ARM deployment files. Otherwise, the script also deploys the ARM deployments. 

### Running as a Cron Job 

You can set up this script to run as a cron job by using the Windows Task Scheduler. Here's a sample script to create a scheduled task: 

```azurecli
$action = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument '-File "C:\Path\To\vmware-batch-enable.ps1" -VCenterId "<vCenterId>" -EnableGuestManagement -VMCountPerDeployment 3 -DryRun' 
$trigger = New-ScheduledTaskTrigger -Daily -At 3am 
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "EnableVMs" 
```

Replace `<vCenterId>` with the ARM ID of your vCenter. 

To unregister the task, run the following command: 

```azurecli
Unregister-ScheduledTask -TaskName "EnableVMs"
```

# [Out-of-band methods](#tab/Out-of-band)

You can install Arc agents directly on machines without relying on VMware tools or APIs. By using the out-of-band approach, first onboard the machines as Arc-enabled Server resources with the resource type `Microsoft.HybridCompute/machines`. After that, perform the **Link to vCenter** operation to update the machine's `Kind` property as `VMware`, which enables virtual lifecycle operations.  

1. **Connect the machines as Arc-enabled Server resources:** Install Arc agents by using Arc-enabled Server scripts. 

    To install Arc agents at scale, use any of the following automation approaches:

   - [Install Arc agents at scale by using a Service Principal](../servers/onboard-service-principal.md).
   - [Install Arc agents at scale by using Configuration Manager script](../servers/onboard-configuration-manager-powershell.md).
   - [Install Arc agents at scale with a Configuration Manager custom task sequence](../servers/onboard-configuration-manager-custom-task.md).
   - [Install Arc agents at scale by using Group policy](../servers/onboard-group-policy-powershell.md).
   - [Install Arc agents at scale by using Ansible playbook](../servers/onboard-ansible-playbooks.md).

1. **Link Arc-enabled Server resources to the vCenter:** The following commands update the `Kind` property of Hybrid Compute machines to **VMware**. When you link the machines to vCenter, it enables virtual lifecycle operations and power cycle operations (start, stop, and other actions) on the machines.  

   - The following command scans all the Arc for Server machines that belong to the vCenter in the specified subscription. It links the machines with that vCenter. 

     [!INCLUDE [azure-cli-subscription](./includes/azure-cli-subscription.md)]

   - The following command scans all the Arc for Server machines that belong to the vCenter in the specified resource group. It links the machines with that vCenter. 

     [!INCLUDE [azure-cli-all](./includes/azure-cli-all.md)]

   - Use the following command to link an individual Arc for Server resource to vCenter. 

     [!INCLUDE [azure-cli-specified-arc](./includes/azure-cli-specified-arc.md)]

--- 

## Next steps

- [Set up and manage self-service access to VMware resources through Azure RBAC](setup-and-manage-self-service-access.md)
- [Manage and maintain the Azure Connected Machine agent](../servers/manage-agent.md)
- [VM Extension Management with Azure Arc-Enabled Servers](../servers/manage-vm-extensions.md)
