---
title: Connectivity modes and requirements
description: Explains Azure Arc-enabled data services connectivity options for from your environment to Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: anatripathi-git
ms.author: anatripathi
ms.reviewer: mikeray
ms.date: 07/19/2023
ms.topic: concept-article
# Customer intent: "As a system administrator, I want to understand the connectivity modes available for Azure Arc-enabled data services, so that I can choose the appropriate configuration for my infrastructure based on business policies and compliance requirements."
---

# Connectivity mode and requirements

This article describes the connectivity mode available for Azure Arc-enabled data services, and their respective requirements.

## Connectivity mode

Azure Arc-enabled data services supports direct connectivity mode. 

> [!NOTE]
> Indirectly connected is retired. [September, 2025](release-notes.md#september-2025).

When Azure Arc-enabled data services are directly connected to Azure, you can use [Azure Resource Manager APIs](/rest/api/resources/), the Azure CLI, and the Azure portal to operate the Azure Arc data services. The experience in directly connected mode is much like how you would use any other Azure service with provisioning/de-provisioning, scaling, configuring, and so on, all in the Azure portal. 

Additionally, Microsoft Entra ID and Azure Role-Based Access Control can be used in the directly connected mode only because there's a dependency on a continuous and direct connection to Azure to provide this functionality.

Some Azure-attached services are only available when they can be directly reached such as Container Insights, and automatic backup to Azure blob storage.

- Directly connected
    - Description: Provides all services when a direct connection to Azure is available. Connections are always initiated from your environment to Azure using standard ports/protocols (for example HTTPS/443).
    - Current availability: Available
    - Typical use cases: Public cloud environments (Azure, AWS, GCP); edge sites with permitted Internet connectivity (retail, manufacturing); corporate datacenters with permissive connectivity policies.
    - How data is sent to Azure: Data is automatically and continuously sent to Azure.

- Indirectly connected (Retired)
    - Description: Offers most management services locally with no continuous connection to Azure. A minimal amount of billing and inventory data is exported to a file and uploaded to Azure at least once per month; some Azure-dependent features are unavailable.
    - Current availability: Retired
    - Typical use cases: Regulated on-premises data centers (financial, healthcare, government); edge sites without Internet (oil/gas, military); sites with intermittent connectivity (stadiums, cruise ships).
    - How data is sent to Azure: One of three options — (1) automated process that exports from the secure region to Azure, (2) automated export to a less secure region and upload to Azure, or (3) manual export and upload. The first two can be scheduled for frequent transfers.

## Feature availability by connectivity mode


|**Feature**|**Indirectly connected** (Retired) |**Directly connected**|
|---|---|---|
|**Automatic high availability**|Supported|Supported|
|**Self-service provisioning**|Supported<br/>Use Azure Data Studio, the appropriate CLI, or Kubernetes native tools like Helm, `kubectl`, or `oc`, or use Azure Arc-enabled Kubernetes GitOps provisioning.|Supported<br/>In addition to the indirectly connected mode creation options, you can also create through the Azure portal, Azure Resource Manager APIs, the Azure CLI, or ARM templates. 
|**Elastic scalability**|Supported|Supported<br/>|
|**Billing**|Supported<br/>Billing data is periodically exported out and sent to Azure.|Supported<br/>Billing data is automatically and continuously sent to Azure and reflected in near real time. |
|**Inventory management**|Supported<br/>Inventory data is periodically exported out and sent to Azure.<br/><br/>Use client tools like Azure Data Studio, Azure Data CLI, or `kubectl` to view and manage inventory locally.|Supported<br/>Inventory data is automatically and continuously sent to Azure and reflected in near real time. As such, you can manage inventory directly from the Azure portal.|
|**Automatic upgrades and patching**|Supported<br/>The data controller must either have direct access to the Microsoft Container Registry (MCR) or the container images need to be pulled from MCR and pushed to a local, private container registry that the data controller has access to.|Supported|
|**Automatic backup and restore**|Supported<br/>Automatic local backup and restore.|Supported<br/>In addition to automated local backup and restore, you can _optionally_ send backups to Azure blob storage for long-term, off-site retention.|
|**Monitoring**|Supported<br/>Local monitoring using local dashboards.|Supported<br/>In addition to local monitoring dashboards, you can _optionally_ send monitoring data and logs to Azure Monitor for at-scale monitoring of multiple sites in one place. |
|**Authentication**|Use local username/password for data controller and dashboard authentication. Use SQL and Postgres logins or Active Directory (AD isn't currently supported) for connectivity to database instances. Use Kubernetes authentication providers for authentication to the Kubernetes API.|In addition to or instead of the authentication methods for the indirectly connected mode, you can _optionally_ use Microsoft Entra ID.|
|**Role-based access control (RBAC)**|Use Kubernetes RBAC on Kubernetes API. Use SQL and Postgres RBAC for database instances.|You can use Microsoft Entra ID and Azure RBAC.|

## Connectivity requirements

**Some functionality requires a connection to Azure.**

**All communication with Azure is always initiated from your environment.** This is true even for operations that are initiated by a user in the Azure portal. In that case, there's effectively a task, which is queued up in Azure. An agent in your environment initiates the communication with Azure to see what tasks are in the queue, runs the tasks, and reports back the status/completion/fail to Azure.

|**Type of Data**|**Direction**|**Required/Optional**|**Additional Costs**|**Mode Required**|**Notes**|
|---|---|---|---|---|---|
|**Container images**|Microsoft Container Registry -> Customer|Required|No|Indirect or direct|Container images are the method for distributing the software. In an environment which can connect to the Microsoft Container Registry (MCR) over the Internet, the container images can be pulled directly from MCR. If the deployment environment doesn’t have direct connectivity, you can pull the images from MCR and push them to a private container registry in the deployment environment. At creation time, you can configure the creation process to pull from the private container registry instead of MCR. This also applies to automated updates.|
|**Resource inventory**|Customer environment -> Azure|Required|No|Indirect or direct|An inventory of data controllers, database instances is kept in Azure for billing purposes and also for purposes of creating an inventory of all data controllers and database instances in one place which is especially useful if you have more than one environment with Azure Arc data services. As instances are provisioned, deprovisioned, scaled out/in, scaled up/down the inventory is updated in Azure.|
|**Billing telemetry data**|Customer environment -> Azure|Required|No|Indirect or direct|Utilization of database instances must be sent to Azure for billing purposes. |
|**Monitoring data and logs**|Customer environment -> Azure|Optional|Maybe depending on data volume (see [Azure Monitor pricing](https://azure.microsoft.com/pricing/details/monitor/))|Indirect or direct|You might want to send the locally collected monitoring data and logs to Azure Monitor for aggregating data across multiple environments into one place and also to use Azure Monitor services like alerts, using the data in Azure Machine Learning, etc.|
|**Azure Role-based Access Control (Azure RBAC)**|Customer environment -> Azure -> Customer Environment|Optional|No|Direct only|If you want to use Azure RBAC, then connectivity must be established with Azure at all times. If you don’t want to use Azure RBAC, then local Kubernetes RBAC can be used.|
|**Microsoft Entra ID (Future)**|Customer environment -> Azure -> Customer environment|Optional|Maybe, but you might already be paying for Microsoft Entra ID|Direct only|If you want to use Microsoft Entra ID for authentication, then connectivity must be established with Azure at all times. If you don’t want to use Microsoft Entra ID for authentication, you can use Active Directory Federation Services (ADFS) over Active Directory. **Pending availability in directly connected mode**|
|**Backup and restore**|Customer environment -> Customer environment|Required|No|Direct or indirect|The backup and restore service can be configured to point to local storage classes. |
|**Azure backup - long term retention (Future)**| Customer environment -> Azure | Optional| Yes for Azure storage | Direct only |You might want to send backups that are taken locally to Azure Backup for long-term, off-site retention of backups and bring them back to the local environment for restore. |
|**Provisioning and configuration changes from Azure portal**|Customer environment -> Azure -> Customer environment|Optional|No|Direct only|Provisioning and configuration changes can be done locally using Azure Data Studio or the appropriate CLI. In directly connected mode, you can also provision and make configuration changes from the Azure portal.|

## Details on internet addresses, ports, encryption, and proxy server support

[!INCLUDE [network-requirements](includes/network-requirements.md)]

## Additional network requirements

In addition, resource bridge requires [Arc-enabled Kubernetes endpoints](../network-requirements-consolidated.md#azure-arc-enabled-kubernetes-endpoints).
