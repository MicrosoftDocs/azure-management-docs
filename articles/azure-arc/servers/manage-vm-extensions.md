---
title: VM Extension Management with Azure Arc-Enabled Servers
description: Azure Arc-enabled servers can manage deployment of virtual machine extensions that provide post-deployment configuration and automation tasks with non-Azure VMs.
ms.date: 05/08/2025
ms.topic: concept-article
---

# Virtual machine extension management with Azure Arc-enabled servers

Virtual machine (VM) extensions are small applications that provide post-deployment configuration and automation tasks on Azure VMs. For example, if a virtual machine requires software installation, antivirus protection, or the running of a script, you can use a VM extension.

With Azure Arc-enabled servers, you can deploy, remove, and update Azure VM extensions to non-Azure Windows and Linux VMs. This ability simplifies the management of your hybrid machines through their life cycle. You can manage VM extensions by using the following methods on your hybrid machines or servers managed by Azure Arc-enabled servers:

- [Azure portal](manage-vm-extensions-portal.md)
- [Azure CLI](manage-vm-extensions-cli.md)
- [Azure PowerShell](manage-vm-extensions-powershell.md)
- [Azure Resource Manager templates](manage-vm-extensions-template.md)

> [!NOTE]
> Azure Arc-enabled servers doesn't support deploying and managing VM extensions to Azure virtual machines. For Azure VMs, see the [VM extension overview](/azure/virtual-machines/extensions/overview) article.
>
> Currently, you can update extensions only from the Azure portal or the Azure CLI. Updating extensions from Azure PowerShell or an Azure Resource Manager template isn't currently supported.

## Key benefits

VM extension support for Azure Arc-enabled servers provides the following key benefits:

- Collect log data for analysis with [Azure Monitor Logs](/azure/azure-monitor/logs/data-platform-logs) by enabling the Azure Monitor agent VM extension. You can do complex analysis across log data from various sources.

- With [VM insights](/azure/azure-monitor/vm/vminsights-overview), analyze the performance of your Windows and Linux VMs, and monitor their processes and dependencies on other resources and external processes. You achieve these capabilities by enabling the VM extensions for both the Azure Monitor agent and the Dependency agent.

- Download and run scripts on hybrid connected machines by using the Custom Script Extension. This extension is useful for post-deployment configuration, software installation, or any other configuration or management tasks.

- Automatically refresh certificates stored in [Azure Key Vault](/azure/key-vault/general/overview).

## Availability

VM extension functionality is available only in the [supported regions](overview.md#supported-regions). Be sure to onboard your machine in one of these regions.

Additionally, you can configure lists of the extensions that you want to allow and block on servers. For more information, see [Extension allowlists and blocklists](/azure/azure-arc/servers/security-overview#extension-allowlists-and-blocklists).

## Extensions

In this release, we support the following VM extensions on Windows and Linux machines.

To learn about the Azure Connected Machine agent package and the Extension agent component, see [Agent overview](agent-overview.md).

> [!NOTE]
> The Desired State Configuration VM extension is no longer available for Azure Arc-enabled servers. We recommend that you [migrate to machine configuration](/azure/governance/machine-configuration/migrate-from-azure-automation) or use the Custom Script Extension to manage the post-deployment configuration of your server.

Azure Arc-enabled servers supports moving machines with one or more VM extensions installed between resource groups or another Azure subscription without experiencing any impact to their configuration. The source and destination subscriptions must exist within the same [Microsoft Entra tenant](/azure/active-directory/develop/quickstart-create-new-tenant). This support is enabled starting with the Connected Machine agent version 1.8.21197.005. For more information about moving resources and considerations before you proceed, see [Move resources to a new resource group or subscription](/azure/azure-resource-manager/management/move-resource-group-and-subscription).

### Windows extensions

|Extension |Publisher |Type |Additional information |
|----------|----------|-----|-----------------------|
|Microsoft Defender for Cloud integrated vulnerability scanner |Qualys |WindowsAgent.AzureSecurityCenter |[Microsoft Defender for Cloud integrated vulnerability assessment solution for Azure and hybrid machines](/azure/security-center/deploy-vulnerability-assessment-vm)|
|Microsoft Antimalware extension |Microsoft.Azure.Security |IaaSAntimalware |[Microsoft Antimalware extension for Windows](/azure/virtual-machines/extensions/iaas-antimalware-windows) |
|Custom Script Extension |Microsoft.Compute | CustomScriptExtension |[Windows Custom Script Extension](/azure/virtual-machines/extensions/custom-script-windows)|
|Azure Monitor for VMs (insights) |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentWindows | [Dependency agent virtual machine extension for Windows](/azure/virtual-machines/extensions/agent-dependency-windows)|
|Azure Key Vault virtual machine extension for Windows | Microsoft.Azure.Key.Vault |KeyVaultForWindows | [Key Vault virtual machine extension for Windows](/azure/virtual-machines/extensions/key-vault-windows) |
|Azure Monitor agent |Microsoft.Azure.Monitor |AzureMonitorWindowsAgent |[Install the Azure Monitor agent](/azure/azure-monitor/agents/azure-monitor-agent-manage) |
|Azure Automation Hybrid Runbook Worker extension |Microsoft.Compute |HybridWorkerForWindows |[Deploy an extension-based user Hybrid Runbook Worker](/azure/automation/extension-based-hybrid-runbook-worker-install) (to execute runbooks locally) |
|Azure Extension for SQL Server |Microsoft.AzureData |WindowsAgent.SqlServer |[Install Azure Extension for SQL Server](/sql/sql-server/azure-arc/connect#initiate-the-connection-from-azure) (to initiate a SQL Server connection to Azure) |
|Windows Admin Center (preview) |Microsoft.AdminCenter |AdminCenter |[Manage Azure Arc-enabled servers by using Windows Admin Center in Azure](/windows-server/manage/windows-admin-center/azure/manage-arc-hybrid-machines) |
|Windows OS Update Extension |Microsoft.SoftwareUpdateManagement |WindowsOsUpdateExtension |[Overview of Azure Update Manager](/azure/update-manager/overview?tabs=azure-vms) |
|Windows Patch extension |Microsoft.CPlat.Core |WindowsPatchExtension |[Automatic guest patching for Azure virtual machines and scale sets](/azure/virtual-machines/automatic-vm-guest-patching) |

### Linux extensions

|Extension |Publisher |Type |Additional information |
|----------|----------|-----|-----------------------|
|Microsoft Defender for Cloud integrated vulnerability scanner |Qualys |LinuxAgent.AzureSecurityCenter |[Microsoft Defender for Cloud integrated vulnerability assessment solution for Azure and hybrid machines](/azure/security-center/deploy-vulnerability-assessment-vm)|
|Custom Script Extension |Microsoft.Azure.Extensions |CustomScript |[Linux Custom Script Extension version 2](/azure/virtual-machines/extensions/custom-script-linux) |
|Azure Monitor for VMs (insights) |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentLinux |[Dependency agent virtual machine extension for Linux](/azure/virtual-machines/extensions/agent-dependency-linux) |
|Azure Key Vault virtual machine extension for Linux | Microsoft.Azure.Key.Vault |KeyVaultForLinux | [Key Vault virtual machine extension for Linux](/azure/virtual-machines/extensions/key-vault-linux) |
|Azure Monitor agent |Microsoft.Azure.Monitor |AzureMonitorLinuxAgent |[Install the Azure Monitor agent](/azure/azure-monitor/agents/azure-monitor-agent-manage) |
|Azure Automation Hybrid Runbook Worker extension  |Microsoft.Compute |HybridWorkerForLinux |[Deploy an extension-based user Hybrid Runbook Worker](/azure/automation/extension-based-hybrid-runbook-worker-install) (to execute runbooks locally)|
|Linux OS Update Extension  |Microsoft.SoftwareUpdateManagement |LinuxOsUpdateExtension |[Overview of Azure Update Manager](/azure/update-manager/overview?tabs=azure-vms)|
|Linux Patch Extension  |Microsoft.CPlat.Core |LinuxPatchExtension |[Automatic guest patching for Azure virtual machines and scale sets](/azure/virtual-machines/automatic-vm-guest-patching)|

## Prerequisites

This feature depends on the following Azure resource providers in your subscription:

- **Microsoft.HybridCompute**
- **Microsoft.GuestConfiguration**

If they aren't already registered, follow the steps in [Register Azure resource providers](prerequisites.md#azure-resource-providers).

Review the documentation for each VM extension referenced in the previous tables to understand if it has any network or system requirements. This effort can help prevent connectivity issues with an Azure service or feature that relies on that VM extension.

### Required permissions

To deploy an extension to Azure Arc-enabled servers, a user needs the following permissions:

- `microsoft.hybridcompute/machines/read`
- `microsoft.hybridcompute/machines/extensions/read`
- `microsoft.hybridcompute/machines/extensions/write`

The role **Azure Connected Machine Resource Administrator** includes the permissions required to deploy extensions. It also includes permission to delete Azure Arc-enabled server resources.

### Azure Monitor Agent VM extension

Before you install the extension, we suggest that you review the [deployment options for the Azure Monitor Agent](azure-monitor-agent-deployment.md) to understand the available methods and which one meets your needs.

### Key Vault VM extension

The Key Vault VM extension doesn't support the following Linux operating systems:

- Red Hat Enterprise Linux (RHEL) 7 (x64)
- Amazon Linux 2 (x64)

Deploying the Key Vault VM extension is supported only when you're using:

- The Azure CLI
- Azure PowerShell
- An Azure Resource Manager template

Before you deploy the extension, you need to complete the following steps:

1. [Create a vault and a certificate](/azure/key-vault/certificates/quick-create-portal) (self-signed or imported).

2. Grant the Azure Arc-enabled server access to the certificate secret. If you're using the [Azure role-based access control (RBAC) preview](/azure/key-vault/general/rbac-guide), search for the name of the Azure Arc resource and assign it the **Key Vault Secrets User (preview)** role. If you're using a [Key Vault access policy](/azure/key-vault/general/assign-access-policy-portal), assign secret **Get** permissions to the Azure Arc resource's system-assigned identity.

### Connected Machine agent

Verify that your machine matches the [supported versions](prerequisites.md#supported-operating-systems) of Windows and Linux operating systems for the Azure Connected Machine agent.

The minimum version of the Connected Machine agent that's supported with this feature on Windows and Linux is the 1.0 release.

To upgrade your machine to the required version of the agent, see [Upgrade the agent](manage-agent.md#upgrade-the-agent).

## Availability of operating system extensions

The following extensions are available for Windows and Linux machines.

### Windows extension availability

|Operating system |Azure Monitor agent |Dependency VM insights |Qualys |Custom script |Key Vault |Hybrid runbook |Antimalware extension |Windows Admin Center |
|-----------------|--------------------|-----------------------|-------|--------------|----------|---------------|----------------------|---------------------|
|Windows Server 2022 |X |X |X |X | |X | |X |
|Windows Server 2019 |X |X |X |X |X | | |X |
|Windows Server 2016 |X |X |X |X |X |X |Built in |X |
|Windows Server 2012 R2 |X |X |X |X | |X |X | |
|Windows Server 2012 |X |X |X |X |X |X |X | |
|Windows Server 2008 R2 SP1 |X |X |X |X | |X |X | |
|Windows Server 2008 R2 | | |X |X | |X |X | |
|Windows Server 2008 SP2 | | |X |X | |X | | |
|Windows 11 client OS |X | |X | | | | | |
|Windows 10 1803 (RS4) and later |X | |X |X | | | | |
|Windows 10 Enterprise (including multi-session) and Pro (server scenarios only) |X |X |X |X | |X | | |
|Windows 8 Enterprise and Pro (server scenarios only) | |X |X | | |X | | |
|Windows 7 SP1 (server scenarios only) | |X |X | | |X | | |
|Azure Local (server scenarios only) | | |X | | |X | | |

### Linux extension availability

|Operating system |Azure Monitor agent |Dependency VM insights |Qualys |Custom script |Key Vault |Hybrid runbook |Antimalware extension |Connected Machine agent |
|-----------------|--------------------|-----------------------|-------|--------------|----------|---------------|----------------------|------------------------|
|Amazon Linux 2 | | |X | | |X |X |X |
|Debian 10 |X | |X |X | |X | |X |
|Debian 9 |X |X |X |X | | | |X |
|Debian 8 | |X |X | | |X | |X |
|Debian 7 | | |X | | |X | |X |
|OpenSUSE 13.1+ | | |X |X | | | |X |
|Oracle Linux 8 |X | |X |X | |X |X |X |
|Oracle Linux 7 |X | |X |X | |X |X |X |
|Oracle Linux 6 | | |X |X | |X |X |X |
|Red Hat Enterprise Linux Server 8 |X | |X |X | |X |X |X |
|Red Hat Enterprise Linux Server 7 |X |X |X |X | |X |X |X |
|Red Hat Enterprise Linux Server 6 | |X |X | | |X | |X |
|SUSE Linux Enterprise Server 15.2 |X | |X |X |X | |X |X |
|SUSE Linux Enterprise Server 15.1 |X | |X |X |X |X |X |X |
|SUSE Linux Enterprise Server 15 SP1 |X |X |X |X |X |X |X |X |
|SUSE Linux Enterprise Server 15 |X |X |X |X |X |X |X |X |
|SUSE Linux Enterprise Server 15 SP5 |X |X |X |X | |X |X |X |
|SUSE Linux Enterprise Server 12 SP5 |X |X |X |X |X | |X |X |
|Ubuntu 24.04 LTS |X |X |X |X | |X |X |X |
|Ubuntu 22.04 LTS |X |X |X |X | |X |X |X |
|Ubuntu 20.04 LTS |X |X |X |X | |X |X |X |
|Ubuntu 18.04 LTS |X |X |X |X |X |X |X |X |
|Ubuntu 16.04 LTS |X |X |X | | |X |X |X |
|Ubuntu 14.04 LTS | | |X | | |X | |X |

For the regional availabilities of Azure services and VM extensions that are available for Azure Arc-enabled servers, refer to the [Azure Global Product Availability Roadmap](https://global.azure.com/product-availability/roadmap).

## Related content

- You can deploy, manage, and remove VM extensions by using the [Azure CLI](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), the [Azure portal](manage-vm-extensions-portal.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).
