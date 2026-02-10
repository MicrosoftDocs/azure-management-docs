---
title: What's new with Azure Connected Machine agent
description: This article has release notes for Azure Connected Machine agent. For many of the summarized issues, there are links to more details.
ms.topic: overview
ms.date: 12/17/2025
ms.custom: references_regions
# Customer intent: As a system administrator, I want to access the release notes for the Azure Connected Machine agent, so that I can stay informed about updates, fixes, and new features to ensure optimal performance and compliance of my cloud-based infrastructure.
---

# What's new with Azure Connected Machine agent

> [!WARNING]
> Only Connected Machine agent versions released within the last year are officially supported by the product group. All customers should update to an agent version within this window or [enable automatic agent upgrades (preview)](manage-agent.md#automatic-agent-upgrade-preview). Microsoft recommends staying up to date with the latest agent version whenever possible.

The Azure Connected Machine agent receives improvements on an ongoing basis. To stay up to date with the most recent developments, this article provides you with information about:

- The latest releases
- Known issues
- Bug fixes

This page is updated monthly, so revisit it regularly. If you're looking for items older than six months, you can find them in [archive for What's new with Azure Connected Machine agent](agent-release-notes-archive.md).

> [!WARNING]
> Effective February 2027, the Azure Connected Machine agent will no longer accept certificates with negative serial numbers, in compliance with RFC 5280 Section 4.1.2.2, which states that "the serial number MUST be a positive integer assigned by the CA to each certificate."

> [!IMPORTANT]
> Starting from version 1.56 of the Connected Machine agent for Windows (excluding Windows Server 2012 and Windows Server 2012 R2), you must configure cipher suites for at least one of the recommended TLS versions. For more information, see [Windows TLS configuration issues](troubleshoot-networking.md#windows-tls-configuration-issues).
> 
## Version 1.61 - February 2026

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.60/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

|Feature|Windows| Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**  |**1.29.105.0**| **1.26.105.0**   ||
|Fixed support for security baseline customization on localized operating systems.|✓|✓|Bug Fix|
|Resolved an issue where upgrading the Run Command extension caused previously executed commands to re-run.|✓|✓|Bug Fix|
|Fixed a bug that caused extension enable operations to time out prematurely.|✓|✓|Bug Fix|
|Improved reliability of GPG signature validation of extensions on Linux distributions.||✓|Improvement|
|**Azcmagent**|**1.61.03310.2719**|**1.61.03310848**||
|Added support for ARM64 Oracle 8, x86_64 Oracle 10, and Debian 13.||✓|Feature|
|Auto upgrade script now respects proxy configuration on Linux.||✓|Improvement|
|Added option to disable automatic upgrades locally.|✓|✓|Feature|
|Added MSI signature verification to the Windows installation script for enhanced security.|✓||Improvement|
|Fixed a bug in the Linux install script where the wrong package manager was invoked on some distributions.||✓|Bug Fix|

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

If the Windows installer is launched by double-clicking (followed by the UAC prompt), it may fail to configure the Arc services properly. To ensure successful installation, please use one of the following methods:

- **Right-click** the installer and select **Run as administrator**, or

- Execute the installer using `msiexec` from an **elevated PowerShell or Command Prompt**.

## Version 1.56 - September 2025

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.56/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

| Feature| Windows |Linux|Change Type|
| -------- | -------- | -------- | -------- |
| **Guest Config**   | **1.29.99.0**   |**1.26.94.0**||
|Increased timeout when retrieving data from HIMDS on Azure Local servers.| ✓ |✓|Bug Fix|
|**Azcmagent**|**1.56.03167.2465**|**1.56.03167.593**||
|Added support for ARM64 Debian 12.||✓|New Distro Support|
|Declared bundled OpenSSL in the spec file on RPM-based OSes.||✓|Security, Bug Fix|
|Azcmagent commands requiring admin privileges now confirm the pipe owner as HIMDS during IPC.|✓||Security, Bug Fix|
|Enforces minimum-required TLS cipher-suite enablement.|✓||Security, Bug Fix|
|Removed requests concerning the Arc gateway feature for ALDO (Azure Local Disconnected Operations).|✓ |✓ |Bug Fix|
|Increased token acquisition timeout.|✓ |✓ |Bug Fix|
|HIMDS now reports 'service stop' only after cleanup tasks complete.|✓ ||Bug Fix|

## Version 1.55 - August 2025

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.55/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Improved logic to accurately detect whether the server is Azure Local.

- Arc proxy no longer requests tokens from HIMDS unless explicitly enabled.

- [Windows Only] Minor accessibility improvements to the GUI application.

## Version 1.54 - July 2025

Download for [Windows](https://gbl.his.arc.azure.com/azcmagent/1.54/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#install-a-specific-version-of-the-agent)

### Fixed

- Fixed issues related to double-free memory errors and updating policy compliance status.

- [Linux Only] Updated Boost on Linux to resolve service start issues caused by compatibility problems.

- [Linux Only] Corrected Arc proxy log file permission during upgrade.

- [Windows Only] Updated local PATH environment variable to resolve service install/delete errors.

### New features and enhancements

- Added support for managed identity-based custom policy downloads.

> [!NOTE]
> This article contains updates covering the past six months. For earlier releases, see [Archive for What's new with Azure Connected Machine agent](agent-release-notes-archive.md)

## Next steps

- Before evaluating or enabling Azure Arc-enabled servers across multiple hybrid machines, read [Connected Machine agent overview](agent-overview.md) to understand requirements, technical details about the agent, and deployment methods.
- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
