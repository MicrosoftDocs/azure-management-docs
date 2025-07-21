---
title: What is Azure Arc-enabled VMware vSphere?
description: Azure Arc-enabled VMware vSphere extends Azure governance and management capabilities to VMware vSphere infrastructure and delivers a consistent management experience across both platforms. 
ms.topic: overview
ms.date: 06/24/2025
ms.custom:
  - references_regions
  - build-2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
---

# What is Azure Arc-enabled VMware vSphere?

Azure Arc-enabled VMware vSphere is an [Azure Arc](../overview.md) service that helps you simplify management of hybrid IT estate distributed across VMware vSphere and Azure. It does so by extending the Azure control plane to VMware vSphere infrastructure and enabling the use of Azure experiences for VM management and Azure services for consistent security, governance, monitoring and patching across VMware vSphere on-premises private clouds, Azure VMware Solution (AVS) private clouds and Azure.

Azure Arc-enabled VMware vSphere allows you to:

- Discover your VMware vSphere estate (VMs, templates, networks, datastores, clusters/hosts/resource pools) and register resources with Azure at scale.

- Perform various virtual machine (VM) operations directly from Azure, such as create, resize, delete, and power cycle operations such as start/stop/restart on VMware VMs consistently with Azure. 

- Empower developers and application teams to self-serve VM operations on-demand using [Azure role-based access control](/azure/role-based-access-control/overview) (RBAC).

- Install the Azure connected machine agent at scale on VMware VMs to [govern, protect, configure, and monitor](../servers/overview.md#supported-cloud-operations) them.

- Browse your VMware vSphere resources (VMs, templates, networks, and storage) in Azure, providing you with a single pane view for your infrastructure across both environments.

- Build automation and self-service pipelines using Python, Java, JavaScript, Go, and .NET SDKs; Terraform, ARM, and Bicep templates; Azure REST APIs, CLI, and PowerShell.

- Leverage Azure Arc benefits such as [Windows Server management](/azure/azure-arc/servers/windows-server-management-overview?tabs=portal) for VMs with Software Assurance licenses, [Extended Security Updates](/azure/azure-arc/vmware-vsphere/deliver-extended-security-updates-for-vmware-vms-through-arc) benefits for Windows Server and SQL Server with pay-as-you-go billing for on-premises VMs and free ESUs for AVS VMs.

## How does it work?

Azure Arc-enabled VMware vSphere provides these capabilities by integrating with your VMware vCenter Server. To connect your VMware vCenter Server to Azure Arc, you need to deploy the [Azure Arc resource bridge](../resource-bridge/overview.md) in your vSphere environment. Azure Arc resource bridge is a virtual appliance that hosts the components that communicate with your vCenter Server and Azure. 

When a VMware vCenter Server is connected to Azure, an automatic discovery of the inventory of vSphere resources is performed. This inventory data is continuously kept in sync with the vCenter Server. 

All guest OS-based capabilities are provided by enabling guest management (installing the Arc agent) on the VMs. Once guest management is enabled, VM extensions can be installed to use the Azure management capabilities. You can perform virtual hardware operations such as resizing, deleting, adding disks, and power cycling without guest management enabled. 

## Architecture

The following image shows the architecture for the Azure Arc-enabled VMware vSphere:

:::image type="content" source="media/overview/architecture.png" alt-text="Screenshot of Arc enabled VMware vSphere - architecture." lightbox="media/overview/architecture.png":::

## How is Arc-enabled VMware vSphere different from Arc-enabled Servers

The easiest way to think of this is as follows:

- Azure Arc-enabled servers interact on the guest operating system level, with no awareness of the underlying infrastructure fabric and the virtualization platform that they're running on. Since Arc-enabled servers also support bare-metal machines, there can, in fact, not even be a host hypervisor in some cases.

- Azure Arc-enabled VMware vSphere is a superset of Arc-enabled servers that extends management capabilities beyond the guest operating system to the VM itself. This provides lifecycle management and CRUD (Create, Read, Update, and Delete) operations on a VMware vSphere VM. These lifecycle management capabilities are exposed in the Azure portal and look and feel just like a regular Azure VM. Azure Arc-enabled VMware vSphere also provides guest operating system management—in fact, it uses the same components as Azure Arc-enabled servers. 

You have the flexibility to start with either option, and incorporate the other one later without any disruption. With both the options, you enjoy the same consistent experience.

> [!NOTE]
>For guidance on choosing the right Azure Arc service for your virtual machines, see [Choose the right Azure Arc service for machines](../choose-service.md).

## Supported scenarios

- Azure Arc-enabled VMware vSphere currently works with vCenter Server versions 7 and 8 with a maximum of 9500 VMs.
- Multiple vCenters can be onboarded using a single Azure Arc resource bridge if the total number of VMs managed by these vCenters do not exceed 9500 VMs.
- Azure Arc-enabled VMware vSphere works with Azure VMware Solution (AVS) private clouds.
- Virtualized Infrastructure Administrators/Cloud Administrators can connect a vCenter instance to Azure.
- Administrators can then use the Azure portal to browse VMware vSphere inventory and register virtual machines resource pools, networks, and templates into Azure.
- Administrators can provide app teams/developers fine-grained permissions on those VMware resources through Azure RBAC.
- App teams can use Azure interfaces (portal, CLI, PowerShell, SDKs, Terraform, Bicep, ARM templates, or REST API) to manage the lifecycle of on-premises VMs they use for deploying their applications (CRUD, Start/Stop/Restart).
- Administrators can install Azure Connected Machine agent on vCenter-managed VMs at-scale and can perform the following actions:
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
- Application teams can create and manage Kubernetes clusters on VMware infrastructure using Azure consistent experiences with [Azure Kubernetes Services (AKS) Arc on VMware (Preview)](https://learn.microsoft.com/azure/aks/aksarc/aks-vmware-overview).

## Supported regions

For the most up-to-date information about region availability of Azure Arc-enabled VMware vSphere, see [Azure Products by Region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table) page.

## Data Residency

Azure Arc-enabled VMware vSphere doesn't store/process customer data outside the region the customer deploys the service instance in. By default, customer data stays within the region the customer deploys the service instance in. For region with data residency requirements, customer data is always kept within the same region.

## Next steps

- Plan your resource bridge deployment by reviewing the [support matrix for Arc-enabled VMware vSphere](support-matrix-for-arc-enabled-vmware-vsphere.md).
- Once ready, [connect VMware vCenter to Azure Arc using the helper script](quick-start-connect-vcenter-to-arc-using-script.md).
- To enable Arc for Azure VMware Solution (AVS) private cloud, see [Deploy Arc-enabled VMware vSphere for Azure VMware Solution private cloud](/azure/azure-vmware/deploy-arc-for-azure-vmware-solution?tabs=windows).
- Try out Azure Arc-enabled VMware vSphere by using the [Azure Arc Jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_vsphere).
- [Consider unified operations and plan for hybrid and multicloud environments with the Cloud Adoption Framework](/azure/cloud-adoption-framework/scenarios/hybrid/plan).
- [Choose the Azure Hybrid solution that meets your business requirements with guidance from the Azure Architecture Center](/azure/architecture/guide/technology-choices/hybrid-considerations).
