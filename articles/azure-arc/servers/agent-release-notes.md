---
title: What's new with Azure Connected Machine agent
description: This article has release notes for Azure Connected Machine agent. For many of the summarized issues, there are links to more details.
ms.topic: overview
ms.date: 02/12/2025
ms.custom: references_regions
---

# What's new with Azure Connected Machine agent

> [!WARNING]
> Effective January 2026, the Azure Connected Machine agent will no longer accept certificates with negative serial numbers, in compliance with RFC 5280 Section 4.1.2.2, which states that "the serial number MUST be a positive integer assigned by the CA to each certificate."
> 

The Azure Connected Machine agent receives improvements on an ongoing basis. To stay up to date with the most recent developments, this article provides you with information about:

- The latest releases
- Known issues
- Bug fixes

This page is updated monthly, so revisit it regularly. If you're looking for items older than six months, you can find them in [archive for What's new with Azure Connected Machine agent](agent-release-notes-archive.md).

> [!WARNING]
> Only Connected Machine agent versions within the last 1 year are officially supported by the product group. Customers should update to an agent version within this window. Microsoft recommends staying up to date with the latest agent version whenever possible.
> 

## Version 1.49 - February 2025

Download for [Windows](https://aka.ms/AzureConnectedMachineAgent) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

### Fixed

- Added retry logic for reading the status file before sending a report if the agent fails to deserialize it the first time.
- Suppressed terminal error due to a certificate having a negative serial number. This error will be re-enabled in January 2026; customers should update their certificates before then, especially if using SSL inspection.

### New features and enhancements

- Increased package size limit for AMA only.
- Preserved `HandlerManifest.json` file during deletion to prevent extension removal failures.
- Added detection for PostgreSQL and MySQL.
- Compressed archived logs.
- Display certificate chain information for failed requests (if the TLS handshake reaches the cert stage).
- Display absolute path for log zip files to improve visibility.
- Updated recommended actions for failures to reach Service endpoints.
- Windows only 
    - The agent now saves MSI certificates both on disk and in the Windows cert store (only for Windows Servers 2019 (10.0.17763) and newer).
    - Added authentication option to use certificates from the Windows cert store for `azcmagent connect` and `azcmagent disconnect`.

## Version 1.48 - January 2025

Download for [Windows](https://download.microsoft.com/download/2/8/6/2867a351-6546-4af8-b97f-cc4483ef4192/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

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
- Introduced a scheduled task that checks for agent updates on a daily basis. Currently, the update mechanism is inactive and no changes are made to your server even if a newer agent version is available. In the future, you'll be able to schedule updates of the Azure Connected Machine agent from Azure. For more information, see [Automatic agent upgrades](manage-agent.md#automatic-agent-upgrades).


## Version 1.47 - October 2024

Download for [Windows](https://download.microsoft.com/download/2/1/d/21dfb0f5-ed95-46d5-8146-ece13381056a/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

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

Download for [Windows](https://download.microsoft.com/download/6/c/0/6c0775bc-9ed7-49af-9637-79653f783062/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

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

Download for [Windows](https://download.microsoft.com/download/0/6/1/061e3c68-5603-4c0e-bb78-2e3fd10fef30/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

### Fixed

- Fixed an issue where EnableEnd telemetry would sometimes be sent too soon.
- Added sending a failed timed-out EnableEnd telemetry log if extension takes longer than the allowed time to complete.

### New features

- Azure Arc proxy now supports HTTP traffic.
- New proxy.bypass value 'AMA' added to support AMA VM extension proxy bypass.

## Version 1.44 - July 2024

Download for [Windows](https://download.microsoft.com/download/d/a/f/daf3cc3e-043a-430a-abae-97142323d4d7/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

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
> 

## Version 1.43 - June 2024 

Download for [Windows](https://download.microsoft.com/download/0/7/8/078f3bb7-6a42-41f7-b9d3-9a0eb4c94df8/AzureConnectedMachineAgent.msi) or [Linux](manage-agent.md#installing-a-specific-version-of-the-agent)

### Fixed

- Fix for OpenSSL Vulnerability for Linux (Upgrading OpenSSL version from 3.0.13 to 3.014)
- Added Server Name Indicator (SNI) to our service calls, fixing Proxy and Firewall scenarios
- Skipped lockdown policy on the downloads directory under Guest Configuration

## Next steps

- Before evaluating or enabling Azure Arc-enabled servers across multiple hybrid machines, review [Connected Machine agent overview](agent-overview.md) to understand requirements, technical details about the agent, and deployment methods.
- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
