---
title: Migration agent capabilities in Agents (preview) in Azure Copilot
description: Agents (preview) in Azure Copilot lets you get help with migration tasks help using intelligent agent capabilities.
author: JnHs
ms.author: jenhayes
ms.date:  02/10/2026
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use agents to help with migration, so that I can be more successful performing tasks in  my Azure tenant."
---

# Migration capabilities in Agents (preview) in Azure Copilot

[Agents (preview) in Azure Copilot](agents-preview.md) intelligently surfaces the right agent to help with your tasks. When you ask Azure Copilot for help with [migration tasks](/azure/migrate/migrate-services-overview), you can get help with planning, assessing, strategizing, and moving workloads to Azure.

Azure Copilot works to understand your migration goals, such as faster migration, modernization to PaaS, and regional targets, then creates artifacts such as [business cases](/azure/migrate/concepts-business-case-calculation) and [assessments](/azure/migrate/concepts-overview) using your existing context. You can also get help setting up [landing zones](/azure/cloud-adoption-framework/ready/landing-zone/), comparing alternative options, and resolving blocking issues.

> [!IMPORTANT]
> The functionality described in this article is only available for tenants that have access to [Agents (preview) in Azure Copilot](agents-preview.md) and don't use the [**Bring your own storage** option for conversation history in Azure Copilot](bring-your-own-storage.md). If your tenant uses **Bring your own storage**, you can still ask Azure Copilot for help with migration tasks, but agent capabilities won't be available.

## Supported resource types

Currently, Agents (preview) in Azure Copilot supports full end-to-end migration for VMware workloads. You can also get help with migration tasks related to Hyper-V and physical servers discovered in Azure Migrate.

## Migration sample prompts

Here are a few examples of the kinds of prompts you can use to get help with migratoin tasks. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. When using these kinds of prompts, be sure to enable agent mode by selecting the icon in the chat window.

:::image type="content" source="media/agent-preview/azure-copilot-agent-mode-active.png" alt-text="Screenshot showing agent mode (preview) enabled in Azure Copilot.":::

- "How should I plan moving VMware workloads to Azure?"
- "I want to move my servers and PGSQL database to Azure. How should I proceed?"
- "What discovery methods are available?"
- "Use RVTools to discover VMware workloads."
- "I finished deploying the appliance and discovering data. Summarize the discovered inventory."
- "What are the other options for moving workloads to Azure?"
- "Provide a ROI comparison between lift-and-shift and modernization."
- "Show servers that have the support status `Out of support`"
- "Tag these servers as `upgraderequired:yes`"
- "Assign tag `application:ZavaOrderProcessingApp` to the `vm-web-tier` and `vm-app-tier` servers."
- "Assign tag `application:ZavaOrderProcessingApp` to the PGSQL database with name `WIN-PG-04` and Version `17.5`
- "List all the workloads tagged with `application:ZavaOrderProcessingApp`
- "What is the cloud readiness of my inventory tagged with `application:ZavaOrderProcessingApp`?
- "Show the readiness summary of my assessment."
- "Generate a new platform landing zone."
- "Based on this plan, how can I execute the actual migration now?"

## Current considerations and limitations

Keep in mind the following considerations and limitations when working with migration in Agents (preview) in Azure Copilot.

- While Azure Copilot can provide guidance, create plans, and apply tags, it can't perform actual migration actions such as server replication on your behalf.
- Migration tasks that require Azure Migrate to interact with other Azure services, such as Azure Backup and Azure Site Recovery, aren't currently supported.
