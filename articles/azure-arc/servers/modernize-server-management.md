---
title: How to modernize server management from Configuration Manager to Azure Arc
description: Learn how to modernize server management from Configuration Manager to Azure Arc.
ms.date: 02/18/2025
ms.topic: how-to
# Customer intent: "As a server administrator using Configuration Manager, I want to transition management to Azure Arc, so that I can leverage modern capabilities for OS patching, compliance monitoring, and unified reporting across both Windows and Linux environments."
---

# Modernize server management from Configuration Manager to Azure Arc

Azure Arc delivers a modern server management experience for [Microsoft Endpoint Configuration Manager (MECM)](/mem/configmgr/) and [Systems Center Configuration Manager (SCCM)](/mem/configmgr/) customers. Analogous to Microsoft Intune for client endpoints, Azure delivers the next iteration of Microsoft’s server management capabilities. This article provides a functional comparison of Configuration Manager and Azure Arc, a roadmap of upcoming Azure features, the benefits of Azure Arc, and modernization guidance.

## Functionality Mapping

Core Functionality | Azure Management Experience
--- | ---
**OS Patching** | [Azure Update Manager](/azure/update-manager/) provides a centralized solution for update assessment and management across Windows and Linux:<ul><li>Automated patch assessment and deployment</li><li>Scheduling capabilities for update management</li><li>Compliance reporting and monitoring</li></ul>Azure Update Manager supports Azure VMs and servers running on-premises and other public clouds via a VM extension for Azure Arc-enabled servers.
**Configuration** | [Azure Machine Configuration](/azure/governance/machine-configuration/) (formerly Azure Policy Guest Configuration) aligns with SCCM's desired state configuration features, enabling:<ul><li>Definition and enforcement of configuration policies for application and operating system settings</li><li>Continuous assessment of machine compliance with out-of-box reporting on compliance status</li><li>Automated remediation of noncompliant settings</li></ul>You can author customized Desired State Configuration for custom policies and configurations. Machine Configuration is available natively for Azure Arc-enabled servers and as a VM extension for Azure VMs.
**Reporting** | [Azure Change Tracking and Inventory](/azure/automation/change-tracking/overview-monitoring-agent?tabs=win-az-vm) delivers unified reporting of software, registries, applications, and daemons across Azure VMs and Azure Arc-enabled servers, including a history of their changes and out-of-box logging.<br><br>[Azure Resource Graph](/azure/governance/resource-graph/) can be used for custom queries and reporting across a fleet of Azure Arc-enabled servers. The Azure portal offers at-scale and granular visualizations for Azure VMs and Azure Arc-enabled servers.
**Scripting** | [Run Command](run-command.md) allows administrators to remotely and securely execute scripts for various server management tasks, including application management, security enforcement, and diagnostics:<ul><li>Centralized script management (creation, update, deletion, sequencing, and listing operations for scripts)</li><li>Task automation for installing software, configuring firewall rules, running health checks, and troubleshooting issues</li></ul>Run Command is available for both Azure Arc-enabled servers and Azure VMs.
**Software Distribution** | [Virtual Machine Apps](/azure/virtual-machines/vm-applications?tabs=ubuntu) (VM Apps) allows administrators to safely package and distribute software to their Azure VMs. You can upload VM Application images to their Azure Compute Gallery and specify the target scope of Azure VMs. Azure Arc-enabled servers don't currently support VM Apps. You can use Run Command scripts and customized Machine Configuration for software distribution.

## Condensed Roadmap

The Azure Arc and Azure Update Manager products are actively investing in closing core capability gaps with SCCM.

* **Software Distribution:** Azure Arc-enabled servers introduce support for Virtual Machine apps, enabling point-and-click software distribution on non-Azure infrastructure.
* **App Patching:** Azure Update Manager introduces support for third-party application patching across Azure VMs and Azure Arc-enabled servers.
* **Distribution Points:** Local caching options provide an alternative to direct download from Azure for key content distribution scenarios (apps, patches, extensions, and updates).
* **Unified Operations:** At-scale onboarding, consumption experiences, and pricing across key Azure management services for modernization from SCCM.

## Azure Arc Advantages

Azure Arc offers key modernization advantages across pricing, Linux support, and Copilot and Migrate integrations.

* **Pricing:** Azure Arc-enabled server customers with Windows Server Software Assurance, Windows Server Pay-as-you-Go licensing, or those running workloads on Azure VMs benefit from these management services at no additional cost (excluding associated log data ingestion). Other servers connected to Azure Arc use pay-as-you-go pricing plans.
* **Linux Support:** Unlike SCCM and MECM, Azure server management natively supports Linux (including Red Hat Enterprise Linux, Ubuntu, Oracle Linux, and Debian), allowing for consistent management of Windows and Linux servers.
* **Azure Copilot:** Integrated natively across key management and operational scenarios, Azure Copilot delivers an AI-enhanced server management experience for querying and remediating across Azure VMs and Azure Arc-enabled servers.
* **Migration:** Azure Arc offers assessment capabilities that ease customer migration and modernization to Azure. You can use the same Azure management capabilities across Azure Arc and Azure VMs, reducing operational overhead.

## Modernization Guidance

Following are some key guidance recommendations to help define your SCCM modernization strategy:

* **Onboarding:** Connect servers to Azure Arc using a Scheduled Task with Configuration Manager. This installs the Azure Connected Machine agent and establishes connectivity to Azure. Customers with active Windows Server Software Assurance should attest to their coverage.
* **Gradual Management:** Deploy Azure Arc in phases to evaluate specific capabilities over time. Popular evaluation services include [Microsoft Defender for Servers](/azure/defender-for-cloud/defender-for-servers-overview), [Microsoft Sentinel](/azure/sentinel/), and [Azure Monitor](/azure/azure-monitor/) for security and monitoring.
* **Joint Management:** Manage servers using both Configuration Manager and Azure Arc-enabled servers simultaneously. Customers often separate tasks across solutions (for example, SCCM for software distribution, Azure Update Manager for OS patching, and Azure Machine Configuration for OS configuration).
* **SCVMM:** For organizations using System Center Virtual Machine Manager (SCVMM), Azure Arc offers integration capabilities with Azure Arc-enabled SCVMM, facilitating VM lifecycle management and connection of SCVMM environments through Azure Arc.
* **SCOM:** For organizations using Systems Center Operations Manager (SCOM), Azure Arc and Azure VMs support Azure Monitor, enabling logging, alerting, and visualization of application and infrastructure performance. Azure Monitor includes robust reporting with Azure Workbooks and Azure Dashboards. You can use SCOM Managed Instance (SCOM MI).

Ultimately, Azure Arc represents a significant evolution in server management, offering a cloud-native approach to hybrid and multicloud environments. While it might not fully replicate all SCCM functionalities, its advantages in pricing flexibility, Linux support, and AI integrations make it a compelling option for organizations looking to modernize their infrastructure management. As Azure Arc continues to evolve, it's poised to become an increasingly powerful tool for unified server management across diverse environments.



