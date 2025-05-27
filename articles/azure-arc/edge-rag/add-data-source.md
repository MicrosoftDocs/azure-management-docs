---
title: Add Data Source for Edge RAG Chat Solution
description: "Learn how to add and manage data sources for Edge RAG chat solutions, including setup, and ingestion processes."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/13/2025

#CustomerIntent: As a developer or data scientist, I want to add a data source to Azure AI Search so that I can enable intelligent search capabilities across my hybrid and multiloud environments.

# Customer intent: As a developer, I want to add a data source for my Edge RAG chat solution, so that I can effectively set up and manage data ingestion for intelligent search across hybrid and multicloud environments.
---
# Add data source for Edge RAG Preview, enabled by Azure Arc

Start building your chat solution by setting up the data you want to ingest.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, review [Building chat solution overview for Edge RAG](build-chat-solution-overview.md) to plan for data ingestion and choose the right prompt and model parameters. Also review [supported data sources](requirements.md#supported-data-sources).

## Set up data ingestion

To get started, create a data source by using the local developer portal.

1. Go to the developer portal using the domain name provided at deployment and app registration. For example: `https://arcrag.contoso.com`.
1. Sign in with developer credentials (with both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned"). If you have the right access configured, you're automatically  redirected to the developer portal.
1. Select **Get Started**.
1. Go to the **Data** tab.
1. Select **Add a data source**.
1. On the **Source data** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Name | Name for the data ingestion |
    | Data source (read only) | Network share |
    | Network File Share | Path to your network file server (NFS) share |
    | User ID | NFS user ID |
    | Group ID | NFS group ID |

1. Select **Next**.
1. On the **Vector index** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Schedule updates | Frequency at which your data is synced for updates |
    | Chunk size | Select the appropriate chunk size |
    | Chunk overlap |Select the appropriate chunk overlap |

    :::image type="content" source="media/add-data-source/configure-index.png" alt-text="Screenshot of the vector index tab where you configure chunk size and overlap.":::

1. Select **Next**.
1. On the **Review and finish** tab, review your configurations.
1. When you're satisfied, select **Create**.

## Related content

- [Building chat solution overview for Edge RAG](build-chat-solution-overview.md)
