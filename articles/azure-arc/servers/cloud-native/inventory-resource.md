---
title: Cloud-native inventory and resource organization with Azure Arc-enabled servers
description: Cloud-native inventory means all your servers, Azure or not, show up in one consolidated view, and you use Azure's organizational tools to sort and manage them.
ms.date: 08/01/2025
ms.topic: concept-article
# Customer intent: "As a system architect managing a hybrid cloud environment, I want to understand Azure's hierarchy for organizing cloud resources and how to use Azure to manage my hybrid server organization."
---

# Cloud-native inventory and resource organization with Azure Arc-enabled servers

Cloud-native inventory means all your servers show up in one consolidated view, and you use Azure's organizational tools to sort and manage them. You can organize your resources, apply tags as labels, and use tools like Change Tracking and Azure Resource Graph for a live inventory. The result is a clear view of "servers everywhere" from Azure, with flexible grouping to meet your organization's needs.

## Resource organization in Azure

Managing a server inventory in Azure starts with understanding Azure's resource hierarchy and how it compares to traditional Active Directory (AD) organization. In a domain environment, you might have security groups, device collections, forests, domains, and OUs (Organizational Units) to group servers, often reflecting business structure or locations. Azure has its own hierarchy for organizing cloud resources, including the concepts described in this section.

### Azure tenants

A tenant is a [Microsoft Entra ID](/entra/fundamentals/whatis) entity that typically encompasses an organization. You can think of a tenant as the top-level container for all your Azure resources and identities, analogous in some ways to an AD forest. It's the trust boundary within which your subscriptions and users reside. However, Microsoft Entra is primarily an identity store, and it doesn't organize servers by OU. Instead, Azure uses subscriptions, management groups, and resource groups for resource organization.

### Subscriptions and management groups

An Azure *subscription* is a billing and resource container within your tenant. For example, you might use separate subscriptions for different environments or departments.

Multiple subscriptions can be grouped under *management groups* to apply governance, similar to how multiple domains or sites in AD might be overseen together. Management groups let you create hierarchies that reflect your organization's structure and apply policies or access controls across subscriptions. You might think of management groups as higher-level groupings (like an entire company or division) and subscriptions as major units of deployment.

### Resource groups

In Azure, a resource group is the closest concept to an organizational unit (OU) for resources. It's a logical container within a subscription that can hold related resources such as VMs, Arc-enabled servers, and storage accounts.

System administrators can use resource groups to group servers by application, environment, or location, and then assign policies and user roles at that group level. For example, just as you might have an "HQ-Servers" OU with delegated admins and certain Active Directory Group Policy Objects (GPOs), you could have an "HQ-Servers" resource group with specific role assignments and Azure policies. In Azure, a server can only belong to one resource group at a time (similar to OU nesting), but you can move it between resource groups if needed.

### Tags

In Active Directory or System Center Configuration Manager (SCCM), you might use attributes or create dynamic collections (queries) to group servers together. In Azure, you can achieve this [categorization by using tags](/azure/azure-resource-manager/management/tag-resources), which are custom key-value metadata you attach to resources.

Tags let you flexibly group and filter resources across resource groups or even subscriptions. You can use Azure Resource Graph queries to list or report on all servers with certain tags, similar to running a query in SCCM for a collection. This capability provides inventory views that aren't bound by a single hierarchy, an advantage over static OU groupings.

### Azure Service Groups

A new concept in preview, [Azure Service Groups](/azure/governance/service-groups/overview) allow dynamic grouping of resources across subscriptions and resource groups. Think of it as creating a custom group of servers (or other resources) from anywhere in your tenant to monitor or manage together.

For example, you could create an "All SQL Servers" group to aggregate SQL Server instances across multiple resource groups, without including other resource types. Service Groups enable multiple hierarchies in parallel (one resource can belong to multiple groups) without changing the underlying resource organization.

While you can't apply Azure Policy or Azure role-based access control (Azure RBAC) to Service Groups, they can be very useful for inventory views and aggregated monitoring of related systems. These uses are similar to how you might use SCCM device collections or AD dynamic groups to watch a set of machines.

## Azure Change Tracking and Inventory

[Azure Change Tracking and Inventory](/azure/automation/change-tracking/overview-monitoring-agent) continuously logs changes in software, registry, services, and files on Azure VMs and Arc-enabled servers, as if you had an always-on Configuration Management Database (CMDB) and change auditor. Detailed records show what software is installed on each server and when changes were made, helping with troubleshooting and compliance. For example, you can see when a service was stopped, an update was applied, or a configuration file was changed.

## Azure Resource Graph

Azure Resource Graph is a powerful query engine that lets you query all your Azure resources (including Arc-enabled servers) using Kusto Query Language. For example, you can run a query to find all Arc-enabled servers in a specific region with a certain tag, or all VMs running Windows Server 2012 in real-time.

The ability to create custom queries becomes extremely useful as your environment scales, helping you maintain visibility across all of your servers. And because Azure Arc treats on-premises servers as Azure resources, these servers are included in these queries by default, with no separate inventory system needed.
