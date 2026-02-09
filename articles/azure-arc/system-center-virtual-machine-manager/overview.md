---
title: Overview of the Azure Arc-enabled System Center Virtual Machine Manager
description: This article provides a detailed overview of the Azure Arc-enabled System Center Virtual Machine Manager.
ms.date: 02/09/2026
ms.topic: overview
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
keywords: "VMM, Arc, Azure, System Center"
ms.custom:
  - references_regions
  - build-2025
# Customer intent: As a system administrator, I want to connect my System Center Virtual Machine Manager to Azure, so that I can manage VMs consistently across hybrid environments using Azure's security and governance capabilities.
---

# Overview of Azure Arc-enabled System Center Virtual Machine Manager

Azure Arc-enabled System Center Virtual Machine Manager (SCVMM) empowers System Center customers to connect their VMM environment to Azure and perform VM self-service operations from Azure portal. By extending the Azure control plane to SCVMM managed infrastructure, Azure Arc-enabled SCVMM enables you to use Azure security, governance, and management capabilities consistently across your System Center managed estate and Azure.

By using Azure Arc-enabled SCVMM, you can manage your hybrid environment consistently and perform self-service VM operations through Azure portal. For Microsoft Azure Pack customers, this solution is intended as an alternative to perform VM self-service operations.

Azure Arc-enabled SCVMM allows you to:

- Perform various VM lifecycle operations such as start, stop, pause, and delete VMs on SCVMM managed VMs directly from Azure.
- Empower developers and application teams to self-serve VM operations on demand by using [Azure role-based access control (RBAC)](/azure/role-based-access-control/overview).
- Browse your VMM resources (VMs, templates, VM networks, and storage) in Azure, providing you with a single pane view for your infrastructure across both environments.
- Discover and onboard existing SCVMM managed VMs to Azure.
- Install the Azure Connected Machine agent at scale on SCVMM VMs to [govern, protect, configure, and monitor them](../servers/overview.md#supported-cloud-operations).
- Build automation and self-service pipelines by using Python, Java, JavaScript, Go, and .NET SDKs; Terraform, ARM, and Bicep templates; Azure REST APIs, CLI, and PowerShell.
- Leverage Azure Arc benefits such as [Windows Server management](/azure/azure-arc/servers/windows-server-management-overview?tabs=portal) for VMs with Software Assurance licenses, and pay-as-you-go billing for [Extended Security Updates](/azure/azure-arc/system-center-virtual-machine-manager/deliver-esus-for-system-center-virtual-machine-manager-vms) for Windows Server and SQL Server VMs. 

For updates on the capabilities and enhancements of Azure Arc, see the [Tech Community blog](https://techcommunity.microsoft.com/category/azure/blog/azurearcblog) for Azure Arc.

## How does it work?

To Azure Arc-enable an SCVMM management server, deploy [Azure Arc resource bridge](../resource-bridge/overview.md) in the VMM environment. Azure Arc resource bridge is a virtual appliance that connects VMM management server to Azure. By using Azure Arc resource bridge, you can represent the SCVMM resources (clouds, VMs, templates, and more) in Azure and perform various operations on them.

## Architecture

The following image shows the architecture for the Azure Arc-enabled SCVMM:

:::image type="content" source="media/architecture/azure-arc-scvmm-architecture.png" alt-text="Screenshot of Arc enabled SCVMM - architecture." lightbox="media/architecture/azure-arc-scvmm-architecture.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

## How is Azure Arc-enabled SCVMM different from Azure Arc-enabled servers

- Azure Arc-enabled servers interact on the guest operating system level, with no awareness of the underlying infrastructure fabric and the virtualization platform that they're running on. Since Azure Arc-enabled servers also support bare-metal machines, a host hypervisor might not exist in some cases.
- Azure Arc-enabled SCVMM is a superset of Azure Arc-enabled servers that extends management capabilities beyond the guest operating system to the VM itself. This extension provides lifecycle management and CRUD (Create, Read, Update, and Delete) operations on an SCVMM VM. The Azure portal exposes these lifecycle management capabilities, and they look and feel just like a regular Azure VM. Azure Arc-enabled SCVMM also provides guest operating system management by using the same components as Azure Arc-enabled servers.

You can start with either option and add the other one later without any disruption. Both options provide the same consistent experience.

> [!NOTE]
> For guidance on choosing the right Azure Arc service for your virtual machines, see [Choose the right Azure Arc service for machines](../choose-service.md).

### Supported scenarios

Azure Arc-enabled SCVMM supports the following scenarios:

- SCVMM administrators can connect a VMM instance to Azure and browse the SCVMM virtual machine inventory in Azure.
- Administrators can use the Azure portal to browse SCVMM inventory and register SCVMM cloud, virtual machines, VM networks, and VM templates into Azure.
- Administrators can provide app teams/developers fine-grained permissions on those SCVMM resources through Azure RBAC.
- App teams can use Azure interfaces (portal, CLI, PowerShell, SDKs, Terraform, Bicep, ARM templates, or REST API) to manage the lifecycle of on-premises VMs they use for deploying their applications (CRUD, Start/Stop/Restart).
- Administrators can install Azure Connected Machine agent on SCVMM-managed VMs at-scale and can perform the following actions:
     - **Govern**:
         * Assign [Azure machine configurations](/azure/governance/machine-configuration/overview) to audit settings inside the machine.
     - **Protect**:
         * Protect non-Azure servers with [Microsoft Defender for Endpoint](/microsoft-365/security/defender-endpoint), included through [Microsoft Defender for Cloud](/azure/security-center/defender-for-servers-introduction), for threat detection, for vulnerability management, and to proactively monitor for potential security threats. Microsoft Defender for Cloud presents the alerts and remediation suggestions from the threats detected.
         * Use [Microsoft Sentinel](/azure/azure-arc/servers/scenario-onboard-azure-sentinel) to collect security-related events and correlate them with other data sources.
     - **Configure**:
         * Use [Azure Automation](/azure/automation/extension-based-hybrid-runbook-worker-install?tabs=windows) for frequent and time-consuming management tasks using PowerShell and Python [runbooks](/azure/automation/automation-runbook-execution). Assess configuration changes for installed software, Microsoft services, Windows registry and files, and Linux daemons using the Azure Monitor agent for [change tracking and inventory](/azure/automation/change-tracking/overview-monitoring-agent?tabs=win-az-vm).
         * Use [Azure Update Manager](/azure/update-manager/overview) to manage operating system updates for Windows and Linux servers. Automate onboarding and configuration of a set of Azure services when you use [Azure Automanage](/azure/automanage/automanage-arc).
         * Perform post-deployment configuration and automation tasks using supported [Arc-enabled servers VM extensions](/azure/azure-arc/servers/manage-vm-extensions) for non-Azure Windows or Linux machine.
     - **Monitor**:
         * Monitor operating system performance and discover application components to monitor processes and dependencies with other resources using [VM insights](/azure/azure-monitor/vm/vminsights-overview).
         * Collect other log data, such as performance data and events, from the operating system or workloads running on the machine with the [Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-overview). This data is stored in a [Log Analytics workspace](/azure/azure-monitor/logs/log-analytics-workspace-overview).


     Log data collected and stored in a Log Analytics workspace from the hybrid machine contains properties specific to the machine, such as a Resource ID, to support [resource-context](/azure/azure-monitor/logs/manage-access#access-mode) log access.

     Watch this video to learn more about Azure monitoring, security, and update services across hybrid and multicloud environments.

     > [!VIDEO https://www.youtube.com/embed/mJnmXBrU1ao]
- Administrators can install the Azure Connected Machine agent at scale and leverage Azure Arc benefits such as [Windows Server management](/azure/azure-arc/servers/windows-server-management-overview?tabs=portal) for VMs with Software Assurance licenses, and pay-as-you-go billing for [Extended Security Updates](/azure/azure-arc/system-center-virtual-machine-manager/deliver-esus-for-system-center-virtual-machine-manager-vms) for Windows Server and SQL Server VMs.

### Unsupported scenarios

Azure Arc-enabled SCVMM doesn't support:

- Azure-based management of VMware vCenter VMs managed by SCVMM. To onboard VMware VMs to Azure Arc, use [Azure Arc-enabled VMware vSphere](/azure/azure-arc/vmware-vsphere/overview).
- Azure-based management of Azure Local VMs managed by SCVMM. To onboard Azure Local VMs to Azure Arc, use [Azure Arc VM management capabilities of Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview).

### Supported VMM versions

Azure Arc-enabled SCVMM works with VMM 2025, 2022, and 2019 versions. It supports SCVMM management servers with a maximum of 15,000 VMs.

### Supported regions

For the most up-to-date information about regional availability of Azure Arc-enabled SCVMM, see [Product Availability by Region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table).

## Data residency

Azure Arc-enabled SCVMM stores customer data. By default, customer data stays within the region the customer deploys the service instance in. For regions with data residency requirements, customer data is always kept within the same region.

## Next steps

- Plan your Azure Arc-enabled SCVMM deployment by reviewing the [support matrix](support-matrix-for-system-center-virtual-machine-manager.md).
- When ready, [connect your SCVMM management server to Azure Arc using the onboarding script](quickstart-connect-system-center-virtual-machine-manager-to-arc.md).
- [Deliver operations Management disciplines using hybrid and multicloud tools in Cloud adoption Framework](/azure/cloud-adoption-framework/scenarios/hybrid/manage).
- [Cloud Adoption Framework introduces Azure hybrid and multicloud products on Azure](/azure/cloud-adoption-framework/scenarios/hybrid/toolchain).

