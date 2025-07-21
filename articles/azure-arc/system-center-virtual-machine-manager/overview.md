---
title:  Overview of the Azure Arc-enabled System Center Virtual Machine Manager 
description: This article provides a detailed overview of the Azure Arc-enabled System Center Virtual Machine Manager.
ms.date: 06/18/2025
ms.topic: overview
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: jsuri
author: jyothisuri
keywords: "VMM, Arc, Azure, System Center"
ms.custom:
  - references_regions
  - build-2025
# Customer intent: As a system administrator, I want to connect my System Center Virtual Machine Manager to Azure, so that I can manage VMs consistently across hybrid environments using Azure's security and governance capabilities.
---

# Overview of Azure Arc-enabled System Center Virtual Machine Manager

Azure Arc-enabled System Center Virtual Machine Manager (SCVMM) empowers System Center customers to connect their VMM environment to Azure and perform VM self-service operations from Azure portal. Azure Arc-enabled SCVMM extends the Azure control plane to SCVMM managed infrastructure, enabling the use of Azure security, governance, and management capabilities consistently across System Center managed estate and Azure.

Azure Arc-enabled SCVMM also allows you to manage your hybrid environment consistently and perform self-service VM operations through Azure portal. For Microsoft Azure Pack customers, this solution is intended as an alternative to perform VM self-service operations.

Azure Arc-enabled SCVMM allows you to:

- Perform various VM lifecycle operations such as start, stop, pause, and delete VMs on SCVMM managed VMs directly from Azure.
- Empower developers and application teams to self-serve VM operations on demand using [Azure role-based access control (RBAC)](/azure/role-based-access-control/overview).
- Browse your VMM resources (VMs, templates, VM networks, and storage) in Azure, providing you with a single pane view for your infrastructure across both environments.
- Discover and onboard existing SCVMM managed VMs to Azure.
- Install the Azure Connected Machine agent at scale on SCVMM VMs to [govern, protect, configure, and monitor them](../servers/overview.md#supported-cloud-operations).
- Build automation and self-service pipelines using Python, Java, JavaScript, Go, and .NET SDKs; Terraform, ARM, and Bicep templates; Azure REST APIs, CLI, and PowerShell.
- Leverage Azure Arc benefits such as [Windows Server management](/azure/azure-arc/servers/windows-server-management-overview?tabs=portal) for VMs with Software Assurance licenses, and pay-as-you-go billing for [Extended Security Updates](/azure/azure-arc/system-center-virtual-machine-manager/deliver-esus-for-system-center-virtual-machine-manager-vms) for Windows Server and SQL Server VMs. 

## How does it work?

To Arc-enable an SCVMM management server, deploy [Azure Arc resource bridge](../resource-bridge/overview.md) in the VMM environment. Azure Arc resource bridge is a virtual appliance that connects VMM management server to Azure. Azure Arc resource bridge enables you to represent the SCVMM resources (clouds, VMs, templates etc.) in Azure and do various operations on them.

## Architecture

The following image shows the architecture for the Azure Arc-enabled SCVMM:

:::image type="content" source="media/architecture/azure-arc-scvmm-architecture.png" alt-text="Screenshot of Arc enabled SCVMM - architecture." lightbox="media/architecture/azure-arc-scvmm-architecture.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

## How is Azure Arc-enabled SCVMM different from Azure Arc-enabled servers

- Azure Arc-enabled servers interact on the guest operating system level, with no awareness of the underlying infrastructure fabric and the virtualization platform that they're running on. Since Azure Arc-enabled servers also support bare-metal machines, there might, in fact, not even be a host hypervisor in some cases.
- Azure Arc-enabled SCVMM is a superset of Azure Arc-enabled servers that extends management capabilities beyond the guest operating system to the VM itself. This provides lifecycle management and CRUD (Create, Read, Update, and Delete) operations on an SCVMM VM. These lifecycle management capabilities are exposed in the Azure portal and look and feel just like a regular Azure VM. Azure Arc-enabled SCVMM also provides guest operating system management, in fact, it uses the same components as Azure Arc-enabled servers.

You have the flexibility to start with either option, and incorporate the other one later without any disruption. With both options, you'll enjoy the same consistent experience.

> [!NOTE]
>For guidance on choosing the right Azure Arc service for your virtual machines, see [Choose the right Azure Arc service for machines](../choose-service.md).

### Supported scenarios

The following scenarios are supported in Azure Arc-enabled SCVMM:

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

- Azure-based management of VMware vCenter VMs managed by SCVMM. To onboard VMware VMs to Azure Arc, we recommend you to use [Azure Arc-enabled VMware vSphere](/azure/azure-arc/vmware-vsphere/overview).
- Azure-based management of Azure Local VMs managed by SCVMM. To onboard Azure Local VMs to Azure Arc, we recommend you to use [Azure Arc VM management capabilities of Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview).

### Supported VMM versions

Azure Arc-enabled SCVMM works with VMM 2025, 2022, and 2019 versions and supports SCVMM management servers with a maximum of 15,000 VMs.

### Supported regions

For the most up-to-date information about regional availability of Azure Arc-enabled SCVMM, see [Product Availability by Region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table) page.

## Data Residency

Azure Arc-enabled SCVMM stores customer data. By default, customer data stays within the region the customer deploys the service instance in. For region with data residency requirements, customer data is always kept within the same region.

## Next steps

- Plan your Azure Arc-enabled SCVMM deployment by reviewing the [support matrix](support-matrix-for-system-center-virtual-machine-manager.md).
- Once ready, [connect your SCVMM management server to Azure Arc using the onboarding script](quickstart-connect-system-center-virtual-machine-manager-to-arc.md).
- [Deliver operations Management disciplines using hybrid and multicloud tools in Cloud adoption Framework](/azure/cloud-adoption-framework/scenarios/hybrid/manage).
- [Cloud Adoption Framework introduces Azure hybrid and multicloud products on Azure](/azure/cloud-adoption-framework/scenarios/hybrid/toolchain).

