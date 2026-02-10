---
title: Azure Arc agent
description: Learn about Azure Arc agent
ms.topic: concept-article
ms.date: 02/10/2026
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.custom:
  - build-2025
# Customer intent: As a cloud administrator, I want to understand the architecture and components of the Azure Connected Machine agent, so that I can effectively manage VMs on-premises or across different cloud providers.
---

# Azure Arc agent

When you [enable guest management](enable-guest-management-at-scale.md) on VMware VMs, the Azure Connected Machine agent is installed on the VMs. This agent is the same agent that Arc-enabled servers use. By using the Azure Connected Machine agent, you can manage your Windows and Linux machines that are hosted outside of Azure on your corporate network or other cloud providers. This article provides an architectural overview of the Azure Connected Machine agent.

## Agent components

:::image type="content" source="media/azure-arc-agent/connected-machine-agent.png" alt-text="Diagram of Azure Connected Machine agent architectural overview." lightbox="media/azure-arc-agent/connected-machine-agent.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

The Azure Connected Machine agent package contains several logical components bundled together:

* The Hybrid Instance Metadata service (HIMDS) manages the connection to Azure and the connected machine's Azure identity.

* The guest configuration agent provides functionality such as assessing whether the machine complies with required policies and enforcing compliance.

    Note the following behavior with Azure Policy [guest configuration](/azure/governance/machine-configuration/overview) for a disconnected machine:

  * An Azure Policy assignment that targets disconnected machines is unaffected.
  * The guest assignment is stored locally for 14 days. Within the 14-day period, if the Connected Machine agent reconnects to the service, policy assignments are reapplied.
  * Assignments are deleted after 14 days and aren't reassigned to the machine after the 14-day period.

* The Extension agent manages VM extensions, including install, uninstall, and upgrade. Azure downloads extensions and copies them to the `%SystemDrive%\%ProgramFiles%\AzureConnectedMachineAgent\ExtensionService\downloads` folder on Windows, and to `/opt/GC_Ext/downloads` on Linux. On Windows, the extension installs to the path `%SystemDrive%\Packages\Plugins\<extension>`, and on Linux the extension installs to `/var/lib/waagent/<extension>`.

>[!NOTE]
> The [Azure Monitor agent (AMA)](/azure/azure-monitor/agents/azure-monitor-agent-overview) is a separate agent that collects monitoring data. It doesn't replace the Connected Machine agent. The AMA only replaces the Log Analytics agent, Diagnostics extension, and Telegraf agent for both Windows and Linux machines.

## Agent resources

The following information describes the directories and user accounts that the Azure Connected Machine agent uses.

### Windows agent installation details

The Windows agent is available as a Windows Installer package (MSI). Download the Windows agent from the [Microsoft Download Center](https://aka.ms/AzureConnectedMachineAgent).
Installing the Connected Machine agent for Window applies the following system-wide configuration changes:

* The installation process creates the following folders during setup.

    | Directory | Description |
    |-----------|-------------|
    | %ProgramFiles%\AzureConnectedMachineAgent | azcmagent CLI and instance metadata service executables.|
    | %ProgramFiles%\AzureConnectedMachineAgent\ExtensionService\GC | Extension service executables.|
    | %ProgramFiles%\AzureConnectedMachineAgent\GCArcService\GC | Guest configuration (policy) service executables.|
    | %ProgramData%\AzureConnectedMachineAgent | Configuration, log, and identity token files for azcmagent CLI and instance metadata service.|
    | %ProgramData%\GuestConfig | Extension package downloads, guest configuration (policy) definition downloads, and logs for the extension and guest configuration services.|
    | %SYSTEMDRIVE%\packages | Extension package executables. |

* The installation process creates the following Windows services on the target machine.

    | Service name | Display name | Process name | Description |
    |--------------|--------------|--------------|-------------|
    | himds | Azure Hybrid Instance Metadata Service | himds | Synchronizes metadata with Azure and hosts a local REST API for extensions and applications to access the metadata and request Microsoft Entra managed identity tokens |
    | GCArcService | Guest configuration Arc Service | gc_service | Audits and enforces Azure guest configuration policies on the machine. |
    | ExtensionService | Guest configuration Extension Service | gc_service | Installs, updates, and manages extensions on the machine. |

* The installation process creates the following virtual service account.

    | Virtual Account  | Description |
    |------------------|-------------|
    | NT SERVICE\\himds | Unprivileged account used to run the Hybrid Instance Metadata Service. |

    > [!TIP]
    > This account requires the *Log on as a service* right. The installation process automatically grants this right, but if your organization configures user rights assignments by using Group Policy, you might need to adjust your Group Policy Object to grant the right to  **NT SERVICE\\himds** or **NT SERVICE\\ALL SERVICES** to allow the agent to function.

* The installation process creates the following local security group.

    | Security group name | Description |
    |---------------------|-------------|
    | Hybrid agent extension applications | Members of this security group can request Microsoft Entra tokens for the system-assigned managed identity |

* The installation process creates the following environmental variables

    | Name | Default value | Description |
    |------|---------------|------------|
    | IDENTITY_ENDPOINT | `http://localhost:40342/metadata/identity/oauth2/token` |
    | IMDS_ENDPOINT | `http://localhost:40342` |

* Several log files are available for troubleshooting, as described in the following table.

    | Log | Description |
    |-----|-------------|
    | %ProgramData%\AzureConnectedMachineAgent\Log\himds.log | Records details of the heartbeat and identity agent component. |
    | %ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log | Contains the output of the azcmagent tool commands. |
    | %ProgramData%\GuestConfig\arc_policy_logs\gc_agent.log | Records details about the guest configuration (policy) agent component. |
    | %ProgramData%\GuestConfig\ext_mgr_logs\gc_ext.log | Records details about extension manager activity (extension install, uninstall, and upgrade events). |
    | %ProgramData%\GuestConfig\extension_logs | Directory containing logs for individual extensions. |

* The process creates the local security group **Hybrid agent extension applications**.

* After uninstalling the agent, the following artifacts remain:

  * `%ProgramData%\AzureConnectedMachineAgent\Log`
  * `%ProgramData%\AzureConnectedMachineAgent`
  * `%ProgramData%\GuestConfig`
  * `%SystemDrive%\packages`

### Linux agent installation details

The Connected Machine agent for Linux is available in the preferred package format (`.rpm` or `.deb`) for your distribution in the Microsoft [package repository](https://packages.microsoft.com/). The shell script bundle [Install_linux_azcmagent.sh](https://aka.ms/azcmagent) installs and configures the agent.

You don't need to install, upgrade, or remove the Connected Machine agent after a server restart.

When you install the Connected Machine agent for Linux, it makes the following system-wide configuration changes:

* Setup creates the following installation folders.

    | Directory | Description |
    |-----------|-------------|
    | /opt/azcmagent/ | azcmagent CLI and instance metadata service executables. |
    | /opt/GC_Ext/ | Extension service executables. |
    | /opt/GC_Service/ | Guest configuration (policy) service executables. |
    | /var/opt/azcmagent/ | Configuration, log, and identity token files for azcmagent CLI and instance metadata service.|
    | /var/lib/GuestConfig/ | Extension package downloads, guest configuration (policy) definition downloads, and logs for the extension and guest configuration services.|

* Installing the agent creates the following daemons.

    | Service name | Display name | Process name | Description |
    |--------------|--------------|--------------|-------------|
    | himdsd.service | Azure Connected Machine Agent Service | himds | This service implements the Hybrid Instance Metadata service (IMDS) to manage the connection to Azure and the connected machine's Azure identity.|
    | gcad.service | GC Arc Service | gc_linux_service | Audits and enforces Azure guest configuration policies on the machine. |
    | extd.service | Extension Service | gc_linux_service | Installs, updates, and manages extensions on the machine. |

* Several log files are available for troubleshooting, as described in the following table.

    | Log | Description |
    |-----|-------------|
    | /var/opt/azcmagent/log/himds.log | Records details of the heartbeat and identity agent component. |
    | /var/opt/azcmagent/log/azcmagent.log | Contains the output of the azcmagent tool commands. |
    | /var/lib/GuestConfig/arc_policy_logs | Records details about the guest configuration (policy) agent component. |
    | /var/lib/GuestConfig/ext_mgr_logs | Records details about extension manager activity (extension install, uninstall, and upgrade events). |
    | /var/lib/GuestConfig/extension_logs | Directory containing logs for individual extensions. |

* Agent installation creates the following environment variables, set in `/lib/systemd/system.conf.d/azcmagent.conf`.

    | Name | Default value | Description |
    |------|---------------|-------------|
    | IDENTITY_ENDPOINT | `http://localhost:40342/metadata/identity/oauth2/token` |
    | IMDS_ENDPOINT | `http://localhost:40342` |

* After uninstalling the agent, the following artifacts remain:

  * /var/opt/azcmagent
  * /var/lib/GuestConfig

## Agent resource governance

The Azure Connected Machine agent manages agent and system resource consumption. The agent follows these resource governance rules:

* The Guest Configuration agent uses up to 5% of the CPU to evaluate policies.
* The Extension Service agent uses up to 5% of the CPU to install, upgrade, run, and delete extensions. Some extensions apply more restrictive CPU limits once installed. The following exceptions apply:

  | Extension type | Operating system | CPU limit |
  | -------------- | ---------------- | --------- |
  | AzureMonitorLinuxAgent | Linux | 60% |
  | AzureMonitorWindowsAgent | Windows | 100% |
  | AzureSecurityLinuxAgent | Linux | 30% |
  | LinuxOsUpdateExtension | Linux | 60% |
  | MDE.Linux | Linux | 60% |
  | MicrosoftDnsAgent | Windows | 100% |
  | MicrosoftMonitoringAgent | Windows | 60% |
  | OmsAgentForLinux | Windows | 60%|

During normal operations, defined as the Azure Connected Machine agent being connected to Azure and not actively modifying an extension or evaluating a policy, the agent consumes the following system resources:

|     | Windows | Linux |
| --- | ------- | ----- |
| **CPU usage (normalized to 1 core)** | 0.07% | 0.02% |
| **Memory usage** | 57 MB | 42 MB |

The performance data was gathered in April 2023 on virtual machines running Windows Server 2022 and Ubuntu 20.04. The actual agent performance and resource consumption vary based on the hardware and software configuration of your servers.

## Instance metadata

The Connected Machine agent collects metadata about a connected machine after it registers with Azure Arc-enabled servers. The agent collects the following metadata:

* Operating system name, type, and version
* Computer name
* Computer manufacturer and model
* Computer fully qualified domain name (FQDN)
* Domain name (if joined to an Active Directory domain)
* Active Directory and DNS fully qualified domain name (FQDN)
* UUID (BIOS ID)
* Connected Machine agent heartbeat
* Connected Machine agent version
* Public key for managed identity
* Policy compliance status and details (if using guest configuration policies)
* SQL Server installed (Boolean value)
* Cluster resource ID (for Azure Local nodes)
* Hardware manufacturer
* Hardware model
* CPU family, socket, physical core, and logical core counts
* Total physical memory
* Serial number
* SMBIOS asset tag
* Cloud provider
* Amazon Web Services (AWS) metadata, when running in AWS:
  * Account ID
  * Instance ID
  * Region
* Google Cloud Platform (GCP) metadata, when running in GCP:
  * Instance ID
  * Image
  * Machine type
  * Project ID
  * Project number
  * Service accounts
  * Zone

The agent requests the following metadata information from Azure:

* Resource location (region)
* Virtual machine ID
* Tags
* Microsoft Entra managed identity certificate
* Guest configuration policy assignments
* Extension requests - install, update, and delete.

> [!NOTE]
> Azure Arc-enabled servers don't store or process customer data outside the region where the customer deploys the service instance.

## Next steps

- [Connect VMware vCenter Server to Azure Arc](quick-start-connect-vcenter-to-arc-using-script.md).
- [Install Arc agent at scale for your VMware VMs](enable-guest-management-at-scale.md).
