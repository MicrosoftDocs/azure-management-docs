---
title: Install Arc agent on SCVMM VMs
description: Learn how to enable guest management at scale for Arc-enabled SCVMM VMs. 
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: jsuri
author: jyothisuri
ms.topic: how-to 
ms.date: 06/26/2025
keywords: "VMM, Arc, Azure"
ms.custom:
  - build-2025
  - sfi-ropc-nochange
# Customer intent: As an IT infrastructure admin, I want to install Arc agents at scale for my SCVMM VMs so that I can leverage Azure management services for securing, patching, and monitoring those virtual machines effectively.
---

# Install Arc agents on SCVMM VMs

In this article, you learn how to install Azure connected machine agents for SCVMM VMs which is a prerequisite  to use Azure services for securing, patching, monitoring your VMs and leverage Azure Arc benefits such as Extended Security Updates, pay-as-you-go licensing for Windows Server and SQL servers, and Software Attestation benefits.

There are multiple avenues available to install Arc agents on SCVMM VMs which you can leverage based on your deployment preferences: 

- Azure portal
- Script-based manual installation
- Programmatic methods such as Azure CLI, Azure PowerShell, Azure REST APIs, Azure SDKs, Terraform, Bicep and ARM templates. The reference section of this documentation repository has information on the exact syntax. 
- Out-of-band methods such as using a Service Principal, System Center Configuration Manager script, System Center Configuration Manager custom task sequence, Group policy and Ansible playbook.  

## Prerequisites

Ensure the following before you install Arc agents at scale for SCVMM VMs:

- The SCVMM management server and the SCVMM console must be in the same Long-Term Servicing Channel (LTSC) and Update Rollup (UR) version.
- The SCVMM management server must be in a *Connected* state and its associated Azure Arc resource bridge in a *Running* state. 
- *Azure Arc SCVMM VM Contributor* role or a custom Azure role with permissions to install Arc agents on the target machines.
- All the target machines are:
    - Powered on.
    - Running a [supported operating system](../servers/prerequisites.md#supported-operating-systems).
    - Able to connect through the firewall to communicate over the internet and [these URLs](../servers/network-requirements.md?tabs=azure-cloud#urls) aren't blocked.

## Install Arc agents 

# [Azure portal](#tab/azure-portal)

This method is applicable only if you are running: 

- SCVMM 2025, 2022 UR1 or later, and 2019 UR5 or later versions of SCVMM server or console.
- VMs running Windows Server 2012 R2, 2016, 2019, 2022, 2025, Windows 10, and Windows 11.
- For other SCVMM versions, Linux VMs or Windows VMs running WS 2012 or earlier, install Arc agents through the script or out-of-band methods. 

An administrator can install agents for multiple machines from the Azure portal if the machines share the same administrator credentials.

1. Navigate to the **SCVMM management servers** blade on [Azure Arc Center](https://portal.azure.com/#view/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/~/overview), and select the SCVMM management server resource.
2. Select the machines you want to onboard to Arc at-scale and choose the **Enable in Azure** option.
3. Select **Enable guest management** checkbox to install Arc agents on the selected machines. This allows you to use Azure services such as Azure Update Manager, Azure Monitor, Microsoft Defender for Cloud, Azure Policy, Azure Automation, Change Tracking and Inventory, etc. to secure, govern, patch and monitor your virtual machines.

     :::image type="content" source="media/enable-guest-management-at-scale/virtual-machines.png" alt-text="Screenshot of virtual machines screen." lightbox="media/enable-guest-management-at-scale/virtual-machines.png":::

4. If you enable guest management on any of your machines, based on your organization's network policies, choose the connectivity method for the Arc agents that runs in your SCVMM VMs to connect to Azure. The available options are Public endpoint, Proxy server and Private endpoint.   
     - If you want to connect the Arc agent via proxy, provide the proxy server details.
     - If you want to connect Arc agent via private endpoint, follow these [steps](../servers/private-link-security.md) to set up Azure private link and provide the same details. 

      >[!Note]
      > Private endpoint connectivity is only available for Arc agent to Azure communications. For Arc resource bridge to Azure connectivity, Azure Private link isn't supported.

5. Provide the administrator username and password for the machine. For Windows VMs, the account must be part of the local administrator group; and for Linux VM, it must be a root account. 

6. Select **Enable** to start the installation of the Arc agent in the specified machines. Once installation is complete, the Guest management column will switch to Enabled for the machines with Arc agent running. You can start using Azure services for these machines. These credentials won't be persisted in Azure. They're used to install the Azure Arc agent and then discarded.

# [Manual installation script](#tab/installation-script)

>[!NOTE]
>- If you're using a Linux VM, the account must not prompt for login on sudo commands. To override the prompt, from a terminal, run `sudo visudo`, and `add <username> ALL=(ALL) NOPASSWD:ALL` at the end of the file. Ensure you replace `<username>`.
>- If your VM template has these changes incorporated, you won't need to do this for the VM created from that template.

1. Sign in to the target VM as an administrator.
2. Install and run the Azure CLI with the `az` command from either Windows Command Prompt or PowerShell.
3. Sign in to your Azure account in Azure CLI using `az login --use-device-code`
4. Run the downloaded script *arcscvmm-enable-guest-management.ps1* or *arcscvmm-enable-guest-management.sh*, as applicable, using the following commands. The `vmmServerId` parameter should denote your VMM Server’s ARM ID.

    **For a Windows VM:**

    ```azurecli-interactive
    ./arcscvmm-enable-guest-management.ps1 -<vmmServerId> '/subscriptions/<subscriptionId>/resourceGroups/<rgName>/providers/Microsoft.ScVmm/vmmServers/<vmmServerName>
    ```

    **For a Linux VM:**

    ```azurecli-interactive
    ./arcscvmm-enable-guest-management.sh -<vmmServerId> '/subscriptions/<subscriptionId>/resourceGroups/<rgName>/providers/Microsoft.ScVmm/vmmServers/<vmmServerName>
    ```

# [Out-of-band methods](#tab/Out-of-band)

The out-of-band methods first onboard the machines as Arc-enabled Server resources with Resource type as *Microsoft.HybridCompute/machines*. Then, perform **Link to SCVMM** operation to update the machine's Kind property as **SCVMM** which enables VM lifecycle and powercycle operations. 

1. **Connect the machines as Arc-enabled Server resources** using any one of the following automation approaches: 
    - [Service Principal](/azure/azure-arc/servers/onboard-service-principal). 
    - [Configuration Manager script](/azure/azure-arc/servers/onboard-configuration-manager-powershell). 
    - [Configuration Manager custom task sequence](/azure/azure-arc/servers/onboard-configuration-manager-custom-task). 
    - [Group policy](/azure/azure-arc/servers/onboard-group-policy-powershell). 
    - [Ansible playbook](/azure/azure-arc/servers/onboard-ansible-playbooks). 

2. **Link Azure Arc-enabled Server resources to SCVMM**: The following commands update the Kind property of Hybrid Compute machines as **SCVMM**. Linking the machines to SCVMM enables VM lifecycle operations (Create/Delete) and powercycle operations (Start/Restart/Stop) on the machines. 

   - The following command scans all the Azure Arc-enabled Server machines that belong to the SCVMM in the specified subscription and links the machines with that SCVMM. 

      ```azurecli-interactive
      az scvmm vm create-from-machines --subscription contoso-sub --scvmm-id /subscriptions/01234567-0123-0123-0123-0123456789ab/resourceGroups/contoso-rg/providers/Microsoft.ScVmm/vmmServers/contoso-vmmserver 
      ```

   - The following command scans all the Azure Arc-enabled Server machines that belong to the SCVMM in the specified resource group and links the machines with that SCVMM. 

      ```azurecli-interactive
      az scvmm vm create-from-machines –resource group contoso-rg --scvmm-id /subscriptions/01234567-0123-0123-0123-0123456789ab/resourceGroups/contoso-rg/providers/Microsoft.ScVmm/vmmServers/contoso-vmmserver   
      ```

   - The following command can be used to link an individual Azure Arc-enabled Server resource to SCVMM. 

      ```azurecli-interactive
      az scvmm vm create-from-machines –resource group contoso-rg --name contoso-vm --scvmm-id /subscriptions/01234567-0123-0123-0123-0123456789ab/resourceGroups/contoso-rg/providers/Microsoft.ScVmm/vmmServers/contoso-vmmserver   
      ```

---

## Next steps

- [Set up and manage self-service access to SCVMM resources](set-up-and-manage-self-service-access-scvmm.md)
- [Manage and maintain the Azure Connected Machine agent](../servers/manage-agent.md).
- [Manage VM extensions to use Azure management services for your SCVMM VMs](../servers/manage-vm-extensions.md).
