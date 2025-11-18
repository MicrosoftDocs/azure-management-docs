---
title: Resiliency agent capabilities in Agents (preview) in Azure Copilot
description: Agents (preview) in Azure Copilot lets you get resiliency help using intelligent agent capabilities.
author: JnHs
ms.author: jenhayes
ms.date: 11/18/2025
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use agents to help with troubleshooting, so that I can be more successful performing tasks in  my Azure tenant."
---

# Resiliency capabilities in Agents (preview) in Azure Copilot

[Agents (preview) in Azure Copilot](agents-preview.md) intelligently surfaces the right agent to help with your tasks. When you ask Azure Copilot for help with resiliency tasks, you can get help with ensuring your Azure resources are resilient and can recover from failures.

Azure Copilot can help you improve the resiliency of resources and applications against cyber attacks, data corruptions and datacenter outages. You can get information about service groups and resources that are missing zonal resiliency goals, are vulnerable to outages, or have key alerts pending and need attention. When issues are identified, Azure Copilot guides you to enable solutions such as Azure Backup, Azure Site Recovery, and native zonal resiliency solutions. Azure Copilot can even provide ready-to-deploy scripts to configure zonal resiliency for supported resource types.

The resiliency capabilities of Azure Copilot help you improve the security posture of recovery services and Azure Backup vaults. You can identify resources that lack recent backups due to failed backup jobs or lack of a backup policy, then fix the issues with those resources. Azure Copilot can help you perform Backup vault operations such as creation, deletion, data copy, and troubleshooting errors and failed backups. You can also review resources that don't have Azure Backup or Azure Site Recovery configured, helping to identify resources that could be vulnerable during outages or attacks.

> [!IMPORTANT]
> The functionality described in this article is only available for tenants that have access to [Agents (preview) in Azure Copilot](agents-preview.md).

## Resiliency sample prompts

Here are a few examples of the kinds of prompts you can use to get help with resiliency tasks. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. When using these kinds of prompts, be sure to enable agent mode by selecting the icon in the chat window.

:::image type="content" source="media/agent-preview/azure-copilot-agent-mode-active.png" alt-text="Screenshot showing agent mode (preview) enabled in Azure Copilot.":::

- "Configure zonal resiliency for this resource."
- "Which service groups are currently not zonally resilient?"
- "Which resources aren't zonally resilient?"
- "What are the steps to define a drill?
- "How can I define a recovery plan?"
- "What are the key alerts raised since the past 24 hours?"
- "How many backup jobs failed in the last 24 hours?"
- "Help me create vault `ABC`."
- "Help me delete this vault."
- "Increase the security level of this vault."
- "Which data sources don't have a recovery point within the last 7 days?"

## Current considerations and limitations

Keep in mind the following considerations and limitations when working with resiliency in Agents (preview) in Azure Copilot.

- Ready-to-deploy scripts to configure zonal resiliency are currently supported only for virtual machines, App Services, Azure Database for PostgreSQL, Azure Database for MySQL, SQL Managed Instance, Azure Cache for Redis, and Azure Firewall.
- Configuration of some advanced capabilities, such as multi-user authorization and Azure Site Recovery, require manual intervention. Azure Copilot can provide guidance to help you complete these tasks.
