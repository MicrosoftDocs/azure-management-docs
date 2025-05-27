---
title: Azure Arc resource bridge security overview 
description: Understand security configuration and considerations for Azure Arc resource bridge.
ms.topic: concept-article
ms.date: 09/20/2024
# Customer intent: As a security administrator, I want to understand the security configuration and considerations for Azure Arc resource bridge, so that I can ensure compliance and protect sensitive data before deploying it in my enterprise environment.
---

# Azure Arc resource bridge security overview

This article describes the security configuration and considerations you should evaluate before deploying Azure Arc resource bridge in your enterprise.

## Managed identity

By default, a Microsoft Entra system-assigned [managed identity](/azure/active-directory/managed-identities-azure-resources/overview) is created and assigned to the Azure Arc resource bridge. Azure Arc resource bridge currently supports only a system-assigned identity. The `clusteridentityoperator` identity initiates the first outbound communication and fetches the Managed Service Identity (MSI) certificate used by other agents for communication with Azure.

## Identity and access control

Azure Arc resource bridge is represented as a resource in a resource group inside an Azure subscription. Access to this resource is controlled by standard [Azure role-based access control](/azure/role-based-access-control/overview). From the [**Access Control (IAM)**](/azure/role-based-access-control/role-assignments-portal) page in the Azure portal, you can verify who has access to your Azure Arc resource bridge.

Users and applications who are granted the [Contributor](/azure/role-based-access-control/built-in-roles#contributor) or Administrator role to the resource group can make changes to the resource bridge, including deploying or deleting cluster extensions.

## Data residency

Azure Arc resource bridge follows data residency regulations specific to each region. If applicable, data is backed up in a secondary pair region in accordance with data residency regulations. Otherwise, data resides only in that specific region. Data isn't stored or processed across different geographies.

## Data encryption at rest

Azure Arc resource bridge stores resource information in Azure Cosmos DB. As described in [Data encryption in Azure Cosmos DB](/azure/cosmos-db/database-encryption-at-rest), all the data is encrypted at rest.

## Security audit logs

The [activity log](/azure/azure-monitor/essentials/activity-log-insights) is an Azure platform log that provides insight into subscription-level events. This includes tracking when the Azure Arc resource bridge is modified, deleted, or added.

You can [view the activity log](/azure/azure-monitor/essentials/activity-log-insights#view-the-activity-log) in the Azure portal or retrieve entries with PowerShell and Azure CLI. By default, activity log events are [retained for 90 days](/azure/azure-monitor/essentials/activity-log-insights#retention-period) and then deleted.

## Next steps

- Understand [system requirements](system-requirements.md) and [network requirements](network-requirements.md) for Azure Arc resource bridge.
- Review the [Azure Arc resource bridge overview](overview.md) to understand more about features and benefits.
- Learn more about [Azure Arc](../overview.md).
