---
title: Extensions security for Azure Arc-enabled servers
description: Learn about security fundamentals for VM extensions used with Azure Arc-enabled servers.
ms.topic: concept-article
ms.date: 07/28/2025
# Customer intent: "As a system administrator, I want to configure and secure VM extensions for Azure Arc-enabled servers, so that I can manage monitoring and security capabilities while restricting unauthorized access and maintaining compliance with security policies."
---

# Extensions security for Azure Arc-enabled servers

This article describes the fundamentals of [VM extensions](manage-vm-extensions.md) for Azure Arc-enabled servers and details how extension settings can be customized.

## Extension basics

[Virtual machine (VM) extensions for Azure Arc-enabled servers](manage-vm-extensions.md) are optional add-ons that enable other functionality, such as monitoring, patch management, and script execution. Extensions are published by Microsoft and select third parties from the Azure Marketplace and stored in Microsoft-managed storage accounts. All extensions are scanned for malware as part of the publishing process. Extensions supported for Azure Arc-enabled servers are identical to those available for Azure VMs, ensuring consistency across your operating environments.

Extensions are downloaded directly from Azure Storage (`*.blob.core.windows.net`) at the time they are installed or upgraded, unless you've configured private endpoints. The storage accounts regularly change and can't be predicted in advance. When private endpoints are used, extensions are proxied via the regional URL for the Azure Arc service instead.

A digitally signed catalog file is downloaded separately from the extension package and used to verify the integrity of each extension before the extension manager opens or executes the extension package. If the downloaded ZIP file for the extension doesn't match the contents in the catalog file, the extension operation will be aborted.

Extensions can take settings to customize or configure installation, such as proxy URLs or API keys that connect a monitoring agent to its cloud service. Extension settings fall into two categories: regular settings and protected settings. Protected settings aren't persisted in Azure and are encrypted at rest on your local machine.

All extension operations originate from Azure through an API call, CLI, PowerShell, or Azure portal action. This design ensures that any action to install, update, or upgrade an extension on a server gets logged in the Azure Monitor Activity Log. The Azure Connected Machine agent does allow extensions to be removed locally for troubleshooting and cleanup purposes. However, if the extension is removed locally and the service still expects the machine to have the extension installed, it will be reinstalled the next time the extension manager syncs with Azure.

## Script execution

The extension manager can be used to run scripts on machines using the Custom Script Extension or Run Command. By default, these scripts run in the extension manager's user context – Local System on Windows or root on Linux – meaning these scripts have unrestricted access to the machine. If you don't intend to use these features, you can block them using an [allowlist or blocklist](#allowlists-and-blocklists).

You can use available controls to restrict or disable unnecessary management features. For instance, unless you intend to use custom script extension for remote code execution, it's best to disable its use, as it can be used by attackers to remotely execute commands to place malware or other malicious code into your virtual machine. You can use the allowlist mechanism to disable use of the custom script extension if its use doesn't meet your security requirements.

## Local agent security controls

You can optionally limit the extensions that can be installed on your server and disable Guest Configuration. These controls can be useful when connecting servers to Azure for a single purpose, such as collecting event logs, without allowing other management capabilities to be used on the server.

These security controls can only be configured by running a command on the server itself, and they can't be modified from Azure. This approach preserves the server admin's intent when enabling remote management scenarios with Azure Arc, but also means that it's more difficult to change these options later. These controls are intended for sensitive servers such as Active Directory domain controllers, servers that handle payment data, and servers subject to strict change control measures). In most other cases, it's not necessary to modify these settings.

## Allowlists and blocklists

The Azure Connected Machine agent supports an allowlist and blocklist to restrict which extensions can be installed on your machine. Allowlists are exclusive, meaning that only the specific extensions you include in the list can be installed. Blocklists are exclusive, meaning anything except those extensions can be installed. Allowlists are preferable to blocklists because they inherently block any new extensions that become available in the future.

Allowlists and blocklists are configured locally on a per-server basis. This ensures that nobody, not even a user with Owner or Global Administrator permissions in Azure, can override your security rules by trying to install an unauthorized extension. If someone tries to install an unauthorized extension, the extension manager refuses to install it and marks the extension installation report as a failure to Azure.

Allowlists and blocklists can be configured anytime after the agent is installed, including before the agent is connected to Azure.

If no allowlist or blocklist is configured on the agent, all extensions are allowed.

The most secure option is to explicitly allow the extensions you expect to be installed. Any extension not in the allowlist is automatically blocked. For example, to configure the Azure Connected Machine agent to allow only the Azure Monitor Agent for Linux, run the following command on each server:

```bash
azcmagent config set extensions.allowlist "Microsoft.Azure.Monitor/AzureMonitorLinuxAgent"
```

Here is an example blocklist that blocks all extensions with the capability of running arbitrary scripts:

```bash
azcmagent config set extensions.blocklist "Microsoft.Cplat.Core/RunCommandHandlerWindows, Microsoft.Cplat.Core/RunCommandHandlerLinux,Microsoft.Compute/CustomScriptExtension,Microsoft.Azure.Extensions/CustomScript,Microsoft.Azure.Automation.HybridWorker/HybridWorkerForWindows,Microsoft.Azure.Automation/HybridWorkerForLinux,Microsoft.EnterpriseCloud.Monitoring/MicrosoftMonitoringAgent, Microsoft.EnterpriseCloud.Monitoring/OMSAgentForLinux"
```

Specify extensions with their publisher and type, separated by a forward slash `/`. See the list of the [most common extensions](manage-vm-extensions.md#extensions) in the docs.

You can list the VM extensions that are already installed on your server in the [portal](manage-vm-extensions-portal.md#list-extensions-installed), [Azure PowerShell](manage-vm-extensions-powershell.md#list-extensions-installed), or [Azure CLI](manage-vm-extensions-cli.md#list-extensions-installed).

The table describes the behavior when performing an extension operation against an agent that has the allowlist or blocklist configured.

| Operation | In the allowlist | In the blocklist | In both the allowlist and blocklist | Not in any list, but an allowlist is configured |
|--|--|--|--|
| Install extension | Allowed | Blocked | Blocked | Blocked |
| Update (reconfigure) extension | Allowed | Blocked | Blocked | Blocked |
| Upgrade extension | Allowed | Blocked | Blocked | Blocked |
| Delete extension | Allowed | Allowed | Allowed | Allowed |

> [!IMPORTANT]
> If an extension is already installed on your server before you configure an allowlist or blocklist, it isn't automatically removed. It's your responsibility to delete the extension from Azure to fully remove it from the machine. Delete requests are always accepted to accommodate this scenario. Once deleted, the allowlist and blocklist determine whether or not to allow future install attempts.

The allowlist value `Allow/None` instructs the extension manager to run, but not allow any extensions to be installed. This value is recommended when using Azure Arc to deliver Windows Server 2012 Extended Security Updates (ESU) without intending to use any other extensions.

```bash
azcmagent config set extensions.allowlist "Allow/None"
```

## Azure Policy

Another option to restrict which extensions can be installed is to use [Azure Policy](/azure/governance/policy). Policies have the advantage of being configurable in the cloud, so a change on each individual server isn't required if you need to change the list of approved extensions. However, anyone with permission to modify policy assignments could override or remove this protection. If you choose to use Azure Policy to restrict extensions, make sure you review which accounts in your organization have permission to edit policy assignments and that appropriate change control measures are in place.

## Agent monitor mode

By default, the Connected Machine agent runs in *full mode*, which allows all extensions to be installed and used (unless restricted by allowlists, blocklists, or Azure Policy). A simple way to configure local security controls for monitoring and security scenarios is to enable *monitor mode* for the Connected Machine agent.

When the agent is in monitor mode, only extensions that are related to monitoring and security, such as the Azure Monitor Agent and Microsoft Defender for Cloud, can be deployed. The agent blocks any extensions that could change the system configuration or run arbitrary scripts, and disables the guest configuration policy agent.

As new extensions become available, Microsoft updates the monitor mode allowlist. You can review the current list of allowed extensions by running [`azcmagent config list`](azcmagent-config.md#azcmagent-config-list).

To enable monitor mode, run the following command:

```bash
azcmagent config set config.mode monitor
```

You can check the current mode of the agent and allowed extensions with the following command:

```bash
azcmagent config list
```

While in monitor mode, you can't modify the extension allowlist or blocklist. If you need to change either list, change the agent back to full mode and specify your own allowlist and blocklist instead of using monitor mode.

To change the agent back to full mode, run the following command:

```bash
azcmagent config set config.mode full
```

## Locked down machine best practices

When configuring the Azure Connected Machine agent with a reduced set of capabilities, it's important to consider the mechanisms that someone could use to remove those restrictions and implement appropriate controls. Anybody capable of running commands as an administrator or root user on the server can change the Azure Connected Machine agent configuration. Extensions and guest configuration policies execute in privileged contexts on your server, and as such might be able to change the agent configuration. If you apply local agent security controls to lock down the agent, Microsoft recommends the following best practices to ensure only local server admins can update the agent configuration:

* Use allowlists for extensions instead of blocklists whenever possible.
* Don't include the Custom Script Extension in the extension allowlist, to prevent execution of arbitrary scripts that could change the agent configuration.
* Disable Guest Configuration to prevent the use of custom Guest Configuration policies that could change the agent configuration.

### Example configuration for monitoring and security scenarios

It's common to use Azure Arc to monitor your servers with Azure Monitor and Microsoft Sentinel, and to secure them with Microsoft Defender for Cloud. This section contains examples for how to lock down the agent to only support monitoring and security scenarios.

#### Azure Monitor Agent only

On Windows servers, run the following commands in an elevated command console:

```powershell
azcmagent config set extensions.allowlist "Microsoft.Azure.Monitor/AzureMonitorWindowsAgent"
azcmagent config set guestconfiguration.enabled false
```

On Linux servers, run the following commands:

```bash
sudo azcmagent config set extensions.allowlist "Microsoft.Azure.Monitor/AzureMonitorLinuxAgent"
sudo azcmagent config set guestconfiguration.enabled false
```

#### Monitoring and security

Microsoft Defender for Cloud deploys extensions on your server to identify vulnerable software on your server and enable Microsoft Defender for Endpoint (if configured). Microsoft Defender for Cloud also uses Guest Configuration for its regulatory compliance feature. Since a custom Guest Configuration assignment could be used to undo the agent limitations, you should carefully evaluate whether you need the regulatory compliance feature and, as a result, Guest Configuration to be enabled on the machine.

On Windows servers, run the following commands in an elevated command console:

```powershell
azcmagent config set extensions.allowlist "Microsoft.EnterpriseCloud.Monitoring/MicrosoftMonitoringAgent,Qualys/WindowsAgent.AzureSecurityCenter,Microsoft.Azure.AzureDefenderForServers/MDE.Windows,Microsoft.Azure.AzureDefenderForSQL/AdvancedThreatProtection.Windows"
azcmagent config set guestconfiguration.enabled true
```

On Linux servers, run the following commands:

```bash
sudo azcmagent config set extensions.allowlist "Microsoft.EnterpriseCloud.Monitoring/OMSAgentForLinux,Qualys/LinuxAgent.AzureSecurityCenter,Microsoft.Azure.AzureDefenderForServers/MDE.Linux"
sudo azcmagent config set guestconfiguration.enabled true
```

## Disable the extension manager

If you don’t need to use extensions with Azure Arc, you can also disable the extension manager entirely. You can disable the extension manager by using the [`azcmagent config set`](azcmagent-config.md#azcmagent-config-set) command (run locally on each machine):

```bash
azcmagent config set extensions.enabled false
```

Disabling the extension manager doesn't remove any extensions already installed on your server. Extensions that are hosted in their own Windows or Linux services, such as the legacy Log Analytics Agent, might continue to run even if the extension manager is disabled. Other extensions that are hosted by the extension manager itself, such as the Azure Monitor Agent, don't run if the extension manager is disabled. To ensure no extensions continue to run on the server, [remove all extensions](manage-vm-extensions-portal.md#remove-extensions) before disabling the extension manager.
