---
title: How to modernize server management from Configuration Manager to Azure Arc
description: Learn how to modernize server management from Configuration Manager to Azure Arc.
ms.date: 02/18/2025
ms.topic: how-to
---

# Modernize server management from Configuration Manager to Azure Arc


# Modernize Server Management with Azure Arc

Azure Arc delivers a modern server management experience for Microsoft Endpoint Configuration Manager (MECM) and Systems Center Configuration Manager (SCCM) customers. Analogous to Microsoft Intune for client endpoints, Azure delivers the next iteration of Microsoft’s server management capabilities. This documentation outlines the mapping between Configuration Manager functionalities and Azure Arc-enabled servers, a condensed roadmap of upcoming Azure management capabilities, advantages of Azure Arc-based management, and guidance on modernization.

## Functionality Mapping

Core Functionality | Azure Management Experience
--- | ---
**OS Patching** | Azure Update Manager provides a centralized solution for update assessment and management across Windows and Linux:<ul><li>Automated patch assessment and deployment</li><li>Scheduling capabilities for update management</li><li>Compliance reporting and monitoring</li></ul>Azure Update Manager supports Azure VMs and servers running on-premises and other public clouds via a VM extension for Azure Arc-enabled servers.
**Configuration** | Azure Machine Configuration (formerly Azure Policy Guest Configuration) aligns with SCCM's desired state configuration features, enabling:<ul><li>Definition and enforcement of configuration policies for application and operating system settings</li><li>Continuous assessment of machine compliance with out-of-box reporting on compliance status</li><li>Automated remediation of non-compliant settings</li></ul>Customers can author customized Desired State Configuration for custom policies and configurations. Machine Configuration is available natively for Azure Arc-enabled servers and as a VM extension for Azure VMs.
**Reporting** | Azure Change Tracking and Inventory delivers unified reporting of software, registries, applications, and daemons across Azure VMs and Azure Arc-enabled servers, including a history of their changes and out-of-box logging.<br><br>Azure Resource Graph can be used for custom queries and reporting across a fleet of Azure Arc-enabled servers. The Azure portal offers at-scale and granular visualizations for Azure VMs and Azure Arc-enabled servers.
**Scripting** | Run Command allows administrators to remotely and securely execute scripts for various server management tasks, including application management, security enforcement, and diagnostics:<ul><li>Centralized script management (creation, update, deletion, sequencing, and listing operations for scripts)</li><li>Task automation for installing software, configuring firewall rules, running health checks, and troubleshooting issues</li></ul>Run Command is available for both Azure Arc-enabled servers and Azure VMs.
**Software Distribution** | Virtual Machine Apps (VM Apps) allows administrators to safely package and distribute software to their Azure VMs. Customers can upload VM Application images to their Azure Compute Gallery and specify the target scope of Azure VMs. Azure Arc-enabled servers do not currently support VM Apps. Customers can use Run Command scripts and customized Machine Configuration for software distribution.

## Condensed Roadmap

The Azure Arc and Azure Update Manager products are actively investing in closing core capability gaps with SCCM.

* **Software Distribution:** Azure Arc-enabled servers will introduce support for Virtual Machine apps, enabling point-and-click software distribution on non-Azure infrastructure.
* **App Patching:** Azure Update Manager will introduce support for third-party application patching across Azure VMs and Azure Arc-enabled servers.
* **Distribution Points:** Local caching options will provide an alternative to direct download from Azure for key content distribution scenarios (apps, patches, extensions, and updates).
* **Unified Operations:** At-scale onboarding, consumption experiences, and pricing across key Azure management services for modernization from SCCM.

## Azure Arc Advantages

Azure Arc offers key modernization advantages across pricing, Linux support, and Copilot and Migrate integrations.

* **Pricing:** Azure Arc-enabled server customers with Windows Server Software Assurance, Windows Server Pay-as-you-Go licensing, or those running workloads on Azure VMs benefit from these management services at no additional cost (excluding associated log data ingestion). Other servers connected to Azure Arc use pay-as-you-go pricing plans.
* **Linux Support:** Unlike SCCM and MECM, Azure server management capabilities natively support Linux, including Red Hat Enterprise Linux, Ubuntu, Oracle Linux, Debian, and more, enabling consistent management across Windows and Linux server estates.
* **Azure Copilot:** Integrated natively across key management and operational scenarios, Azure Copilot delivers an AI-enhanced server management experience for querying and remediating across Azure VMs and Azure Arc-enabled servers.
* **Migration:** Azure Arc offers assessment capabilities that ease customer migration and modernization to Azure. Customers can use the same Azure management capabilities across Azure Arc and Azure VMs, reducing operational overhead.

## Modernization Guidance

To help define their SCCM modernization strategy, we outline key guidance and recommendations.

* **Onboarding:** Connect servers to Azure Arc using a Scheduled Task with Configuration Manager. This installs the Azure Connected Machine agent and establishes connectivity to Azure. Customers with active Windows Server Software Assurance should attest to their coverage.
* **Gradual Management:** Deploy Azure Arc in phases to evaluate specific capabilities over time. Popular evaluation services include Microsoft Defender for Servers, Microsoft Sentinel, and Azure Monitor for security and monitoring.
* **Joint Management:** Manage servers using both Configuration Manager and Azure Arc-enabled servers simultaneously. Customers often separate tasks across solutions (e.g., SCCM for software distribution, Azure Update Manager for OS patching, and Azure Machine Configuration for OS configuration).
* **SCVMM:** For organizations using System Center Virtual Machine Manager (SCVMM), Azure Arc offers integration capabilities with Azure Arc-enabled SCVMM, facilitating VM lifecycle management and connection of SCVMM environments through Azure Arc.
* **SCOM:** For organizations using Systems Center Operations Manager (SCOM), Azure Arc and Azure VMs support Azure Monitor, enabling logging, alerting, and visualization of application and infrastructure performance. Azure Monitor includes robust reporting with Azure Workbooks and Azure Dashboards. Customers can use SCOM Managed Instance (SCOM MI).

Ultimately, Azure Arc represents a significant evolution in server management, offering a cloud-native approach to hybrid and multi-cloud environments. While it may not fully replicate all SCCM functionalities, its advantages in pricing flexibility, Linux support, and AI integrations make it a compelling option for organizations looking to modernize their infrastructure management. As Azure Arc continues to evolve, it is poised to become an increasingly powerful tool for unified server management across diverse environments.



