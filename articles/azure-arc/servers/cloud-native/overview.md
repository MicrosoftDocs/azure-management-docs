---
title: Cloud-native server management with Azure Arc-enabled servers
description: Azure Arc transforms traditional server management by extending Azure's control plane to your on-premises environment. 
ms.date: 08/19/2025
ms.topic: overview
# Customer intent: "As a system architect managing a hybrid cloud environment, I want to understand how to extend Azure's management capabilities to my on-premises servers."
---

# Cloud-native server management with Azure Arc-enabled servers

By using [Azure Arc-enabled servers](../overview.md), your Windows and Linux machines outside Azure (in your datacenter or other clouds) become Azure resources. You can manage them just like Azure virtual machines (VMs): organize them into resource groups, apply policies, run scripts, and tag them for search, all within the Azure portal or by using tools like Azure CLI. The journey isn't just about moving VMs to Azure; it's about shifting the entire management experience (inventory, configuration, governance, scripting, patching, identity) into Azure's unified platform.

When you connect a server by using the [Azure Connected Machine agent](../agent-overview.md), it gets a unique Azure Resource ID and appears in your subscription along with native Azure resources. You can use Azure to perform tasks that used to require disparate on-premises tools (such as Active Directory Group Policy, Microsoft Configuration Manager (SCCM), Microsoft Endpoint Configuration Manager (MECM), or PowerShell remoting). For example, you can use [Azure Policy](/azure/governance/policy/overview) to audit or enforce OS settings (similar to GPO), [Azure Update Manager](/azure/update-manager/overview) to schedule patches (replacing WSUS/SCCM maintenance plans), and [Run Command](../run-command.md) to execute scripts remotely (instead of RDP/SSH into each server). Azure Arc brings cloud practices such as centralized management, at-scale automation, and uniform governance to all your servers. As a system administrator, you can manage hybrid infrastructure from a "single pane of glass" in Azure.

If you're comfortable with Active Directory and System Center, Azure Arc and Azure VM management is Microsoft's next-generation analog for servers, much like Microsoft Intune is for client endpoints. You still use familiar concepts (groups, policies, identity), but through Azure’s lens. The big picture is a holistic shift: not only is infrastructure gradually moving to Azure, but so is the control plane for managing that infrastructure.

The following table provides an overview of key Azure services and concepts that enable cloud-native server management.

| Core Functionality | Azure Arc capabilities |
| --- | --- |
| **Governance** | [Azure Policy](/azure/governance/policy/overview) provides a unified way to define and enforce policies across Azure Arc-enabled servers and native Azure VMs. You can audit compliance, enforce configurations, and remediate noncompliant resources. [Azure Machine Configuration](/azure/governance/machine-configuration/) extends these capabilities by allowing you to author customized Desired State Configurations, ensuring consistency across your server fleet. |
| **Patching** | [Azure Update Manager](/azure/update-manager/) provides a centralized solution for update assessment and management across Windows and Linux. [Hotpatching](/azure/update-manager/manage-hot-patching-arc-machines) can deliver OS security updates  without requiring a reboot. |
| **Inventory** | [Azure Resource Manager](/azure/azure-resource-manager/management/overview) provides a management layer for creating, updating, and deleting resources, with features like access control, locks, and tags to secure and organize resources after deployment. [Azure Change Tracking and Inventory](/azure/automation/change-tracking/overview-monitoring-agent) monitors changes and provides detailed inventory logs for servers across Azure, on-premises, and other cloud environments. [Azure Resource Graph](/azure/governance/resource-graph/) can be used for custom queries and reporting across a fleet of servers. |
| **Scripting** | [Run Command](../run-command.md) allows administrators to remotely and securely execute scripts for various server management tasks, including application management, security enforcement, and diagnostics. [SSH access to Arc-enabled servers](../ssh-arc-overview.md) enables connection without requiring a public IP address or additional open ports. |
| **Software deployment** | [Virtual Machine Applications](/azure/virtual-machines/vm-applications) (VM Applications) allows administrators to safely package and distribute software to their Azure VMs. While Azure Arc-enabled servers don't currently support VM Applications, you can use Run Command scripts and customized Machine Configuration for software distribution. |
| **Licensing** | Pay-as-you-go licensing for Windows Server and SQL Server lets you pay for only what you use when you use it. Extended Security Updates (ESUs) via Azure Arc simplifies the process of delivering security updates to your on-premises Windows Server 2012 machines. |
| **Identity** | [Microsoft Entra](/entra/fundamentals/whatis) helps manage user identities, apps, and access to Microsoft resources. [Azure  role-based access control (Azure RBAC)](/azure/role-based-access-control/overview) lets you assign specific roles to follow the principle of least privilege. |

Explore the articles in this section to learn how typical on-premises system administration tasks can be achieved with cloud-native methods through Azure-Arc enabled servers.

- [Cloud-native inventory and resource organization](inventory-resource.md)
- [Cloud-native governance and policy](governance-policy.md)
- [Cloud-native licensing and cost management](licensing-cost-management.md)
- [Cloud-native scripting and task automation](scripting-task-automation.md)
- [Cloud-native patch management](patch-management.md)
- [Cloud-native identity and access management](identity-access.md)
- [Cloud-native monitoring and alerts](monitor-alerts.md)
- [Next steps on the cloud native journey](next-steps.md)
