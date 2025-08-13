---
title: Archive for What's new with Azure Connected Machine agent
description: Release notes for Azure Connected Machine agent versions older than six months
ms.topic: overview
ms.date: 08/12/2025
ms.custom: references_regions
# Customer intent: "As an IT administrator managing hybrid environments, I want to access detailed release notes for older versions of the Azure Connected Machine agent, so that I can understand issues about these agent versions, even though they are no longer actively supported."
---

# Archive for What's new with Azure Connected Machine agent

> [!CAUTION]
> This article references CentOS, a Linux distribution that is End Of Life (EOL) status. Please consider your use and planning accordingly. For more information, see the [CentOS End Of Life guidance](/azure/virtual-machines/workloads/centos/centos-end-of-life).

This article contains information about older releases of the Connected Machine agent. The primary [What's new in Azure Connected Machine agent?](agent-release-notes.md) article contains updates for the last six months. Microsoft recommends staying up to date with the latest agent version whenever possible.

## Version 1.48 - January 2025

Download for [Windows](https://download.microsoft.com/download/2/8/6/2867a351-6546-4af8-b97f-cc4483ef4192/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Addressed a security issue related to Redirection Guard.
- Resolved a bug that caused issues during the upgrade of the RunCommand extension.
- Fixed high port usage in the Azure Arc proxy.
- Fixed an issue with the Alma Linux install script.
- Improved handling of disassociated Gateway URLs.
- Resolved an issue with disk space queries.
- Improved HIMDS behavior when IMDS data is unavailable.

### New features and enhancements

- Added support for extension telemetry.
- Updated the OpenSSL library for security enhancements.
- Improved error reporting for `azcmagent` commands.
- Increased connectivity check timeout for better reliability.
- Expanded ARM64 platform support to include RHEL 9.
- Updated the `mssqldiscovered` property to include the detection for SQL Server Integration, Analysis, and Reporting services (SSIS, SSAS, SSRS and PBIRS).
- Introduced a scheduled task that checks for agent updates on a daily basis. Currently, the update mechanism is inactive and no changes are made to your server even if a newer agent version is available.

## Version 1.47 - October 2024

Download for [Windows](https://download.microsoft.com/download/2/1/d/21dfb0f5-ed95-46d5-8146-ece13381056a/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Guest Configuration: Fix an issue that caused agent to become unresponsive.
- Fixed a bug to trim error messages when updating AgentStatus.

### New features and enhancements

- Code enhancement to support cloud specific endpoints in the install script.
- Addition of architecture detection to system properties.
- Addition of EndpointConnectivityInfo to AgentData.
- Expansion of ARM64 platform support for the following distributions:
  - Ubuntu 20.04, 22.04, 24.04
  - Azure Linux (CBL-Mariner) 2.0
  - Amazon Linux 2
  - Alma Linux 8


## Version 1.46 - September 2024

Download for [Windows](https://download.microsoft.com/download/6/c/0/6c0775bc-9ed7-49af-9637-79653f783062/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Fixed a bug causing the Guest Config agent to hang in extension creating state when the download of an extension package failed.
- Fixed a bug where onboarding treated conflicting errors as success.

### New features and enhancements

- Improved error messaging for scenarios with extension installation and enablement blockage in the presence of a sideloaded extension.
- Increased checks for recovery of sequence number if the previous request failed.
- Removed casing requirements when reading the proxy from the configuration file.
- Added supported for Azure Linux 3 (Mariner).
- Added initial Linux ARM64 architecture support.
- Added Gateway URL to the output of the show command.

## Version 1.45 - August 2024

Download for [Windows](https://download.microsoft.com/download/0/6/1/061e3c68-5603-4c0e-bb78-2e3fd10fef30/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Fixed an issue where EnableEnd telemetry would sometimes be sent too soon.
- Added sending a failed timed-out EnableEnd telemetry log if extension takes longer than the allowed time to complete.

### New features

- Azure Arc proxy now supports HTTP traffic.
- New proxy.bypass value 'AMA' added to support AMA VM extension proxy bypass.

## Version 1.44 - July 2024

Download for [Windows](https://download.microsoft.com/download/d/a/f/daf3cc3e-043a-430a-abae-97142323d4d7/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Fixed a bug where the service would sometimes reject reports from an upgraded extension if the previous extension was in a failed state.
- Setting OPENSSL_CNF environment at process level to override build openssl.cnf path on Windows.
- Fixed access denied errors in writing configuration files.
- Fixed SYMBIOS GUID related bug with Windows Server 2012 and Windows Server 2012 R2 [Extended Security Updates](/windows-server/get-started/extended-security-updates-overview) enabled by Azure Arc.

### New features

- Extension service enhancements: Added download/validation error details to extension report. Increased unzipped extension package size limit to 1 GB.
- Update of hardwareprofile information to support upcoming Windows Server licensing capabilities.
- Update of the error json output to include more detailed recommended actions for troubleshooting scenarios.
- Block on installation of unsupported operating systems and distribution versions. See [Supported operating systems](prerequisites.md#supported-operating-systems) for details.

> [!NOTE]
> Azure Connected Machine agent version 1.44 is the last version to officially support Debian 10, Ubuntu 16.04, and Azure Linux (CBL-Mariner) 1.0.

## Version 1.43 - June 2024

Download for [Windows](https://download.microsoft.com/download/0/7/8/078f3bb7-6a42-41f7-b9d3-9a0eb4c94df8/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Fix for OpenSSL Vulnerability for Linux (Upgrading OpenSSL version from 3.0.13 to 3.014)
- Added Server Name Indicator (SNI) to our service calls, fixing Proxy and Firewall scenarios
- Skipped lockdown policy on the downloads directory under Guest Configuration

## Version 1.42 - May 2024 (Second release)

Download for [Windows](https://download.microsoft.com/download/9/6/0/9600825a-e532-4e50-a2d5-7f07e400afc1/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Extensions and machine configuration policies can be used with private endpoints again

## Version 1.41 - May 2024

Download for [Windows](https://download.microsoft.com/download/2/a/5/2a57aa86-c445-4f08-bd52-10af2b748fec/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

Customers using private endpoints with Azure Arc may encounter issues with extension management and machine configuration policies with agent version 1.41. Agent version 1.42 resolves this issue.

### New features

- Certificate-based authentication is now supported when using a service principal to connect or disconnect the agent. For more information, see [authentication options for the azcmagent CLI](azcmagent-connect.md#authentication-options).
- [azcmagent check](azcmagent-check.md) now allows you to also check for the endpoints used by the SQL Server enabled by Azure Arc extension using the new `--extensions` flag. This can help you troubleshoot networking issues for both the OS and SQL management components. You can try this out by running `azcmagent check --extensions sql --location eastus` on a server, either before or after it is connected to Azure Arc.

### Fixed

- Fixed a memory leak in the Hybrid Instance Metadata service
- Better handling when IPv6 local loopback is disabled
- Improved reliability when upgrading extensions
- Improved reliability when enforcing CPU limits on Linux extensions
- PowerShell telemetry is now disabled by default for the extension manager and policy services
- The extension manager and policy services now support OpenSSL 3
- Colors are now disabled in the onboarding progress bar when the `--no-color` flag is used
- Improved detection and reporting for Windows machines that have custom [logon as a service rights](prerequisites.md#local-user-logon-right-for-windows-systems) configured.
- Improved accuracy when obtaining system metadata on Windows:
  - VMUUID is now obtained from the Win32 API
  - Physical memory is now checked using WMI
- Fixed an issue that could prevent the region selector in the [Windows GUI installer](onboard-windows-server.md) from loading
- Fixed permissions issues that could prevent the "himds" service from accessing necessary directories on Windows

## Version 1.40 - April 2024

Download for [Windows](https://download.microsoft.com/download/2/1/0/210f77ca-e069-412b-bd94-eac02a63255d/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

The first release of the 1.40 agent may impact SQL Server enabled by Azure Arc when configured with least privileges on Windows servers. The 1.40 agent was re-released to address this problem. To check if your server is affected, run `azcmagent show` and locate the agent version number. Agent version `1.40.02664.1629` has the known issue and agent `1.40.02669.1635` fixes it. Download and install the [latest version of the agent](https://aka.ms/AzureConnectedMachineAgent) to restore functionality for SQL Server enabled by Azure Arc.

### New features

- Oracle Linux 9 is now a [supported operating system](prerequisites.md#supported-operating-systems)
- Customers no longer need to download an intermediate CA certificate for delivery of WS2012/R2 ESUs (Requires April 2024 SSU update)

### Fixed

- Improved error handling when a machine configuration policy has an invalid SAS token
- The installation script for Windows now includes a flag to suppress reboots in case any agent executables are in use during an upgrade
- Fixed an issue that could block agent installation or upgrades on Windows when the installer can't change the access control list on the agent's log directories.
- Extension package maximum download size increased to fix access to the [latest versions of the Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-extension-versions) on Azure Arc-enabled servers.

## Version 1.39 - March 2024

Download for [Windows](https://download.microsoft.com/download/1/9/f/19f44dde-2c34-4676-80d7-9fa5fc44d2a8/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Check which extensions are installed and manually remove them with the new [azcmagent extension](azcmagent-extension.md) command group. These commands run locally on the machine and work even if a machine has lost its connection to Azure.
- You can now [customize the CPU limit](agent-overview.md#custom-resource-limits) applied to the extension manager and machine configuration policy evaluation engine. This might be helpful on small or under-powered VMs where the [default resource governance limits](agent-overview.md#agent-resource-governance) can cause extension operations to time out.

### Fixed

- Improved reliability of the run command feature with long-running commands
- Removed an unnecessary endpoint from the network connectivity check when onboarding machines via an Azure Arc resource bridge
- Improved heartbeat reliability
- Removed unnecessary dependencies

## Version 1.38 - February 2024

Download for [Windows](https://download.microsoft.com/download/4/8/f/48f69eb1-f7ce-499f-b9d3-5087f330ae79/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

Windows machines that try and fail to upgrade to version 1.38 manually or via Microsoft Update might not roll back to the previously installed version. As a result, the machine will appear "Disconnected" and won't be manageable from Azure. A new version of 1.38 was released to Microsoft Update and the Microsoft Download Center on March 5, 2024 that resolves this issue.

If your machine was affected by this issue, you can repair the agent by downloading and installing the agent again. The agent will automatically discover the existing configuration and restore connectivity with Azure. You don't need to run `azcmagent connect`.

### New features

- AlmaLinux 9 is now a [supported operating system](prerequisites.md#supported-operating-systems)

### Fixed

- The hybrid instance metadata service (HIMDS) now listens on the IPv6 local loopback address (::1)
- Improved logging in the extension manager and policy engine
- Improved reliability when fetching the latest operating system metadata
- Reduced extension manager CPU usage

## Version 1.37 - December 2023

Download for [Windows](https://download.microsoft.com/download/f/6/4/f64c574f-d3d5-4128-8308-ed6a7097a93d/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Rocky Linux 9 is now a [supported operating system](prerequisites.md#supported-environments)
- Added Oracle Cloud Infrastructure display name as a [detected property](agent-overview.md#instance-metadata)

### Fixed

- Restored access to servers with Windows Admin Center in Azure
- Improved detection logic for Microsoft SQL Server
- Agents connected to sovereign clouds should now see the correct cloud and portal URL in [azcmagent show](azcmagent-show.md)
- The installation script for Linux now automatically approves the request to import the packages.microsoft.com signing key to ensure a silent installation experience
- Agent installation and upgrades apply more restrictive permissions to the agent's data directories on Windows
- Improved reliability when detecting Azure Stack HCI as a cloud provider
- Removed the log zipping feature introduced in version 1.37 for extension manager and machine configuration agent logs. Log files are still rotated automatically.
- Removed the scheduled tasks for automatic agent upgrades (introduced in agent version 1.30). We'll reintroduce this functionality when the automatic upgrade mechanism is available.
- Resolved [Azure Connected Machine Agent Elevation of Privilege Vulnerability](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-35624)

## Version 1.36 - November 2023

Download for [Windows](https://download.microsoft.com/download/5/e/9/5e9081ed-2ee2-4b3a-afca-a8d81425bcce/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

The Windows Admin Center in Azure feature is incompatible with Azure Connected Machine agent version 1.36. Upgrade to version 1.37 or later to use this feature.

### New features

- [azcmagent show](azcmagent-show.md) now reports extended security license status on Windows Server 2012 server machines.
- Introduced a new [proxy bypass](manage-agent.md#proxy-bypass-for-private-endpoints) option, `ArcData`, that covers the SQL Server enabled by Azure Arc endpoints. This enables you to use a private endpoint with Azure Arc-enabled servers with the public endpoints for SQL Server enabled by Azure Arc.
- The [CPU limit for extension operations](agent-overview.md#agent-resource-governance) on Linux is now 30%. This increase helps improve reliability of extension install, upgrade, and uninstall operations.
- Older extension manager and machine configuration agent logs are automatically zipped to reduce disk space requirements.
- New executable names for the extension manager (`gc_extension_service`) and machine configuration (`gc_arc_service`) agents on Windows to help you distinguish the two services. For more information, see [Windows agent installation details](./agent-overview.md#windows-agent-installation-details).

### Bug fixes

- [azcmagent connect](azcmagent-connect.md) now uses the latest API version when creating the Azure Arc-enabled server resource to ensure Azure policies targeting new properties can take effect.
- Upgraded the OpenSSL library and PowerShell runtime shipped with the agent to include the latest security fixes.
- Fixed an issue that could prevent the agent from reporting the correct product type on Windows machines.
- Improved handling of upgrades when the previously installed extension version wasn't in a successful state.

## Version 1.35 - October 2023

Download for [Windows](https://download.microsoft.com/download/e/7/0/e70b1753-646e-4aea-bac4-40187b5128b0/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

The Windows Admin Center in Azure feature is incompatible with Azure Connected Machine agent version 1.35. Upgrade to version 1.37 or later to use this feature.

### New features

- The Linux installation script now downloads supporting assets with either wget or curl, depending on which tool is available on the system
- [azcmagent connect](azcmagent-connect.md) and [azcmagent disconnect](azcmagent-disconnect.md) now accept the `--user-tenant-id` parameter to enable Lighthouse users to use a credential from their tenant and onboard a server to a different tenant.
- You can configure the extension manager to run, without allowing any extensions to be installed, by configuring the allowlist to `Allow/None`. This supports Windows Server 2012 ESU scenarios where the extension manager is required for billing purposes but doesn't need to allow any extensions to be installed. Learn more about [local security controls](security-extensions.md#local-agent-security-controls).

### Fixed

- Improved reliability when installing Microsoft Defender for Endpoint on Linux by increasing [available system resources](agent-overview.md#agent-resource-governance) and extending the timeout
- Better error handling when a user specifies an invalid location name to [azcmagent connect](azcmagent-connect.md)
- Fixed a bug where clearing the `incomingconnections.enabled` [configuration setting](azcmagent-config.md) would show `<nil>` as the previous value
- Security fix for the extension allowlist and blocklist feature to address an issue where an invalid extension name could impact enforcement of the lists.

## Version 1.34 - September 2023

Download for [Windows](https://download.microsoft.com/download/b/3/2/b3220316-13db-4f1f-babf-b1aab33b364f/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- [Extended Security Updates for Windows Server 2012 and 2012 R2](prepare-extended-security-updates.md) can be purchased and enabled through Azure Arc. If your server is already running the Azure Connected Machine agent, [upgrade to agent version 1.34](manage-agent.md#upgrade-the-agent) or later to take advantage of this new capability.
- New system metadata is collected to enhance your device inventory in Azure:
  - Total physical memory
  - More processor information
  - Serial number
  - SMBIOS asset tag
- Network requests to Microsoft Entra ID (formerly Azure Active Directory) now use `login.microsoftonline.com` instead of `login.windows.net`

### Fixed

- Better handling of disconnected agent scenarios in the extension manager and policy engine.

## Version 1.33 - August 2023

Download for [Windows](https://download.microsoft.com/download/0/c/7/0c7a484b-e29e-42f9-b3e9-db431df2e904/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Security fix

Agent version 1.33 contains a fix for [CVE-2023-38176](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2023-38176), a local elevation of privilege vulnerability. Microsoft recommends upgrading all agents to version 1.33 or later to mitigate this vulnerability. Azure Advisor can help you [identify servers that need to be upgraded](https://portal.azure.com/#view/Microsoft_Azure_Expert/RecommendationListBlade/recommendationTypeId/9d5717d2-4708-4e3f-bdda-93b3e6f1715b/recommendationStatus). Learn more about CVE-2023-38176 in the [Security Update Guide](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2023-38176).

### Known issue

[azcmagent check](azcmagent-check.md) validates a new endpoint in this release: `<geography>-ats.his.arc.azure.com`. This endpoint is reserved for future use and not required for the Azure Connected Machine agent to operate successfully. However, if you're using a private endpoint, this endpoint will fail the network connectivity check. You can safely ignore this endpoint in the results and should instead confirm that all other endpoints are reachable.

This endpoint will be removed from `azcmagent check` in a future release.

### Fixed

- Fixed an issue that could cause a VM extension to disappear in Azure Resource Manager if it's installed with the same settings twice. After upgrading to agent version 1.33 or later, reinstall any missing extensions to restore the information in Azure Resource Manager.
- You can now set the [agent mode](security-extensions.md#agent-monitor-mode) before connecting the agent to Azure.
- The agent now responds to instance metadata service (IMDS) requests even when the connection to Azure is temporarily unavailable.

## Version 1.32 - July 2023

Download for [Windows](https://download.microsoft.com/download/7/e/5/7e51205f-a02e-4fbe-94fe-f36219be048c/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Added support for the Debian 12 operating system
- [azcmagent show](azcmagent-show.md) now reflects the "Expired" status when a machine has been disconnected long enough for the managed identity to expire. Previously, the agent only showed "Disconnected" while the Azure portal and API showed the correct state, "Expired."

### Fixed

- Fixed an issue that could result in high CPU usage if the agent was unable to send telemetry to Azure.
- Improved local logging when there are network communication errors

## Version 1.31 - June 2023

Download for [Windows](https://download.microsoft.com/download/2/6/e/26e2b001-1364-41ed-90b0-1340a44ba409/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issue

The first release of agent version 1.31 had a known issue affecting customers using proxy servers. The issue displays as  `AZCM0026: Network Error` and a message about "no IP addresses found" when connecting a server to Azure Arc using a proxy server. A newer version of agent 1.31 was released on June 14, 2023 that addresses this issue.

To check if you're running the latest version of the Azure connected machine agent, navigate to the server in the Azure portal or run `azcmagent show` from a terminal on the server itself and look for the "Agent version." The table below shows the version numbers for the first and patched releases of agent 1.31.

| Package type | Version number with proxy issue | Version number of patched agent |
| ------------ | ------------------------------- | ------------------------------- |
| Windows | 1.31.02347.1069 | 1.31.02356.1083 |
| RPM-based Linux | 1.31.02347.957 | 1.31.02356.970 |
| DEB-based Linux | 1.31.02347.939 | 1.31.02356.952 |

### New features

- Added support for Amazon Linux 2023
- [azcmagent show](azcmagent-show.md) no longer requires administrator privileges
- You can now filter the output of [azcmagent show](azcmagent-show.md) by specifying the properties you wish to output

### Fixed

- Added an error message when a pending reboot on the machine affects extension operations
- The scheduled task that checks for agent updates no longer outputs a file
- Improved formatting for clock skew calculations
- Improved reliability when upgrading extensions by explicitly asking extensions to stop before trying to upgrade.
- Increased the [resource limits](agent-overview.md#agent-resource-governance) for the Update Manager extension for Linux, Microsoft Defender Endpoint for Linux, and Azure Security Agent for Linux to prevent timeouts during installation
- [azcmagent disconnect](azcmagent-disconnect.md) now closes any active SSH or Windows Admin Center connections
- Improved output of the [azcmagent check](azcmagent-check.md) command
- Better handling of spaces in the `--location` parameter of [azcmagent connect](azcmagent-connect.md)

## Version 1.30 - May 2023

Download for [Windows](https://download.microsoft.com/download/7/7/9/779eae73-a12b-4170-8c5e-abec71bc14cf/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Introduced a scheduled task that checks for agent updates on a daily basis. Currently, the update mechanism is inactive and no changes are made to your server even if a newer agent version is available.

### Fixed

- Resolved an issue that could cause the agent to go offline after rotating its connectivity keys.
- `azcmagent show` no longer shows an incomplete resource ID or Azure portal page URL when the agent isn't configured.

## Version 1.29 - April 2023

Download for [Windows](https://download.microsoft.com/download/2/7/0/27063536-949a-4b16-a29a-3d1dcb29cff7/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- The agent now compares the time on the local system and Azure service when checking network connectivity and creating the resource in Azure. If the clocks are offset by more than 120 seconds (2 minutes), a nonblocking error is shown. You might encounter TLS connection errors if the time of your computer doesn't match the time in Azure.
- `azcmagent show` now supports an `--os` flag to print extra OS information to the console

### Fixed

- Fixed an issue that could cause the guest configuration service (gc_service) to repeatedly crash and restart on Linux systems
- Resolved a rare condition under which the guest configuration service (gc_service) could consume excessive CPU resources
- Removed "sudo" calls in internal install script that could be blocked if SELinux is enabled
- Reduced how long network checks wait before determining a network endpoint is unreachable
- Stopped writing error messages in "himds.log" referring to a missing certificate key file for the ATS agent, an inactive component reserved for future use.

## Version 1.28 - March 2023

Download for [Windows](https://download.microsoft.com/download/5/9/7/59789af8-5833-4c91-8dc5-91c46ad4b54f/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Improved reliability of delete requests for extensions
- More frequent reporting of VM UUID (system firmware identifier) changes
- Improved reliability when writing changes to agent configuration files
- JSON output for `azcmagent connect` now includes Azure portal URL for the server
- Linux installation script now installs the `gnupg` package if it's missing on Debian operating systems
- Removed weekly restarts for the extension and guest configuration services

## Version 1.27 - February 2023

Download for [Windows](https://download.microsoft.com/download/8/4/5/845d5e04-bb09-4ed2-9ca8-bb51184cddc9/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- The extension service now correctly restarts when the Azure Connected Machine agent is upgraded by Update Manager
- Resolved issues with the hybrid connectivity component that could result in the "himds" service crashing, the server showing as "disconnected" in Azure, and connectivity issues with Windows Admin Center and SSH
- Improved handling of resource move scenarios that could impact Windows Admin Center and SSH connectivity
- Improved reliability when changing the [agent configuration mode](security-extensions.md#agent-monitor-mode) from "monitor" mode to "full" mode.
- Increased the [resource limits](agent-overview.md#agent-resource-governance) for the Microsoft Sentinel DNS extension to improve log collection reliability
- Tenant IDs are better validated when connecting the server

## Version 1.26 - January 2023

Download for [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

> [!NOTE]
> Version 1.26 is only available for Linux operating systems.

### Fixed

- Increased the [resource limits](agent-overview.md#agent-resource-governance) for the Microsoft Defender for Endpoint extension (MDE.Linux) on Linux to improve installation reliability

## Version 1.25 - January 2023

Download for [Windows](https://download.microsoft.com/download/2/a/5/2a5b7d19-bc35-443e-80b2-63087577236e/AzureConnectedMachineAgent%20(1).msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Red Hat Enterprise Linux (RHEL) 9 is now a [supported operating system](prerequisites.md#supported-operating-systems)

### Fixed

- Reliability improvements in the machine (guest) configuration policy engine
- Improved error messages in the Windows MSI installer
- Additional improvements to the detection logic for machines running on Azure Stack HCI

## Version 1.24 - November 2022

Download for [Windows](https://download.microsoft.com/download/f/9/d/f9d60cc9-7c2a-4077-b890-f6a54cc55775/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- `azcmagent logs` improvements:
  - Only the most recent log file for each component is collected by default. To collect all log files, use the new `--full` flag.
  - Journal logs for the agent services are now collected on Linux operating systems
  - Logs from extensions are now collected
- Agent telemetry is no longer sent to `dc.services.visualstudio.com`. You might be able to remove this URL from any firewall or proxy server rules if no other applications in your environment require it.
- Failed extension installs can now be retried without removing the old extension as long as the extension settings are different
- Increased the [resource limits](agent-overview.md#agent-resource-governance) for the Azure Update Manager extension on Linux to reduce downtime during update operations

### Fixed

- Improved logic for detecting machines running on Azure Stack HCI to reduce false positives
- Auto-registration of required resource providers only happens when they are unregistered
- Agent will now detect drift between the proxy settings of the command line tool and background services
- Fixed a bug with proxy bypass feature that caused the agent to incorrectly use the proxy server for bypassed URLs
- Improved error handling when extensions don't download successfully, fail validation, or have corrupt state files

## Version 1.23 - October 2022

Download for [Windows](https://download.microsoft.com/download/3/9/8/398f6036-958d-43c4-ad7d-4576f1d860aa/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- The minimum PowerShell version required on Windows Server has been reduced to PowerShell 4.0
- The Windows agent installer is now compatible with systems that enforce a Microsoft publisher-based Windows Defender Application Control policy.
- Added support for Rocky Linux 8 and Debian 11.

### Fixed

- Tag values are correctly preserved when connecting a server and specifying multiple tags (fixes known issue from version 1.22).
- An issue preventing some users who tried authenticating with an identity from a different tenant than the tenant where the server is (will be) registered has been fixed.
- The `azcamgent check` command no longer validates CNAME records to reduce warnings that did not impact agent functionality.
- The agent will now try to obtain an access token for up to 5 minutes when authenticating with an Azure Active Directory service principal.
- Cloud presence checks now only run once at the time the `himds` service starts on the server to reduce local network traffic. If you live migrate your virtual machine to a different cloud provider, it will not reflect the new cloud provider until the service or computer has rebooted.
- Improved logging during the installation process.
- The install script for Windows now saves the MSI to the TEMP directory instead of the current directory.

## Version 1.22 - September 2022

Download for [Windows](https://download.microsoft.com/download/1/3/5/135f1f2b-7b14-40f6-bceb-3af4ebadf434/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

- The 'connect' command uses the value of the last tag for all tags. You will need to fix the tags after onboarding to use the correct values.

### New features

- The default login flow for Windows computers now loads the local web browser to authenticate with Azure Active Directory instead of providing a device code. You can use the `--use-device-code` flag to return to the old behavior or [provide service principal credentials](onboard-service-principal.md) for a non-interactive authentication experience.
- If the resource group provided to `azcmagent connect` does not exist, the agent tries to create it and continue connecting the server to Azure.
- Added support for Ubuntu 22.04
- Added `--no-color` flag for all azcmagent commands to suppress the use of colors in terminals that do not support ANSI codes.

### Fixed

- The agent now supports Red Hat Enterprise Linux 8 servers that have FIPS mode enabled.
- Agent telemetry uses the proxy server when configured.
- Improved accuracy of network connectivity checks
- The agent retains extension allow and blocklists when switching the agent from monitoring mode to full mode. Use [azcmagent config clear](azcmagent-config.md) to reset individual configuration settings to the default state.

## Version 1.21 - August 2022

Download for [Windows](https://download.microsoft.com/download/a/4/c/a4cce0a3-5830-4bd4-a4aa-a26794690e3d/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- `azcmagent connect` usability improvements:
  - The `--subscription-id (-s)` parameter now accepts friendly names in addition to subscription IDs
  - Automatic registration of any missing resource providers for first-time users (extra user permissions required to register resource providers)
  - Added a progress bar during onboarding
  - The onboarding script now supports both the yum and dnf package managers on RPM-based Linux systems
- You can now restrict the URLs used to download machine configuration (formerly Azure Policy guest configuration) packages by setting the `allowedGuestConfigPkgUrls` tag on the server resource and providing a comma-separated list of URL patterns to allow.

### Fixed

- Improved reliability when reporting extension installation failures to prevent extensions from staying in the "creating" state
- Support for retrieving metadata for Google Cloud Platform virtual machines when the agent uses a proxy server
- Improved network connection retry logic and error handling
- Linux only: resolves local escalation of privilege vulnerability [CVE-2022-38007](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-38007)

## Version 1.20 - July 2022

Download for [Windows](https://download.microsoft.com/download/f/b/1/fb143ada-1b82-4d19-a125-40f2b352e257/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

- Some systems might incorrectly report their cloud provider as Azure Stack HCI.

### New features

- Added support for connecting the agent to the Microsoft Azure operated by 21Vianet cloud
- Added support for Debian 10
- Updates to the [instance metadata](agent-overview.md#instance-metadata) collected on each machine:
  - GCP VM OS is no longer collected
  - CPU logical core count is now collected
- Improved error messages and colorization

### Fixed

- Agents configured to use private endpoints correctly download extensions over the private endpoint
- Renamed the `--use-private-link` flag on [azcmagent check](azcmagent-check.md) to `--enable-pls-check` to more accurately represent its function

## Version 1.19 - June 2022

Download for [Windows](https://download.microsoft.com/download/8/9/f/89f80a2b-32c3-43e8-b3b8-fce6cea8e2cf/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

- Agents configured to use private endpoints incorrectly download extensions from a public endpoint. [Upgrade the agent](manage-agent.md#upgrade-the-agent) to version 1.20 or later to restore correct functionality.
- Some systems might incorrectly report their cloud provider as Azure Stack HCI.

### New features

- When installed on a Google Compute Engine virtual machine, the agent detects and reports Google Cloud metadata in the "detected properties" of the Azure Arc-enabled servers resource. [Learn more](agent-overview.md#instance-metadata) about the new metadata.

### Fixed

- Resolved an issue that could cause the extension manager to hang during extension installation, update, and removal operations.
- Improved support for TLS 1.3

## Version 1.18 - May 2022

Download for [Windows](https://download.microsoft.com/download/2/5/6/25685d0f-2895-4b80-9b1d-5ba53a46097f/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- You can configure the agent to operate in [monitoring mode](security-extensions.md#agent-monitor-mode), which simplifies configuration of the agent for scenarios where you only want to use Arc for monitoring and security scenarios. This mode disables other agent functionality and prevents use of extensions that could make changes to the system (for example, the Custom Script Extension).
- VMs and hosts running on Azure Stack HCI now report the cloud provider as "HCI" when [Azure benefits are enabled](/azure-stack/hci/manage/azure-benefits#enable-azure-benefits).

### Fixed

- `systemd` is now an official prerequisite on Linux
- Guest configuration policies no longer create unnecessary files in the `/tmp` directory on Linux servers
- Improved reliability when extracting extensions and guest configuration policy packages
- Improved reliability for guest configuration policies that have child processes

## Version 1.17 - April 2022

Download for [Windows](https://download.microsoft.com/download/a/3/4/a34bb824-d563-4ebf-bcbf-d5c5e7aaf4a3/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- The default resource name for AWS EC2 instances is now the instance ID instead of the hostname. To override this behavior, use the `--resource-name PreferredResourceName` parameter to specify your own resource name when connecting a server to Azure Arc.
- The network connectivity check during onboarding now verifies private endpoint configuration if you specify a private link scope. You can run the same check anytime by running [azcmagent check](azcmagent-check.md) with the new `--use-private-link` parameter.
- You can now disable the extension manager with the [local agent security controls](security-extensions.md#local-agent-security-controls).

### Fixed

- If you attempt to run `azcmagent connect` on a server already connected to Azure, the resource ID is shown on the console to help you locate the resource in Azure.
- Extended the `azcmagent connect` timeout to 10 minutes.
- `azcmagent show` no longer prints the private link scope ID. You can check if the server is associated with an Azure Arc private link scope by reviewing the machine details in the [Azure portal](https://portal.azure.com/#blade/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/servers), [CLI](/cli/azure/connectedmachine?view=azure-cli-latest#az-connectedmachine-show&preserve-view=true), or [PowerShell](/powershell/module/az.connectedmachine/get-azconnectedmachine).
- `azcmagent logs` collects only the two most recent logs for each service to reduce ZIP file size.
- `azcmagent logs` collects Guest Configuration logs again.

## Version 1.16 - March 2022

Download for [Windows](https://download.microsoft.com/download/e/a/4/ea4ea4a9-a947-4c94-995c-52eaf200f651/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

- `azcmagent logs` doesn't collect Guest Configuration logs in this release. You can locate the log directories in the [agent installation details](agent-overview.md#agent-resources).

### New features

- You can now granularly control allowed and blocked extensions on your server and disable the Guest Configuration agent. See [local agent controls to enable or disable capabilities](security-extensions.md#local-agent-security-controls) for more information.

### Fixed

- The "Arc" proxy bypass keyword no longer includes Azure Active Directory endpoints on Linux
- The "Arc" proxy bypass keyword now includes Azure Storage endpoints for extension downloads

## Version 1.15 - February 2022

Download for [Windows](https://download.microsoft.com/download/0/7/4/074a7a9e-1d86-4588-8297-b4e587ea0307/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

- The "Arc" proxy bypass feature on Linux includes some endpoints that belong to Azure Active Directory. As a result, if you only specify the "Arc" bypass rule, traffic destined for Azure Active Directory endpoints will not use the proxy server as expected.

### New features

- Network check improvements during onboarding:
  - Added TLS 1.2 check
  - Onboarding aborts when required networking endpoints are inaccessible
  - New `--skip-network-check` flag to override the new network check behavior
  - On-demand network check now available using `azcmagent check`
- [Proxy bypass](manage-agent.md#proxy-bypass-for-private-endpoints) is now available for customers using private endpoints. This feature allows you to send Azure Active Directory and Azure Resource Manager traffic through a proxy server, but skip the proxy server for traffic that should stay on the local network to reach private endpoints.
- Oracle Linux 8 is now supported

### Fixed

- Improved reliability when disconnecting the agent from Azure
- Improved reliability when installing and uninstalling the agent on Active Directory Domain Controllers
- Extended the device login timeout to 5 minutes
- Removed resource constraints for Azure Monitor Agent to support high throughput scenarios

## Version 1.14 - January 2022

Download for [Windows](https://download.microsoft.com/download/e/8/1/e816ff18-251b-4160-b421-a4f8ab9c2bfe/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Fixed a state corruption issue in the extension manager that could cause extension operations to get stuck in transient states. Customers running agent version 1.13 are encouraged to upgrade to version 1.14 as soon as possible. If you continue to have issues with extensions after upgrading the agent, [submit a support ticket](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## Version 1.13 - November 2021

Download for [Windows](https://download.microsoft.com/download/8/a/9/8a963958-c446-4898-b635-58f3be58f251/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Known issues

- Extensions might get stuck in transient states (creating, deleting, updating) on Windows machines running the 1.13 agent in certain conditions. Microsoft recommends upgrading to agent version 1.14 as soon as possible to resolve this issue.

### Fixed

- Improved reliability when installing or upgrading the agent.

### New features

- Local configuration of agent settings now available using the [azcmagent config command](azcmagent-config.md).
- Support for configuring proxy server settings [using agent-specific settings](manage-agent.md#update-or-remove-proxy-settings) instead of environment variables.
- Extension operations execute faster using a new notification pipeline. You might need to adjust your firewall or proxy server rules to allow the new network addresses for this notification service (see [networking configuration](network-requirements.md)). The extension manager falls back to the existing behavior of checking every 5 minutes when the notification service is inaccessible.
- Detection of the AWS account ID, instance ID, and region information for servers running in Amazon Web Services.

## Version 1.12 - October 2021

Download for [Windows](https://download.microsoft.com/download/9/e/e/9eec9acb-53f1-4416-9e10-afdd8e5281ad/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Improved reliability when validating signatures of extension packages.
- `azcmagent_proxy remove` command on Linux now correctly removes environment variables on Red Hat Enterprise Linux and related distributions.
- `azcmagent logs` now includes the computer name and timestamp to help disambiguate log files.

## Version 1.11 - September 2021

Download for [Windows](https://download.microsoft.com/download/6/d/b/6dbf7141-0bf0-4b18-93f5-20de4018369d/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- The agent now supports on Windows systems with the [System objects: Require case insensitivity for non-Windows subsystems](/windows/security/threat-protection/security-policy-settings/system-objects-require-case-insensitivity-for-non-windows-subsystems) policy set to Disabled.
- The guest configuration policy agent automatically retries if an error occurs during service start or restart events.
- Fixed an issue that prevented guest configuration audit policies from successfully executing on Linux machines.

## Version 1.10 - August 2021

Download for [Windows](https://download.microsoft.com/download/1/c/4/1c4a0bde-0b6c-4c52-bdaf-04851c567f43/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- The guest configuration policy agent can now configure and remediate system settings. Existing policy assignments continue to be audit-only. Learn more about the Azure Policy [guest configuration remediation options](/azure/governance/machine-configuration/remediation-options).
- The guest configuration policy agent now restarts every 48 hours instead of every 6 hours.

## Version 1.9 - July 2021

Download for [Windows](https://download.microsoft.com/download/5/1/d/51d4340b-c927-4fc9-a0da-0bb8556338d0/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

Added support for the Indonesian language

### Fixed

Fixed a bug that prevented extension management in the West US 3 region

## Version 1.8 - July 2021

Download for [Windows](https://download.microsoft.com/download/1/7/5/1758f4ea-3114-4a20-9113-6bc5fff1c3e8/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Improved reliability when installing the Azure Monitor Agent extension on Red Hat and CentOS systems
- Added agent-side enforcement of max resource name length (54 characters)
- Guest Configuration policy improvements:
  - Added support for PowerShell-based Guest Configuration policies on Linux operating systems
  - Added support for multiple assignments of the same Guest Configuration policy on the same server
  - Upgraded PowerShell Core to version 7.1 on Windows operating systems

### Fixed

- The agent continues running if it is unable to write service start/stop events to the Windows Application event log

## Version 1.7 - June 2021

Download for [Windows](https://download.microsoft.com/download/6/1/c/61c69f31-8e22-4298-ac9d-47cd2090c81d/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Improved reliability during onboarding:
  - Improved retry logic when HIMDS is unavailable
  - Onboarding continues instead of aborting if OS information isn't available
- Improved reliability when installing the Log Analytics agent for Linux extension on Red Hat and CentOS systems

## Version 1.6 - May 2021

Download for [Windows](https://download.microsoft.com/download/d/3/d/d3df034a-d231-4ca6-9199-dbaa139b1eaf/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Added support for SUSE Enterprise Linux 12
- Updated Guest Configuration agent to version 1.26.12.0 to include:
  - Policies execute in a separate process.
  - Added V2 signature support for extension validation.
  - Minor update to data logging.

## Version 1.5 - April 2021

Download for [Windows](https://download.microsoft.com/download/1/d/4/1d44ef2e-dcc9-42e4-b76c-2da6a6e852af/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Added support for Red Hat Enterprise Linux 8 and CentOS Linux 8.
- New `-useStderr` parameter to direct error and verbose output to stderr.
- New `-json` parameter to direct output results in JSON format (when used with -useStderr).
- Collect other instance metadata - Manufacturer, model, and cluster resource ID (for Azure Stack HCI nodes).

## Version 1.4 - March 2021

Download for [Windows](https://download.microsoft.com/download/e/b/1/eb128465-8830-47b0-b89e-051eefd33f7c/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

- Added support for private endpoints, which is currently in limited preview.
- Expanded list of exit codes for azcmagent.
- You can pass agent configuration parameters from a file with the `--config` parameter.
- Automatically detects the presence of Microsoft SQL Server on the server

### Fixed

Network endpoint checks are now faster.

## Version 1.3 - December 2020

Download for [Windows](https://download.microsoft.com/download/5/4/c/54c2afd8-e559-41ab-8aa2-cc39bc13156b/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### New features

Added support for Windows Server 2008 R2 SP1.

### Fixed

Resolved issue preventing the Custom Script Extension on Linux from installing successfully.

## Version 1.2 - November 2020

Download for [Windows](https://download.microsoft.com/download/4/c/2/4c287d81-6657-4cd8-9254-881ae6a2d1f4/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

Resolved issue where proxy configuration resets after upgrade on RPM-based distributions.

## Version 1.1 - October 2020

### Fixed

- Fixed proxy script to handle alternate GC daemon unit file location.
- GuestConfig agent reliability changes.
- GuestConfig agent support for US Gov Virginia region.
- GuestConfig agent extension report messages to be more verbose if there is a failure.

## Version 1.0 - September 2020

This version is the first generally available release of the Azure Connected Machine Agent.

### Plan for change

- Support for preview agents (all versions older than 1.0) will be removed in a future service update.
- Removed support for fallback endpoint `.azure-automation.net`. If you have a proxy, you need to allow the endpoint `*.his.arc.azure.com`.
- VM extensions can't be installed or modified from Azure Arc if the agent detects it's running in an Azure VM. This is to avoid conflicting extension operations being performed from the virtual machine's **Microsoft.Compute** and **Microsoft.HybridCompute** resource. Use the **Microsoft.Compute** resource for the machine for all extension operations.
- Name of guest configuration process has changed, from *gcd* to *gcad* on Linux, and *gcservice* to *gcarcservice* on Windows.

### New features

- Added `azcmagent logs` option to collect information for support.
- Added `azcmagent license` option to display EULA.
- Added `azcmagent show --json` option to output agent state in easily parseable format.
- Added flag in `azcmagent show` output to indicate if server is on a virtual machine hosted in Azure.
- Added `azcmagent disconnect --force-local-only` option to allow reset of local agent state when Azure service cannot be reached.
- Added `azcmagent connect --cloud` option to support other clouds. In this release, only Azure is supported by service at time of agent release.
- Agent has been localized into Azure-supported languages.

### Fixed

- Improvements to connectivity check.
- Corrected issue with proxy server settings being lost when upgrading agent on Linux.
- Resolved issues when attempting to install agent on server running Windows Server 2012 R2.
- Improvements to extension installation reliability

## Next steps

- Before evaluating or enabling Arc-enabled servers across multiple hybrid machines, review [Connected Machine agent overview](agent-overview.md) to understand requirements, technical details about the agent, and deployment methods.

- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
