---

title: Delete data for Edge RAG Chat Solution
description: "Learn how to delete data in Edge RAG chat solutions, including removing all data ingestions and evaluation data from the vector database."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 06/05/2025
ms.subservice: edge-rag
#CustomerIntent:  As a developer using Edge RAG, I want to delete all data from the vector database so that I can remove all data ingestions and evaluation data for a clean reset of the index.
ms.custom:
  - build-2025
---
# Delete data in Edge RAG Preview, enabled by Azure Arc

Edge RAG supports only one index, which contains the embeddings for all the data ingested and any evaluation data. You can't delete individual data ingestions or evaluations from the **Data** tab.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

To access to the developer portal, you must have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Delete data from the vector database

If you want to delete the data in your index, including all the individual data ingestions and any evaluation data, complete the following the steps:

1. Go to the developer portal using the domain name provided at deployment and app registration. For example, `https://arcrag.contoso.com`.
1. Sign in with developer credentials that have both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned.
1. Go to **Data** tab.
1. Select on the **Delete** button on the toolbar
1. Confirm if you want to delete all the data from the vector index.

This step clears all the data in the vector database and empties the data ingestions history table.

If there's a data ingestion job running or pending, you see an error message and aren't able to delete the data in the index. Let those processes complete before trying again.

## Related content

- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)