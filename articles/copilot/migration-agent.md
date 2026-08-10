---
title: Azure Copilot Migration Agent
description: The Azure Copilot Migration Agent helps you plan and execute migrations to Azure and get guidance on assessment, compatibility, and step-by-step migration paths.
ms.date:  06/22/2026
ms.service: azure-copilot
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use the Migration Agent, so that I can plan and execute migrations to Azure."
---

# Azure Copilot Migration Agent

The Azure Copilot Migration Agent helps you perform [migration tasks](/azure/migrate/migrate-services-overview). You can get help with planning, assessing, strategizing, and moving workloads to Azure.

The Migration Agent works to understand your migration goals, such as faster migration, modernization to PaaS, and regional targets. It then creates artifacts such as [business cases](/azure/migrate/concepts-business-case-calculation) and [assessments](/azure/migrate/concepts-overview) using your existing context. You can also get help setting up [landing zones](/azure/cloud-adoption-framework/ready/landing-zone/), comparing alternative options, and resolving blocking issues.

> [!IMPORTANT]
> Administrators can [enable or disable access to the Azure Copilot Migration Agent](manage-access.md#manage-user-access-to-azure-copilot-and-agents) and other [Azure Copilot agents](agents.md). If you don't see the Migration Agent as an option when starting a new chat, check with your administrator.

## Supported resource types

Currently, the Azure Copilot Migration Agent supports full end-to-end migration for VMware workloads. You can also get help with migration tasks related to Hyper-V and physical servers discovered in Azure Migrate.

## Migration sample prompts

Here are a few examples of the kinds of prompts you can use with the Migration Agent. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

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

Keep in mind the following considerations and limitations when working with the Azure Copilot Migration Agent.

- While the Migration Agent can provide guidance, create plans, and apply tags, it can't perform actual migration actions such as server replication on your behalf.
- Migration tasks that require Azure Migrate to interact with other Azure services, such as Azure Backup and Azure Site Recovery, aren't currently supported.

## Related content

- Learn more about [using the migration agent capabilities](/azure/migrate/azure-copilot-migration-agent) and ways to [plan and analyze VMware migrations](/azure/migrate/how-to-plan-analyze-migration-with-agent).
