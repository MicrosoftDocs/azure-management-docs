---

title: Delete data for Edge RAG Chat Solution
description: "Learn how to delete data in Edge RAG chat solutions, including removing all data ingestions and evaluation data from the vector database."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/13/2025

#CustomerIntent:  As a developer using Edge RAG, I want to delete all data from the vector database so that I can remove all data ingestions and evaluation data for a clean reset of the index.
ms.custom:
  - build-2025
---
# Delete data in Edge RAG Preview, enabled by Azure Arc

Edge RAG supports only one index, which contains the embeddings for all the data ingested and any evaluation data. You can't delete individual data ingestions or evaluations from the **Data** tab.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Delete data from the vector database

If you want to delete the data in your index, including all the individual data ingestions and any evaluation data, complete the following the steps:

1. Go to **Data** tab on the developer portal.
1. Select on the **Delete** button on the toolbar
1. Confirm if you want to delete all the data from the vector index.

This step clears all the data in the vector database and empties the data ingestions history table.

If there's a data ingestion job running or pending, you see an error message and aren't able to delete the data in the index. Let those processes complete before trying again.

## Related content

- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- [Building chat solution overview for Edge RAG](build-chat-solution-overview.md)