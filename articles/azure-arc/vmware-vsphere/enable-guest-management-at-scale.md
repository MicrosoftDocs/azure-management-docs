---
title: Install Arc agent at scale for your VMware VMs
description: Learn how to enable guest management at scale for Arc enabled VMware vSphere VMs. 
ms.topic: how-to
ms.date: 07/02/2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
ms.custom:
  - build-2025
  - sfi-ropc-nochange
# Customer intent: As an IT infrastructure administrator, I want to install Arc agents at scale on VMware VMs, so that I can leverage Azure management capabilities for efficient resource management and operations.
---

# Install Arc agents at scale for your VMware VMs

In this article, you learn how to install Azure connected machine agents for VMware VMs which is a prerequisite to use Azure services for securing, patching, monitoring your VMs and leverage Azure Arc benefits such as Extended Security Updates, pay-as-you-go licensing for Windows Server and SQL servers, and Software Attestation benefits. 

There are multiple avenues available to install Arc agents on VMware VMs which you can leverage based on your deployment preferences: 

- Azure portal
- Programmatic methods such as Azure CLI, Azure PowerShell, Azure REST APIs, Azure SDKs, Terraform, Bicep and ARM templates. The reference section of this documentation repository has information on the exact syntax.
- Out-of-band methods such as using a Service Principal, System Center Configuration Manager script, System Center Configuration Manager custom task sequence, Group policy and Ansible playbook. 

## Prerequisites

Ensure the following before you install Arc agents at scale for VMware VMs:

- The resource bridge must be in running state.
- The vCenter must be in *Connected* state and its associated Azure Arc resource bridge in a *Running* state.
- *Azure Arc VMware VM Contributor* role or a custom Azure role with permissions to install Arc agents on the target machines.
- All the target machines are:
    - Powered on.
    - Running a [supported operating system](../servers/prerequisites.md#supported-operating-systems).
    - VMware tools are installed on the machines. If VMware tools aren't installed, enable guest management operation is grayed out in the portal.  
        >[!Note]
        >You can use the out-of-band methods to install Arc agents if VMware tools aren't installed.  
    - Able to connect through the firewall to communicate over the internet, and [these URLs](../servers/network-requirements.md#urls) aren't blocked.

   > [!NOTE]
   > If you're using a Linux VM, the account must not prompt for login on sudo commands. To override the prompt, from a terminal, run `sudo visudo`, and add `<username> ALL=(ALL) NOPASSWD:ALL` at the end of the file. Ensure you replace `<username>`. <br> <br>If your VM template has these changes incorporated, you won't need to do this for the VM created from that template.

## Install Arc agents 

# [Azure portal](#tab/azure-portal)

This method is applicable only if VMware tools are installed on the target machines. If VMware tools aren't installed, enable guest management operation is grayed out in the portal and Arc agents can be installed through out-of-band methods.

An administrator can install agents for multiple machines from the Azure portal if the machines share the same administrator credentials.

1. Navigate to **Azure Arc center** and select **vCenter resource**.

2. Select all the target machines and choose **Enable in Azure** option. 

3. Select **Enable guest management** checkbox to install Arc agents on the selected machines. This allows you to use Azure services such as Azure Update Manager, Azure Monitor, Microsoft Defender for Cloud, Azure Policy, Azure Automation, Change Tracking and Inventory, etc. to secure, govern, patch and monitor your virtual machines.

4. If you enable guest management on any of your machines, based on your organization's network policies, choose the connectivity method for the Arc agents that runs in your VMware VMs to connect to Azure. The available options are Public endpoint, Proxy server and Private endpoint. 
     - If you want to connect the Arc agent via proxy, provide the proxy server details.
     - If you want to connect Arc agent via private endpoint, follow these [steps](../servers/private-link-security.md) to set up Azure private link. 

      >[!Note]
      > Private endpoint connectivity is only available for Arc agent to Azure communications. For Arc resource bridge to Azure connectivity, Azure private link isn't supported.

5. Provide the administrator username and password for the machine. For Windows VMs, the account must be part of local administrator group; and for Linux VM, it must be a root account.

6. Select **Enable** to start the installation of the Arc agent in the specified machines. Once installation is complete, the Guest management column will switch to Enabled for the machines with Arc agent running. You can start using Azure services for these machines. These credentials won't be persisted in Azure. They're used to install the Azure Arc agent and then discarded.

# [Auto Arc-enablement script](#tab/ercenablement-script)

This method is applicable only if VMware tools are installed on the target machines. If VMware tools aren't installed, Arc agents can be installed through out-of-band methods. 

Arc agent installation can be automated using the helper script built using the AzCLI command. Download this [helper script](https://aka.ms/arcvmwarebatchenable) to enable VMs and install Arc agents at scale. In a single ARM deployment, the helper script can enable and install Arc agents on 200 VMs.  

### Features of the script

- Creates a log file (vmware-batch.log) for tracking its operations.

- Generates a list of Azure portal links to all the deployments created `(all-deployments-<timestamp>.txt)`. 

- Creates ARM deployment files `(vmw-dep-<timestamp>-<batch>.json)`.

- Can enable up to 200 VMs in a single ARM deployment if guest management is enabled, else enables 400 VMs. 

- Supports running as a cron job to enable all the VMs in a vCenter. 

- Allows for service principal authentication to Azure for automation. 

Before running this script, install az cli and the `connectedvmware` extension. 

### Prerequisites 

Before running this script, install: 

- Azure CLI from [here](/cli/azure/install-azure-cli).

- The `connectedvmware` extension for Azure CLI: Install it by running `az extension add --name connectedvmware`. 

### Usage 

1. Download the script to your local machine. 

2. Open a PowerShell terminal and navigate to the directory containing the script. 

3. Run the following command to allow the script to run, as it's an unsigned script (if you close the session before you complete all the steps, run this command again for the new session): `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`.

4. Run the script with the required parameters. For example, `.\arcvmware-batch-enablement.ps1 -VCenterId "<vCenterId>" -EnableGuestManagement -VMCountPerDeployment 3 -DryRun`. Replace `<vCenterId>` with the ARM ID of your vCenter. 

### Parameters

- `VCenterId`: The ARM ID of the vCenter where the VMs are located. 

- `EnableGuestManagement`: If this switch is specified, the script will enable guest management on the VMs. 

- `VMCountPerDeployment`: The number of VMs to enable per ARM deployment. The maximum value is 200 if guest management is enabled, else it's 400. 

- `DryRun`: If this switch is specified, the script will only create the ARM deployment files. Else, the script will also deploy the ARM deployments. 

### Running as a Cron Job 

You can set up this script to run as a cron job using the Windows Task Scheduler. Here's a sample script to create a scheduled task: 

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

Arc agents can be installed directly on machines without relying on VMware tools or APIs. By following the out-of-band approach, first onboard the machines as Arc-enabled Server resources with Resource type as Microsoft.HybridCompute/machines. After that, perform **Link to vCenter** operation to update the machine's Kind property as VMware, enabling virtual lifecycle operations.  

1. **Connect the machines as Arc-enabled Server resources:** Install Arc agents using Arc-enabled Server scripts. 

    You can use any of the following automation approaches to install Arc agents at scale:

   - [Install Arc agents at scale using a Service Principal](../servers/onboard-service-principal.md).
   - [Install Arc agents at scale using Configuration Manager script](../servers/onboard-configuration-manager-powershell.md).
   - [Install Arc agents at scale with a Configuration Manager custom task sequence](../servers/onboard-configuration-manager-custom-task.md).
   - [Install Arc agents at scale using Group policy](../servers/onboard-group-policy-powershell.md).
   - [Install Arc agents at scale using Ansible playbook](../servers/onboard-ansible-playbooks.md).

2. **Link Arc-enabled Server resources to the vCenter:** The following commands will update the Kind property of Hybrid Compute machines as **VMware**. Linking the machines to vCenter will enable virtual lifecycle operations and power cycle operations (start, stop, etc.) on the machines.  

   - The following command scans all the Arc for Server machines that belong to the vCenter in the specified subscription and links the machines with that vCenter. 

     [!INCLUDE [azure-cli-subscription](./includes/azure-cli-subscription.md)]

   - The following command scans all the Arc for Server machines that belong to the vCenter in the specified Resource Group and links the machines with that vCenter. 

     [!INCLUDE [azure-cli-all](./includes/azure-cli-all.md)]

   - The following command can be used to link an individual Arc for Server resource to vCenter. 

     [!INCLUDE [azure-cli-specified-arc](./includes/azure-cli-specified-arc.md)]

--- 

## Next steps

- [Set up and manage self-service access to VMware resources through Azure RBAC](setup-and-manage-self-service-access.md).
- [Manage and maintain the Azure Connected Machine agent](../servers/manage-agent.md).
- [VM Extension Management with Azure Arc-Enabled Servers](../servers/manage-vm-extensions.md).
