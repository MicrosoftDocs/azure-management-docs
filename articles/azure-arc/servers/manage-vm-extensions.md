---
title: VM Extension Management with Azure Arc-Enabled Servers
description: Azure Arc-enabled servers can manage deployment of virtual machine extensions that provide post-deployment configuration and automation tasks with non-Azure VMs.
ms.date: 06/19/2025
ms.topic: concept-article
# Customer intent: "As a system administrator managing hybrid cloud environments, I want to deploy and manage virtual machine extensions on Arc-enabled servers, so that I can automate configuration tasks and enhance performance monitoring."
---

# Virtual machine extension management with Azure Arc-enabled servers

Virtual machine (VM) extensions are small applications that provide post-deployment configuration and automation tasks on Azure VMs.

For example, VM extensions can be used to enable functionality such as:

- Collect log data for analysis with [Azure Monitor Logs](/azure/azure-monitor/logs/data-platform-logs) by enabling the Azure Monitor agent VM extension.

- With [VM insights](/azure/azure-monitor/vm/vminsights-overview), analyze the performance of your Windows and Linux VMs, and monitor their processes and dependencies on other resources and external processes. You achieve these capabilities by enabling both the Azure Monitor agent and the Dependency agent VM extensions.

- Download and run scripts on hybrid connected machines by using the Custom Script extension. This extension is useful for post-deployment configuration, software installation, or any other configuration or management tasks.

- Automatically refresh certificates stored in [Azure Key Vault](/azure/key-vault/general/overview).

With Azure Arc-enabled servers, you can deploy, remove, and update Azure VM extensions to non-Azure Windows and Linux VMs. This ability simplifies the management of your hybrid machines through their life cycle. You can deploy VM extensions to hybrid machines managed by Azure Arc-enabled servers via the following methods:

- [Azure portal](manage-vm-extensions-portal.md)
- [Azure CLI](manage-vm-extensions-cli.md)
- [Azure PowerShell](manage-vm-extensions-powershell.md)
- [Azure Resource Manager templates](manage-vm-extensions-template.md)

Many VM extensions can be configured for [automatic upgrades](manage-automatic-vm-extension-upgrade.md?tab=azure-portal).

## Availability

VM extension functionality is available only in the [supported regions](overview.md#supported-regions). Be sure to onboard your machine in one of these regions.

For the regional availabilities of Azure services and VM extensions that are available for Azure Arc-enabled servers, refer to the [Azure Global Product Availability Roadmap](https://global.azure.com/product-availability/roadmap).

You can configure lists of the extensions that you want to allow and block on servers. For more information, see [Extension allowlists and blocklists](/azure/azure-arc/servers/security-overview#extension-allowlists-and-blocklists).

## Extensions

Many VM extensions are supported with Azure Arc-enabled servers. While the lists shown here are not exhaustive, they include some of the most popular extensions that you can use with Azure Arc-enabled servers.

### Windows extensions

The following table lists some of the key VM extensions that are available for Azure Arc-enabled servers running Windows. For more information about usage and support, see the "additional information" links.

|Extension |Publisher |Type |Additional information |
|----------|----------|-----|-----------------------|
|Microsoft Antimalware extension |Microsoft.Azure.Security |IaaSAntimalware |[Microsoft Antimalware extension for Windows](/azure/virtual-machines/extensions/iaas-antimalware-windows) |
|Custom Script extension |Microsoft.Compute | CustomScriptExtension |[Windows Custom Script Extension](/azure/virtual-machines/extensions/custom-script-windows)|
|Azure Monitor agent |Microsoft.Azure.Monitor |AzureMonitorWindowsAgent |[Deployment options for Azure Monitor agent on Azure Arc-enabled servers](azure-monitor-agent-deployment.md) |
|Azure Monitor Dependency agent |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentWindows | [Dependency agent virtual machine extension for Windows](/azure/virtual-machines/extensions/agent-dependency-windows)|
|Azure Key Vault extension for Windows | Microsoft.Azure.Key.Vault |KeyVaultForWindows | [Key Vault virtual machine extension for Windows](/azure/virtual-machines/extensions/key-vault-windows) |
|Azure Automation Hybrid Runbook Worker extension |Microsoft.Compute |HybridWorkerForWindows |[Deploy an extension-based user Hybrid Runbook Worker](/azure/automation/extension-based-hybrid-runbook-worker-install) (to execute runbooks locally) |
|Windows Admin Center |Microsoft.AdminCenter |AdminCenter |[Manage Azure Arc-enabled servers by using Windows Admin Center in Azure](/windows-server/manage/windows-admin-center/azure/manage-arc-hybrid-machines) |
|Windows OS Update Extension |Microsoft.SoftwareUpdateManagement |WindowsOsUpdateExtension |[Overview of Azure Update Manager](/azure/update-manager/overview?tabs=azure-vms) |
|Windows Patch extension |Microsoft.CPlat.Core |WindowsPatchExtension |[Automatic guest patching for Azure virtual machines and scale sets](/azure/virtual-machines/automatic-vm-guest-patching) |
|Network Watcher agent | Microsoft.Azure.NetworkWatcher |NetworkWatcherAgentWindows |[Azure Network Watcher overview](/azure/network-watcher/network-watcher-overview) |
|Boot Integrity Monitoring - Guest Attestation | Microsoft.Azure.NetworkWatcher |NetworkWatcherAgentWindows |[Azure Network Watcher overview](/azure/network-watcher/network-watcher-overview) |
|Open SSH for Windows | Microsoft.Azure.OpenSSH | WindowsOpenSSH | [Connect to an Azure Local VM using SSH and RDP over SSH](/azure/azure-local/manage/connect-arc-vm-using-ssh)|
|Azure Site Recovery | Microsoft.SiteRecovery.Dra | Windows | [Configure Azure Site Recovery for Arc-enabled Windows servers](/windows-server/manage/azure-arc/azure-site-recovery-for-windows-server) |
|Azure Extension for SQL Server |Microsoft.AzureData |WindowsAgent.SqlServer |[Connect your SQL Server to Azure Arc](/sql/sql-server/azure-arc/connect?tabs=windows) (installs the extension automatically) |
|Defender for SQL Servers Advanced Threat Protection | Microsoft.Azure.AzureDefenderForSQL | AdvancedThreatProtection.Windows | [Enable Defender for SQL Servers on Machines](/azure/defender-for-cloud/defender-for-sql-usage) |
|SQL Server Backup |Microsoft.Azure.RecoveryServices.WorkloadBackup | AzureBackupWindowsWorkload | [About SQL Server Backup in Azure VMs](/azure/backup/backup-azure-sql-database) |

### Linux extensions

The following table lists some of the key VM extensions that are available for Azure Arc-enabled servers running Linux. For more information about usage and support, see the "additional information" links.

|Extension |Publisher |Type |Additional information |
|----------|----------|-----|-----------------------|
|Custom Script extension |Microsoft.Azure.Extensions |CustomScript |[Linux Custom Script Extension version 2](/azure/virtual-machines/extensions/custom-script-linux) |
|Azure Monitor agent |Microsoft.Azure.Monitor |AzureMonitorLinuxAgent |[Deployment options for Azure Monitor agent on Azure Arc-enabled servers](azure-monitor-agent-deployment.md) |
|Azure Monitor for VMs (insights) |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentLinux |[Dependency agent virtual machine extension for Linux](/azure/virtual-machines/extensions/agent-dependency-linux) |
|Azure Key Vault extension for Linux | Microsoft.Azure.Key.Vault |KeyVaultForLinux | [Key Vault virtual machine extension for Linux](/azure/virtual-machines/extensions/key-vault-linux) |
|Azure Automation Hybrid Runbook Worker extension  |Microsoft.Compute |HybridWorkerForLinux |[Deploy an extension-based user Hybrid Runbook Worker](/azure/automation/extension-based-hybrid-runbook-worker-install) (to execute runbooks locally)|
|Linux OS Update Extension  |Microsoft.SoftwareUpdateManagement |LinuxOsUpdateExtension |[Overview of Azure Update Manager](/azure/update-manager/overview?tabs=azure-vms)|
|Linux Patch Extension  |Microsoft.CPlat.Core |LinuxPatchExtension |[Automatic guest patching for Azure virtual machines and scale sets](/azure/virtual-machines/automatic-vm-guest-patching)|
|Network Watcher agent | Microsoft.Azure.NetworkWatcher |NetworkWatcherAgentLinux |[Azure Network Watcher overview](/azure/network-watcher/network-watcher-overview) |
|Boot Integrity Monitoring - Guest Attestation | Microsoft.Azure.NetworkWatcher |NetworkWatcherAgentLinux |[Azure Network Watcher overview](/azure/network-watcher/network-watcher-overview) |
|Microsoft Entra login extension |Microsoft.Azure.ActiveDirectory |AADSSHLoginForLinux |[SSH access to Azure Arc-enabled servers](ssh-arc-overview.md#optional-install-microsoft-entra-login-extension) |
|Azure Extension for SQL Server |Microsoft.AzureData |LinuxAgent.SqlServer |[Connect your SQL Server to Azure Arc](/sql/sql-server/azure-arc/connect?tabs=linux) (installs the extension automatically) |

### Extensions from partner publishers

Several extensions from partner publishers are supported on Azure Arc-enabled servers. The extensions listed here are available for both Windows and Linux.

|Extension |Publisher |Type |Additional information |
|----------|----------|-----|-----------------------|
|Datadog Agent |Datadog.Agent |DatadogWindowsAgent<br>DatadogLinuxAgent |[Introducing Azure monitoring with one-click Datadog deployment](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/)|
|Dynatrace OneAgent |Dynatrace.Ruxit |OneAgentWindows<br>OneAgentLinux|[What is Azure Native Dynatrace Service?](/azure/partner-solutions/dynatrace/overview) |
|New Relic |NewRelic.Infrastructure.Extensions|newrelic-infra-windows<br>newrelic-infra |[What is Azure Native New Relic Service?](/azure/partner-solutions/new-relic/overview)  |

> [!NOTE]
> The Desired State Configuration VM extension is no longer available for Azure Arc-enabled servers. We recommend that you [migrate to machine configuration](/azure/governance/machine-configuration/migrate-from-azure-automation) or use the Custom Script Extension to manage the post-deployment configuration of your server.

## Extension deployment prerequisites

Review the documentation for each VM extension referenced in the previous tables to understand its network and system requirements beyond the [general prerequisites](prerequisites.md) and [networking requirements](network-requirements.md) for Arc-enabled servers. This effort can help prevent connectivity issues with an Azure service or feature that relies on that VM extension.

To deploy an extension to Azure Arc-enabled servers, a user needs the following permissions:

- `microsoft.hybridcompute/machines/read`
- `microsoft.hybridcompute/machines/extensions/read`
- `microsoft.hybridcompute/machines/extensions/write`

The role **Azure Connected Machine Resource Administrator** includes the permissions required to deploy extensions. It also includes permission to delete Azure Arc-enabled server resources.

Azure Arc-enabled servers with one or more VM extensions installed can be moved between resource groups, or to another Azure subscription, without experiencing any impact to their configuration. The source and destination scopes must exist within the same [Microsoft Entra tenant](/azure/active-directory/develop/quickstart-create-new-tenant). For more information about moving resources and considerations before you proceed, see [Move resources to a new resource group or subscription](/azure/azure-resource-manager/management/move-resource-group-and-subscription).

## Related content

- You can deploy, manage, and remove VM extensions by using the [Azure CLI](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), the [Azure portal](manage-vm-extensions-portal.md), or [Azure Resource Manager templates](manage-vm-extensions-template.md).
