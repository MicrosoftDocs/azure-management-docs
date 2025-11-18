---
title: Add Data Source for Edge RAG Chat Solution
description: "Learn how to add and manage data sources for Edge RAG chat solutions, including setup, and ingestion processes."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 11/11/2025
ms.subservice: edge-rag
#CustomerIntent: As a developer or data scientist, I want to add a data source to Azure AI Search so that I can enable intelligent search capabilities across my hybrid and multiloud environments.
ms.custom:
  - build-2025
---
# Add data source for Edge RAG Preview, enabled by Azure Arc

Add and configure a data source for your Edge RAG chat solution by using the developer portal. Follow the step-by-step instructions to set up data ingestion, and define indexing parameters.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- Review the following articles:
  - [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
  - [Supported data sources](requirements.md#supported-data-sources)
  - [Advanced data parsing for Edge RAG](advanced-data-parsing.md)
- To access to the developer portal, you must have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Set up data ingestion

To get started, create a data source by using the local developer portal.

1. Go to the developer portal using the domain name provided at deployment and app registration. For example: `https://arcrag.contoso.com`.
1. Sign in with developer credentials that have both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned. If you have the right access configured, you're automatically  redirected to the developer portal.
1. Select **Get Started**.
1. Go to the **Data** tab.
1. Select **Add a data source**.
1. On the **Source data** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Name | Name for the data ingestion |
    | Ingestion type|Basic or advanced document parsing|
    | Data source (read only) | Network share |
    | Network File Share | Path to your network file server (NFS) share |
    | User ID | NFS user ID |
    | Group ID | NFS group ID |

   :::image type="content" source="media/add-data-source/source-data.png" alt-text="Screenshot of the source data tab where you define the ingestion type and data source.":::

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

- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
