---
title: Azure Arc overview
description: Learn about what Azure Arc is and how it helps customers enable management and governance of their hybrid resources with other Azure services and features.
ms.date: 08/25/2025
ms.topic: overview
# Customer intent: As an IT administrator managing a hybrid environment, I want to use Azure Arc to unify the management of my on-premises and cloud resources, so that I can simplify governance and operational processes across diverse infrastructures.
---

# Azure Arc overview

Today, companies struggle to control and govern increasingly complex environments that extend across data centers, multiple clouds, and edge. Each environment and cloud possesses its own set of management tools, and new DevOps and ITOps operational models can be hard to implement across resources.

Azure Arc simplifies governance and management by delivering a consistent multicloud and on-premises management platform.

Azure Arc provides a centralized, unified way to:

* Manage your entire environment together by projecting your existing non-Azure and/or on-premises resources into Azure Resource Manager.
* Manage virtual machines, Kubernetes clusters, and databases as if they are running in Azure.
* Use familiar Azure services and management capabilities, regardless of where your resources live.
* Continue using traditional ITOps while introducing DevOps practices to support new cloud native patterns in your environment.
* Configure custom locations as an abstraction layer on top of Azure Arc-enabled Kubernetes clusters and cluster extensions.  

:::image type="content" source="media/overview/azure-arc-control-plane.png" alt-text="Diagram showing the Azure Arc management control plane." lightbox="media/overview/azure-arc-control-plane.png" :::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

Currently, Azure Arc allows you to manage the following resource types hosted outside of Azure:

* [Servers](servers/overview.md) and virtual machines: Manage Windows and Linux physical servers and virtual machines hosted outside of Azure. Provision, resize, delete, and manage virtual machines based on [Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview) and on [VMware vCenter](./vmware-vsphere/overview.md) or [System Center Virtual Machine Manager](./system-center-virtual-machine-manager/overview.md) managed on-premises environments.
* [Kubernetes clusters](kubernetes/overview.md): Attach and configure Kubernetes clusters running anywhere, with multiple supported distributions.
* [Azure data services](data/overview.md): Run SQL Managed Instance on-premises, at the edge, and in public clouds using Kubernetes and the infrastructure of your choice.
* [SQL Server](/sql/sql-server/azure-arc/overview): Extend Azure services to SQL Server instances hosted outside of Azure.

> [!NOTE]
> For more information regarding the different services Azure Arc offers, see [Choosing the right Azure Arc service for machines](/azure/azure-arc/choose-service).

## Key features and benefits

Some of the key scenarios that Azure Arc supports are:

* Implement consistent inventory, management, governance, and security for servers across your environment.

* Configure [Azure VM extensions](./servers/manage-vm-extensions.md) to use Azure management services to monitor, secure, and update your servers.

* Manage and govern Kubernetes clusters at scale.

* [Use GitOps to deploy configurations](kubernetes/conceptual-gitops-flux2.md) across one or more clusters from Git repositories.

* Zero-touch compliance and configuration for Kubernetes clusters using Azure Policy.

* Run [Azure data services](../azure-arc/kubernetes/custom-locations.md) on any Kubernetes environment as if it runs in Azure (specifically Azure SQL Managed Instance, with benefits such as upgrades, updates, security, and monitoring). Use elastic scale and apply updates without any application downtime, even without continuous connection to Azure.

* Create [custom locations](./kubernetes/custom-locations.md) on top of your [Azure Arc-enabled Kubernetes](./kubernetes/overview.md) clusters, using them as target locations for deploying Azure services instances. Deploy your Azure service cluster extensions for [Azure Arc-enabled data services](./data/create-data-controller-direct-azure-portal.md), [Azure Container Apps on Azure Arc](/azure/container-apps/azure-arc-overview), and [Event Grid on Kubernetes](/azure/event-grid/kubernetes/overview).

* Perform virtual machine lifecycle and management operations on [Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview) and on-premises environments managed by [VMware vCenter](./vmware-vsphere/overview.md) and [System Center Virtual Machine Manager (SCVMM)](./system-center-virtual-machine-manager/overview.md) through interactive and non-interactive methods. Empower developers and application teams to self-serve VM operations on-demand using Azure role-based access control (RBAC).

* A unified experience viewing your Azure Arc-enabled resources, whether you are using the Azure portal, the Azure CLI, Azure PowerShell, or Azure REST API.

## Pricing

Below is pricing information for the features available today with Azure Arc.

### Azure Arc-enabled servers

The following Azure Arc control plane functionality is offered at no extra cost:

* Resource organization through Azure management groups and tags
* Searching and indexing through Azure Resource Graph
* Access and security through Azure Role-based access control (RBAC)
* Environments and automation through templates and extensions

Any Azure service that is used on Azure Arc-enabled servers, such as Microsoft Defender for Cloud or Azure Monitor, will be charged as per the pricing for that service. For more information, see the [Azure pricing page](https://azure.microsoft.com/pricing/).

### Azure Arc-enabled VMware vSphere and System Center Virtual Machine Manager

The following Azure Arc-enabled VMware vSphere and System Center Virtual Machine Manager (SCVMM) capabilities are offered at no extra cost:

- All the Azure Arc control plane functionalities that are offered at no extra cost with Azure Arc-enabled servers.
- Discovery and single pane of glass inventory view of your VMware vCenter and SCVMM managed estate (VMs, templates, networks, datastores, clouds/clusters/hosts/resource pools).
- Lifecycle (create, resize, update, and delete) and power cycle (start, stop, and restart) operations of VMs, including the ability to delegate self-service access for these operations using Azure role-based access control (RBAC).
- Management of VMs using Azure portal, CLI, REST APIs, SDKs, and automation through Infrastructure as Code (IaC) templates such as ARM, Terraform, and Bicep.

Any Azure service that is used on Azure Arc-enabled VMware vSphere and SCVMM VMs, such as Microsoft Defender for Cloud or Azure Monitor, will be charged as per the pricing for that service. For more information, see the [Azure pricing page](https://azure.microsoft.com/pricing/).

### Azure Arc-enabled Kubernetes

Any Azure service that is used on Azure Arc-enabled Kubernetes, such as Microsoft Defender for Cloud or Azure Monitor, will be charged as per the pricing for that service.

For more information on pricing for configurations on top of Azure Arc-enabled Kubernetes, see the [Azure pricing page](https://azure.microsoft.com/pricing/).

### Azure Arc-enabled data services

For information, see the [Azure pricing page](https://azure.microsoft.com/pricing/).

## Azure Arc and the adaptive cloud approach

Azure Arc is a key part of Microsoft’s [adaptive cloud](https://azure.microsoft.com/solutions/adaptive-cloud) approach. This approach helps organizations run and manage apps and services across many environments, including Azure, other cloud providers, on-premises datacenters, and edge locations. Azure Arc supports this approach by extending Azure’s management, security, and governance tools to resources outside Azure. This ability makes it easier to keep operations consistent and secure, no matter where your workloads run.

## Next steps

* [Choose the right Azure Arc service for your physical and virtual machines](./choose-service.md).
* Learn about [Azure Arc-enabled servers](./servers/overview.md).
* Learn about [Azure Arc-enabled Kubernetes](./kubernetes/overview.md).
* Learn about [Azure Arc-enabled data services](https://azure.microsoft.com/services/azure-arc/hybrid-data-services/).
* Learn about [SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/overview).
* Learn about [Azure Arc-enabled VM Management on Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview).
* Learn about [Azure Arc-enabled VMware vSphere](vmware-vsphere/overview.md).
* Learn about [Azure Arc-enabled System Center Virtual Machine Manager](system-center-virtual-machine-manager/overview.md).
* Experience Azure Arc by exploring the [Azure Arc Jumpstart](https://aka.ms/AzureArcJumpstart).
* Learn about best practices and design patterns through the [Azure Arc Landing Zone Accelerators](https://aka.ms/ArcLZAcceleratorReady).
* Understand [network requirements for Azure Arc](network-requirements-consolidated.md).
