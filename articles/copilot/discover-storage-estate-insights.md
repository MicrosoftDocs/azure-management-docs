---
title: Discover insights about your Azure Storage estate with Azure Storage Discovery using Microsoft Copilot in Azure
description: Learn how Microsoft Copilot in Azure, in combination with Azure Storage Discovery, uncovers insights about your storage estate not available anywhere else.
ms.date: 10/01/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: fauhse
author: fauhse
# Customer intent: "As a storage administrator or IT leader, I want to utilize an AI assistant to understand my storage estate, improve the security posture and reduce costs of storage accounts, so that I can efficiently manage and optimize my cloud storage resources."
---

# Discover insights about your Azure Storage estate with Azure Storage Discovery using Microsoft Copilot in Azure

[Azure Storage Discovery](/azure/storage-discovery/overview) is a managed service that gives you enterprise-wide visibility into your Azure Blob Storage data estate. It helps you track how your data is growing, uncover opportunities for cost optimization, and check if your storage configurations follow security best practices. With Azure Storage Discovery, you can analyze thousands of storage accounts across subscriptions and regions from one place, using prebuilt reports and interactive dashboards. 

Microsoft Copilot in Azure integration takes Storage Discovery a step further by letting you ask questions about your storage resources and data in natural language. Get instant, visual answers. Instead of writing Kusto queries or combing through logs, you can chat with Copilot in Azure to explore your storage insights. Copilot in Azure understands your questions, retrieves the relevant aggregated data from Storage Discovery, and presents results as dynamic charts or tables directly in the Azure portal. This conversational approach makes it easier for anyone – from IT managers to storage admins – to gain insights and make data-driven decisions.

## Interacting with Copilot in Azure in Storage Discovery

Retrieving storage insights with Copilot in Azure requires a deployment of the [Storage Discovery service](/azure/storage-discovery/deployment-planning). Deploying the service means [creating a Discovery workspace resource](/azure/storage-discovery/create-workspace). Creating a workspace starts the data aggregation required for Copilot in Azure to answer certain questions related to storage insights.

To start using Copilot in Azure, navigate to your Azure Storage Discovery workspace in the Azure portal. On the workspace overview or insights page, look for the Copilot icon, and select it to open the Copilot chat pane. You can also select the Copilot icon at the top of the portal.

Example: You might ask "Show me the trend of my storage usage over time." Copilot in Azure interprets your question, queries the Storage Discovery data, and responds with the requested insight. You can request to receive the answer as a table or any of the common types of charts.

> [!TIP]
> Asking Copilot in Azure about your storage estate requires you to select a Storage Discovery workspace. You need to deploy this resource first before Copilot has the data to answer your questions.

## Identify cost optimization opportunities

Cost optimization is often the first concern when managing a large storage estate. Azure Storage Discovery’s reports already show you metrics like total capacity and data growth trends. With Copilot in Azure, you can dig deeper or customize these insights with simple questions.

**Analyze storage growth trends:** Ask Copilot in Azure about how your stored data is trending over time. For example: "**How is the storage size trending over the past month by region?**" Copilot in Azure returns a line chart plotting the total data size in your storage accounts over the last month, broken down by region. This visualization helps you see which regions are contributing most to growth or if any region’s usage is flattening or spiking.

:::image type="content" source="media/discover-storage-estate-insights/storage-discovery-copilot-trend.png" alt-text="Screenshot of Copilot in Azure showing a storage trend chart.":::
<!-- Source document: Page 3 -->

**Find under-utilized storage (cold data):** Storing much data in a hot tier that isn’t being accessed could waste money. You can have Copilot identify storage accounts with large capacity but low activity. For example: "**Provide a table of storage accounts that have the least transactions and are above 1 TiB in size.**" This prompt asks for a list of large storage accounts with minimal access. Copilot returns a table with storage accounts meeting those conditions. The result table includes columns like the account name, data size, and the number of transactions. You can immediately spot accounts that each hold over 1 TiB of data but handle few transactions. Such accounts might be good candidates to move to a cooler access tier (like Cool or Archive) to save costs.

:::image type="content" source="media/discover-storage-estate-insights/storage-discovery-copilot-cold-table.png" alt-text="Screenshot of Copilot in Azure showing a table of storage accounts and their transaction counts.":::
<!-- Source document: Page 4 -->

**View data distribution by access tier:** To act on cold data, you might want to know how your data is currently distributed across access tiers (Hot, Cool, Cold, Archive). You can ask a question like: "**Show me a distribution of blob count by blob access tier.**" In response, Copilot can provide a bar chart (or pie chart) breaking down how many blobs are in each access tier across your estate. You might discover that a large number of blobs are in the Hot access tier even though they're rarely accessed. This insight is an opportunity to use [lifecycle management policies](/azure/storage/blobs/lifecycle-management-policy-configure) or [Azure Storage Actions](/azure/storage-actions/overview) to move data automatically to cheaper tiers over time.

:::image type="content" source="media/discover-storage-estate-insights/storage-discovery-copilot-tier-chart.png" alt-text="Screenshot of Copilot in Azure showing a bar chart with blob counts per access tier.":::
<!-- Source document: Page 5 -->

Using Copilot in Azure can quickly surface where your storage costs are coming from. It turns what could be a complex query (combining capacity and transaction data across many accounts) into an easy question and answer. The visual results make it straightforward to decide on next steps, such as enabling rules to tier down infrequently used data and reduce costs.

## Assess security configurations & compliance

Another key value of Storage Discovery is surfacing potential security risks or misconfigurations in your storage accounts. For example, it can tell you which accounts allow anonymous access or still use access keys for authentication. With Copilot in Azure, you can interactively query these security insights and even get summaries or breakdowns by region or other dimensions.

**Detect usage of shared access keys:** Microsoft recommends using Microsoft Entra ID with managed identities for Azure Storage authentication whenever possible, rather than shared keys. To ensure compliance, you can ask Copilot in Azure something like: "**How many of my storage accounts have `shared access keys` enabled?**" The response might include a simple count or a list. You can refine the question to get a regional breakdown: "**Show me a pie chart of my storage accounts with shared key enabled, by region.**" Copilot in Azure produces a pie chart where each slice represents a region, showing what portion of accounts in that region still allows shared key authentication. 

:::image type="content" source="media/discover-storage-estate-insights/storage-discovery-copilot-shared-key-pie.png" alt-text="Screenshot of Copilot in Azure showing a pie chart with shared access key enabled storage account counts per region.":::
<!-- Source document: Page 6 -->

This insight is useful when one region has a large slice, which means many accounts there still use shared keys. You might prioritize those regions for rolling out Entra ID authentication. The result helps focus your efforts on the biggest problem areas first.

**Check other security settings:** You can query settings like `anonymous public access` or `minimum required TLS version`. For example: "**List storage accounts that allow anonymous public read access.**" Copilot in Azure returns a list of any accounts with that setting enabled, so you can review if this setting is needed on those accounts. Or you might ask: "**Which storage accounts aren't enforcing encryption at rest?**" All Azure storage accounts have encryption at rest enabled by default, but if any account was misconfigured, Copilot in Azure highlights that.

## Manage data redundancy and resiliency

Azure Storage offers several redundancy options (LRS, ZRS, GRS, etc.). Storage Discovery’s reports show the distribution of accounts across these redundancy settings. Copilot in Azure can help you analyze redundancy configurations and even consider potential optimizations:

**View redundancy distribution:** You might ask: "**Show me a distribution of my storage account count by redundancy option.**" Copilot in Azure returns a chart with each redundancy level on the X-axis and the number of storage accounts on the Y-axis. This representation quickly tells you how many accounts use Locally Redundant Storage (LRS), Zone Redundant (ZRS), Geo-Redundant (GRS), or other options.

:::image type="content" source="media/discover-storage-estate-insights/storage-discovery-copilot-redundancy-bar.png" alt-text="Screenshot of Copilot in Azure showing a bar chart with storage account counts per redundancy type.":::
<!-- Source document: Page 7 -->

Suppose the chart reveals that 80% of accounts are LRS and only a few use ZRS. If these accounts support critical workloads, then they all benefit from ZRS redundancy. This insight lets you verify if those resources are configured correctly. Alternatively, if many accounts are using ZRS but don’t actually need that level of resiliency, you might consider downgrading some to LRS to save costs. Copilot in Azure's visualization highlights potential misalignments between your redundancy choices and your resiliency needs.

**Ask for specific insights:** You can combine filters in your question. For example, ask "**Which storage accounts are using GRS in the West US region?**" and Copilot in Azure lists them. Or ask "**Do I have any storage accounts with only LRS in East US 2?**" to find where you might want to upgrade redundancy.

By querying redundancy information on-demand, you ensure your storage accounts’ configuration meets your disaster recovery and availability targets. Copilot in Azure basically turns the raw configuration data into an actionable summary for you.

## Tips for getting the best results with storage insight queries

**Be specific in your prompts:** While Copilot in Azure can handle natural language, phrasing your question clearly yields better answers. Include what insight you want and any filter (time range, region, tier, etc.) in your question. For example: "How is storage usage changing?" is okay, but "How is storage size trending over the past 30 days by region?" is more likely to produce the detailed chart you want.

**Request a visualization if helpful:** Copilot in Azure decides the format of the answer, such as a table, chart, or just text. You can guide it by specifying a format. For instance, adding "**show me a pie chart of…**" or "**provide a table of…**" to your prompt usually influences Copilot in Azure to return an answer in that format.

**Use follow-up questions:** Copilot in Azure remembers the context within the session. You can ask a broad question first, then follow up with a more specific one without restating everything. For example: "**How many storage accounts do we have?**" The response might be "200 accounts across 5 regions." Then you ask: "**How many are in North Europe?**" Copilot in Azure knows you’re still talking about storage account count, and filters the response to answer in that context.

**Understand limitations:** The Storage Discovery features of Copilot in Azure focuses on analytics reporting, not operational tasks. You can't use Copilot in Azure to create storage resources or change resource configuration.

## How Copilot in Azure enhances the Storage Discovery experience

Copilot in Azure provides on-demand insights beyond the fixed dashboards. If a certain metric or combination isn’t directly shown in a Discovery workspace report, you can ask Copilot in Azure to provide it.

Copilot in Azure democratizes access to insights. Team members who are less familiar with Azure can still retrieve insights by asking for them, without requiring complex queries or scripting.

The interactive nature of Copilot in Azure encourages exploration. You might start with one question and, seeing the result, ask a follow-up or drill-down question. This conversational analysis can lead to findings you might skip if you had to write complex queries each time.

Copilot in Azure answers are based on the same underlying data you see in the Azure portal. You can cross-verify any Copilot answer with the relevant report in the Storage Discovery workspace.

In summary, Copilot in Azure provides a powerful, user-friendly way to access Azure Storage Discovery. It brings your data to your fingertips, whether you’re investigating cost spikes, tightening security, or planning backup strategies. By using natural language and AI-driven visualizations, you can derive more value from your storage insights with less effort.

## Next steps

After understanding the opportunities provided by Storage Discovery and Copilot in Azure, it's a good idea to get more familiar with the Storage Discovery service.

- [Get an overview of the Discovery service](/azure/storage-discovery/overview)
- [Plan your Storage Discovery deployment](/azure/storage-discovery/deployment-planning)
- [Create a Storage Discovery workspace](/azure/storage-discovery/create-workspace)
