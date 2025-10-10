---
title: Azure Connected Machine agent deployment options
description: Learn about the different options to onboard machines to Azure Arc-enabled servers.
ms.date: 03/19/2025
ms.topic: how-to 
# Customer intent: As a system administrator, I want to choose an appropriate onboarding method for the Connected Machine agent so that I can effectively connect my hybrid machines to Azure for management and monitoring.
---

# Azure Connected Machine agent deployment options

You can connect machines in your hybrid environment directly with Azure by using different methods, depending on your requirements and the tools you prefer to use.

## Onboarding methods

The following table highlights each method so that you can determine which one works best for your deployment. For detailed information, follow the links to view the steps for each subject.

| Method | Description |
|--------|-------------|
| Interactively | Manually install the agent on a single or small number of machines to [connect machines by using a deployment script](onboard-portal.md).<br> From the Azure portal, you can generate a script and run it on the machine to automate the installation and configuration steps of the agent.|
| Interactively | [Connect machines from the Windows Admin Center](onboard-windows-admin-center.md). |
| Interactively | [Connect Windows Server machines to Azure through Azure Arc setup](onboard-windows-server.md). |
| Interactively or at scale | [Connect machines by using PowerShell](onboard-powershell.md). |
| At scale | [Connect machines at scale by using Ansible playbooks](onboard-ansible-playbooks.md) to create a service principal for onboarding Ansible-managed nodes to Azure Arc-enabled servers at scale by using Ansible playbooks. |
| At scale | [Connect machines by using a service principal](onboard-service-principal.md) to install the agent at scale noninteractively.|
| At scale | [Connect machines by running PowerShell scripts with Configuration Manager](onboard-configuration-manager-powershell.md).|
| At scale | [Connect machines with a Configuration Manager custom task sequence](onboard-configuration-manager-custom-task.md).|
| At scale | [Connect Windows machines by using Group Policy](onboard-group-policy-powershell.md).|
| At scale | [Connect machines from Azure Automation Update Management](onboard-update-management-machines.md) to create a service principal that installs and configures the agent for multiple machines managed with Azure Automation Update Management to connect machines noninteractively. |
| At scale | [Install the Azure Arc agent on VMware virtual machines (VMs) at scale by using Azure Arc-enabled VMware vSphere](../vmware-vsphere/enable-guest-management-at-scale.md). With Azure Arc-enabled VMware vSphere, you can [connect your VMware vCenter server to Azure](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md), automatically discover your VMware VMs, and install the Azure Arc agent on them. Requires VMware tools on VMs.|
| At scale | [Install the Azure Arc agent on System Center Virtual Machine Manager (SCVMM) VMs at scale by using Azure Arc-enabled SCVMM](../system-center-virtual-machine-manager/enable-guest-management-at-scale.md). With Azure Arc-enabled SCVMM, you can [connect your SCVMM management server to Azure](../system-center-virtual-machine-manager/quickstart-connect-system-center-virtual-machine-manager-to-arc.md), automatically discover your SCVMM VMs, and install the Azure Arc agent on them. |
| At scale | [Connect your AWS cloud through the multicloud connector enabled by Azure Arc](../multicloud-connector/connect-to-aws.md) and [enable the ](../multicloud-connector/onboard-multicloud-vms-arc.md) [Azure Arc onboarding](../multicloud-connector/onboard-multicloud-vms-arc.md) [solution](../multicloud-connector/onboard-multicloud-vms-arc.md) to autodiscover and onboard EC2 VMs. |

> [!IMPORTANT]
> You can't install the Connected Machine agent on an Azure VM. The installation script warns you and rolls back if it detects that the server is running in Azure.

Be sure to review the basic [prerequisites](prerequisites.md) and [network configuration requirements](network-requirements.md) before you deploy the agent and any specific requirements listed in the steps for the onboarding method that you choose. To learn more about what changes the agent makes to your system, see [Overview of the Azure Connected Machine agent](agent-overview.md).

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Related content

* Learn about the Azure Connected Machine agent [prerequisites](prerequisites.md) and [network requirements](network-requirements.md).
* Review the [Planning and deployment guide for Azure Arc-enabled servers](plan-at-scale-deployment.md).
* Learn about [reconfiguring, upgrading, and removing the Connected Machine agent](manage-agent.md).
* Try out Azure Arc-enabled servers by using the [Azure Arc jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_servers).
