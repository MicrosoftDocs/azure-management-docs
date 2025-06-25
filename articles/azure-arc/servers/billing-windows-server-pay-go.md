---
title: Billing service for Windows Server pay-as-you-go
description: Learn about billing services for Windows Server pay-as-you-go enabled by Azure Arc.
ms.date: 11/01/2024
ms.topic: concept-article
# Customer intent: "As a system administrator, I want to manage Windows Server pay-as-you-go billing through Azure Arc, so that I can efficiently control server resources and costs while leveraging cloud management benefits."
---

# Billing service for Windows Server pay-as-you-go enabled by Azure Arc

Machines must be onboarded to Azure Arc through the deployment of the [Azure Connected Machine agent](agent-overview.md). Azure Arc-enabled servers need to be in a connected state. Azure Arc-enabled servers don't support disconnected machines.  

Windows Server pay-as-you-go enabled by Azure Arc supports both Windows Server 2025 Standard and Datacenter editions. Windows Server Datacenter and Standard editions are billed at the same rate for pay-as-you-go enabled by Azure Arc. There's no difference between virtual and physical machines from a billing perspective. Also, there's no minimum CPU core count per machine or per subscription, nor a fixed increment in which CPU cores are counted.

## Benefits  

[Enrolling in Windows Server pay-as-you-go](/windows-server/get-started/windows-server-pay-as-you-go?tabs=gui%2Cazureportal) provides key server management capabilities. Customers benefit from the following management services at no extra cost: 

- Azure Update Manager 
- Azure Change Tracking and Inventory 
- Azure Machine Configuration 

Customers also benefit from access to the following management services: 

- Windows Admin Center 
- Windows Server Best Practices Assessment* 
- Windows Server Remote Support 
- Windows Server Accelerated Networking 
- Windows Server Disaster Recovery* 

For these services, extra costs may be incurred for compute, storage, and log ingestion.

## Trial period 

When you install Windows Server 2025 Standard or Datacenter, onboard your machine to Azure Arc, and enable Windows Server pay-as-you-go, you receive a seven day trial period for the enrolled Azure Arc-enabled server. During this period, invoicing reflects a trial entity for the Azure Arc-enabled servers. At the end of the seven day trial window, billing automatically commences for the Azure Arc-enabled server.  

## Changes in core counts 

If you change the number of cores for your Azure Arc-enabled server enrolled in Windows Server pay-as-you-go, the billing service adjusts the costs within 24 hours.

For example, if you increase the number of cores from 16 to 32, then within 24 hours the charges will be for 32 cores instead of 16 cores. Alternatively, if you decrease the number of cores from 16 to 8, then within 24 hours the charges will be for 8 cores instead of 16 cores.

## Disenrollment and deletion  

When you disable or disenroll an Azure Arc-enabled server from Windows Server pay-as-you-go, billing stops within 24 hours. Similarly, if you delete the Azure Arc-enabled server resource, the billing stops within 24 hours.

Uninstalling the Connected Machine agent won't stop billing unless you explicitly delete the configuration files, or the underlying machine is disconnected from Azure Arc.

If an Azure Arc-enabled server "expires" due to disconnection for over 30 days resulting in an expiration of the associated certificate, billing won't stop until the certificate is expired. 

## Resource moves 

If you move the Azure Arc-enabled server between resource groups, subscriptions, or regions, billing continues. Resource moves across tenants aren't supported for Windows Server pay-as-you-go enabled by Azure Arc.



