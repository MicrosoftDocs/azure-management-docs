---
title: What's new with Azure Connected Machine agent
description: This article has release notes for Azure Connected Machine agent. For many of the summarized issues, there are links to more details.
ms.topic: overview
ms.date: 03/24/2026
ms.custom: references_regions
# Customer intent: As a system administrator, I want to access the release notes for the Azure Connected Machine agent, so that I can stay informed about updates, fixes, and new features to ensure optimal performance and compliance of my cloud-based infrastructure.
---

# What's new with Azure Connected Machine agent

> [!WARNING]
> Only Connected Machine agent versions released within the last year are officially supported by the product group. All customers should update to an agent version within this window or [enable automatic agent upgrades (preview)](manage-agent.md#enable-automatic-agent-upgrade-preview). Microsoft recommends staying up to date with the latest agent version whenever possible.

The Azure Connected Machine agent receives improvements on an ongoing basis. To stay up to date with the most recent developments, this article provides you with information about:

- The latest releases
- Known issues
- Bug fixes

This page is updated monthly, so revisit it regularly. If you're looking for items older than six months, you can find them in [archive for What's new with Azure Connected Machine agent](agent-release-notes-archive.md).

> [!WARNING]
> Effective February 2027, the Azure Connected Machine agent will no longer accept certificates with negative serial numbers, in compliance with RFC 5280 Section 4.1.2.2, which states that "the serial number MUST be a positive integer assigned by the CA to each certificate."

## Version 1.66 - July 2026
Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.66/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

|Feature|Windows|Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config** | **1.29.114.0** | **1.26.115.0** ||
|Updated OpenSSL from 3.6.2 to 3.6.3.|✓|✓|Improvement|
|Improved reliability of extension state file writes when the agent is interrupted while saving them.|✓|✓|Improvement|
|Improved the error message shown when a pre-existing extension directory can't be removed prior to installation.|✓|✓|Improvement|
|Reduced verbose agent logging.|✓|✓|Improvement|
|Partially extracted extension package folders are now cleaned up when extraction fails.|✓|✓|Bug Fix|
|Fixed a failure loop following an extension upgrade by re-extracting the package when its handler manifest is missing.|✓|✓|Bug Fix|
|Added retries when removing an extension folder that is temporarily locked by another process.|✓|✓|Improvement|
|Fixed Run Command cleanup so a failed package download can be removed and the command reinstalled.|✓|✓|Bug Fix|
|Fixed TLS certificate validation failures that could occur when a required root certificate was not already present in the local certificate store.|✓||Bug Fix|
|Fixed a memory spike in the Machine Configuration agent that occurred on every refresh cycle.|✓||Bug Fix|
| **Azcmagent** | **1.66** | **1.66** ||
|Save install script for downgrade to a unique path in the agent directory instead of a fixed path in TEMP.|✓||Security Fix|
|Redacted credentials embedded in proxy URLs from error messages.|✓|✓|Security Fix|
|Fixed unexpected EOF errors when streaming VM application packages.|✓|✓|Bug Fix|
|Blocked use of the repair tool when the agent is installed via the Windows MSI installer.|✓||Security Fix|
|HIMDS token endpoint now honors the `bypass_cache` query parameter so callers can request a guaranteed fresh token.|✓|✓|Bug Fix|
|Arc proxy now omits entries in the gateway bypass list.|✓|✓|Bug Fix|
|Fixed ESU license validation by forwarding the full intermediate certificate chain.|✓||Bug Fix|
|Updated MSAL to 1.7 to fix a localhost loopback redirect issue during interactive authentication.|✓||Bug Fix|


## Version 1.65 - June 2026

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.65/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

|Feature|Windows |Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**   |**1.29.111.0**|**1.26.112.0**||
|Strengthened TLS certificate validation to address CVE-2026-47632.|✓|✓|Security Fix|
|Fixed a crash that could occur while checking extensions for updates.|✓|✓|Bug Fix|
|Increased extension package download/unzip size limits to 2 GB.|✓|✓|Bug Fix|
|Fixed a crash related to a heap memory corruption error that could occur on service shutdown.||✓|Bug Fix|
| **Azcmagent**|**1.65**|**1.65**||
|Added backup file for localconfig.json.|✓|✓|Feature|
|Added warnings to help page for azcmagent disconnect and azcmagent remove extension commands.|✓|✓|Improvement|
|Increased VM application download timeout to 30 minutes.|✓|✓|Bug Fix|
|Added quotes to arguments in auto-upgrade script.||✓|Bug Fix|
|Fixed Linux install script to not recreate repository if package already exists.||✓|Bug Fix|
|Fixed Linux install script to use proxy correctly.||✓|Bug Fix|

## Version 1.64 - May 2026 

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.64/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

|Feature|Windows |Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**   |**1.29.109.0**|**1.26.110.0**||
|Updated OpenSSL from 3.6.1 to 3.6.2.|✓|✓|Improvement|
|Updated bundled PowerShell version from 7.4.14 to 7.4.15.|✓||Improvement|
|Fixed security baseline customization report failing with invalid JSON due to long configuration parameter values.|✓|✓|Bug Fix|
|Fixed compliance reporting for unknown Linux distributions in security baseline assignments to correctly report as non-compliant.||✓|Bug Fix|
|Reduced network bandwidth for policy assignment requests.|✓|✓|Improvement|
|**Azcmagent**|**1.64**|**1.64**||
|Added Arc Gateway bypass list support so configured FQDNs skip the gateway and use the customer's enterprise proxy (or direct connection) instead.|✓|✓|Feature|
|Added Ubuntu Pro subscription status to detected properties.||✓|Feature|
|Windows install script now extracts intermediate certificates from the MSI Authenticode signature to avoid validation failures when intermediates aren't cached.|✓||Improvement|
|HIMDS now refreshes its regional endpoint and retries the heartbeat when the service returns a 421 response.|✓|✓|Improvement|
|Fixed an issue where agentconfig.json was unnecessarily read before onboarding, and added retry logic when saving the agent certificate to the cert store.|✓||Bug Fix|
|Fixed agent version stamping when binaries are replaced during auto-upgrade and removed false positives from the upgrade launcher script.|✓||Bug Fix|
|Updated Configuration UI to paginate Resource Graph subscription queries so all inherited subscriptions are returned.|✓||Improvement|
|Added ESU eligibility to azcmagent show output.|✓|✓|Feature|
|Reverted custom cipher-suite enforcement.|✓||Improvement|

## Version 1.63 - April 2026

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.63/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

| Feature| Windows |Linux|Change Type|
| -------- | -------- | -------- | -------- |
|**Guest Config**   |**1.29.107.0**|**1.26.108.0**||
|Fixed extension package signing validation to match the expected catalog or signature file by name, preventing validation failures when multiple signing files are present.|✓|✓|Bug Fix|
|Fixed status file parsing errors during Run Command extension install recovery.|✓|✓|Bug Fix|
|Added early failure with a clear error message when the extension install path has the noexec mount flag set.||✓|Improvement|
|Improved HIMDS token path handling for environments with symlinked directories.||✓|Improvement|
|Stopped unnecessary error messages from heartbeat scripts appearing in /var/log/messages.||✓|Improvement|
|Updated bundled PowerShell version from 7.4.13 to 7.4.14.|✓|✓|Improvement|
|Improved compliance reporting for security baseline policy assignments.|✓|✓|Improvement|
|Addressed CVE-2026-2673 ||✓|Bug Fix|
|**Azcmagent**|**1.63**|**1.63**||
|Added TLS cipher suite validation to azcmagent check command.|✓|✓|Feature|
|Arc proxy now uses SSL endpoint for communication with HIMDS.|✓|✓|Feature|
|Added BIOS serial number to detected properties.|✓||Feature|
|Added retry logic for heartbeat and cloud config retrieval on 5xx server errors.|✓|✓|Improvement|
|Increased timeout for identity requests.|✓|✓|Improvement|

## Version 1.62 - March 2026

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.62/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

| Feature| Windows|Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**|**1.29.106.0**|**1.26.105.0**||
|Fixed extension installation behavior post cleanup. |✓|✓|Bug Fix|
|**Azcmagent**|**1.62**|**1.62**||
|Support for SLES 16 (x86_64), Alma 10 (x86_64 & Arm64), and Rocky 10 (x86_64 and Arm64). ||✓|Feature|
|Added automatic backup and restore functionality for agentconfig.json to improve reliability.|✓|✓|Feature|
|New `--enable-automatic-upgrade` flag for `azcmagent connect` to enable auto-upgrade during onboarding.|✓|✓|Feature|
|Added `--use-aws-ec2-hostname` flag to use hostname instead of instance ID for AWS EC2 resource names.|✓|✓|Feature|
|Fixed configuration file updates to only write when there are actual changes, reducing unnecessary I/O.|✓|✓|Bug Fix|
|Fixed relay URL, which is used in SSH and WAC scenarios.|✓|✓|Bug Fix|
|Fixed IPv6 detection when setting up SSL endpoint.|✓|✓|Bug Fix|
|Added `azcmagent upgrade` CLI command for upgrading the Azure Connected Machine agent|✓|✓|Feature|
|New `--identity-key-store` flag for `azcmagent connect` to enable TPM-backed Identity. Reserved for future use, see [aka.ms/arc-tpm-backed-identity/preview](https://aka.ms/arc-tpm-backed-identity/preview) to participate in preview.|✓|✓|Feature|
|Added `azcmagent check tpm` to verify TPM-backed Identity readiness. Reserved for future use, see [aka.ms/arc-tpm-backed-identity/preview](https://aka.ms/arc-tpm-backed-identity/preview) to participate in preview.|✓|✓|Feature|

## Version 1.61 - February 2026

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.61/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

|Feature|Windows| Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**  |**1.29.105.0**| **1.26.105.0**   ||
|Fixed support for security baseline customization on localized operating systems.|✓|✓|Bug Fix|
|Resolved an issue where upgrading the Run Command extension caused previously executed commands to re-run.|✓|✓|Bug Fix|
|Fixed a bug that caused extension enable operations to time out prematurely.|✓|✓|Bug Fix|
|Improved reliability of GPG signature validation of extensions on Linux distributions.||✓|Improvement|
|Update OpenSSL to 3.6.1 for improved security and performance.|✓|✓|Improvement|
|**Azcmagent**|**1.61.03310.2719**|**1.61.03310848**||
|Added support for ARM64 Oracle 8, x86_64 Oracle 10, and Debian 13.||✓|Feature|
|Auto upgrade script now respects proxy configuration on Linux.||✓|Improvement|
|Added option to disable automatic upgrades locally.|✓|✓|Feature|
|Added MSI signature verification to the Windows installation script for enhanced security.|✓||Improvement|
|Fixed a bug in the Linux install script where the wrong package manager was invoked on some distributions.||✓|Bug Fix|
|Fix bug where serial number isn't detected by `azcmagent connect` command, causing certificate-based authentication to fail.|✓||Bug Fix|
|Fix bug causing installation to fail on machines with .NET < 4.5.1.|✓||Bug Fix|

### Known issues

On Windows, if a user downgrades the Azure Arc agent from version 1.61 to any earlier version, the agent might become disconnected.

To restore connectivity, a change must be made to the agent configuration file. Please use one of the following methods to edit permissions on the agentconfig.json:

Run the following command from an elevated Command Prompt:

```
attrib -r "C:\ProgramData\AzureConnectedMachineAgent\Config\agentconfig.json"
```

Or run the following command from an elevated PowerShell session:

```powershell
Set-ItemProperty -Path 
"C:\ProgramData\AzureConnectedMachineAgent\Config\agentconfig.json" -Name IsReadOnly -Value $false
```

After making this change, the agent should reconnect automatically within 5 minutes.

## Version 1.60 - January 2026

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.60/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

| Feature|Windows|Linux| Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config** |**1.29.103.0**|**1.26.103.0**||
|Address CVE 2026-21224|✓|✓|Security Fix  |
|Fixed bugs that cause Machine Configuration agent and GC worker to crash|✓|✓|Bug Fix|
|Added additional parameter validation to ExtensionCleanup.ps1|✓||Improvement|
|Enhanced reliability for compliance evaluation for ApplyAndAutoCorrect Machine Configuration policy assignments|✓|✓|Improvement|
|**Azcmagent**|**1.60.03293.2680**|**1.60.03293.809**||
|Address CVE 2026-21224|✓|✓|Security Fix|

## Version 1.59 - December 2025

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.59/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

|Feature |Windows |Linux|Change Type|
| -------- | -------- | -------- | -------- |
|**Guest Config**   |**1.29.102.0** |**1.26.102.0**||
|Increased unzipped extension package size limit from 400 MB to 1GB |✓|✓|Improvement|
|Changed behavior on extension install when extension files already exist. Now, install will continue after attempting cleanup instead of failing.|✓|✓|Improvement|
|Updated bundled PowerShell version from 7.4.7 to 7.4.13.|✓|✓|Improvement|
|Updated Azure Storage API version from 2019-02-02 to 2025-11-05.|✓|✓|Improvement|
|**Azcmagent**|**1.59**|**1.59**||
|Improved error handling for regional identity service network check.|✓|✓|Bug Fix|
|Added new authentication method for azcmagent to use Azure CLI credentials for azcmagent connect and disconnect.|✓|✓|Improvement|
|Fixed crashes caused by concurrent access of the agent's local config.|✓|✓|Bug Fix|
|Updated Configuration UI to talk to Azure Resource Graph API to allow access on subscriptions with inherited permissions.|✓||Improvement|

## Version 1.58 - November 2025

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.58/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

| Feature| Windows|Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config** 	|**1.29.101.0**|**1.26.101.0**||
|Updated OpenSSL library to version 3.6.0 (Windows) and 3.4.3 (Linux) for enhanced security and performance.|✓|✓|Improvement|
|Use "systemctl daemon-reload" instead of "systemctl daemon-reexec" for better stability and compatibility.||✓|Improvement|
|Added customization support for Linux CIS Baseline and ASB security policies.|✓|✓|Improvement|
|**Azcmagent**|**1.58.03224.2567**|**1.58.03224.693**||
|Enhanced log security by escaping newline characters to prevent log injection attacks.|✓|✓|Bug Fix|
|Removed Preview flag from connection.type configuration property as the Arc gateway feature has been promoted to General Availability.|✓|✓|Bug Fix|
|Fixed metadata synchronization: machine FQDN.|✓|✓|Bug Fix|
|Fixed Windows installer service configuration issues when launched via double-click instead of elevated execution.|✓||Bug Fix|
|Fixed security validation to recognize Built-in Administrator account during named pipe ownership verification.|✓||Bug Fix|

## Version 1.57 - October 2025

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.57/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

| Feature | Windows |Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**   | **1.29.100.0**   |||
| Fixed GC worker crashes on Windows Server 2012 Standard machines. | ✓ ||Bug Fix|
|**Azcmagent**|**1.57.03197.2516**|**1.57.03197.640**||
|Connectivity check now marks regional GAS endpoint required|✓ |✓ |Feature|
|Fixed duplicate heartbeat requests that were causing HTTP 429 (Too Many Requests) responses from HIS.|✓|✓|Bug Fix|
|Fixed MSI installer incorrectly removing Arc services during installation.|✓||Bug Fix|
|Fixed RPM installer to install GC and EXT services before starting HIMDS.||✓|Bug Fix|

### Known Issues

If the Windows installer is launched by double-clicking (followed by the UAC prompt), it might fail to configure the Arc services properly. To ensure successful installation, please use one of the following methods:

- **Right-click** the installer and select **Run as administrator**, or

- Execute the installer using `msiexec` from an **elevated PowerShell or Command Prompt**.

> [!NOTE]
> This article contains updates covering the past six months. For earlier releases, see [Archive for What's new with Azure Connected Machine agent](agent-release-notes-archive.md)

## Next steps

- Before evaluating or enabling Azure Arc-enabled servers across multiple hybrid machines, read [Connected Machine agent overview](agent-overview.md) to understand requirements, technical details about the agent, and deployment methods.
- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
