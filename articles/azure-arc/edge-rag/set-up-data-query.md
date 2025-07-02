---
title: Set up the Data Query for Edge RAG Chat Solution
description: "Learn how to set up your data query with Edge RAG to configure model settings and create effective AI-driven chat solutions."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 07/01/2025
ms.subservice: edge-rag
#CustomerIntent: As a developer or data scientist, I want to learn how to use prompt engineering with Azure AI Search so that I can create more effective and accurate AI-driven search experiences for my applications.
ms.custom:
  - build-2025
---

# Set up the data query for chat solution in Edge RAG Preview, enabled by Azure Arc

Configure data queries and model settings for your Edge RAG chat solution to optimize your chat results. Adjust search types, tune model parameters, and refine your chat experience in the Edge RAG developer portal.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- Review [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md) to plan for data ingestion and choose the right prompt and model parameters.
- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- To access to the developer portal, you must have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Configure model settings 

To get started, configure the model settings.

1. Go to the local portal using the domain name provided at deployment and app registration. For example, `https://arcrag.contoso.com`.
1. Sign in with developer credentials that have both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned. If you have the right access configured, you're automatically  redirected to the developer portal.
1. Select **Get Started**.
1. Select the **Chat** tab to get to the **Chat playground**.
1. In the **Data inferencing** section, select the **Type of search**. 


   | Search type              | Description                 |
   |--------------------------|-----------------------------|
   | Hybrid text search       | Search that combines keyword (text) search and vector (contextual) search. |
   | Text search              | Search that queries exact words or phrases in documents.                   |
   | Vector search            | Search that queries contextual similarity rather than exact keyword matching. |
   | Hybrid multimodal search | Search that combines multiple modalities, like text and image, simultaneously.   |

1. Change the model parameters for **Temperature**, **Top-N**, **Top-P**, and others as needed.

   :::image type="content" source="media/set-up-data-query/data-inferencing.png" alt-text="Screenshot of the data inferencing section of the chat window that allows you to change the model parameters.":::

1. Review and update the system prompt as appropriate for your solution.
1. Any changes that you make are applied when you submit a new question in the chat.

## Test chat results

Next, test the chat endpoint.

1. In the chat window, enter a question that uses a simple question and answer format. Queries that require summarization across multiple documents might not return accurate answers.

   :::image type="content" source="media/set-up-data-query/test-chat.png" alt-text="Screenshot of the chat playground with a question in the chat window." lightbox="media/set-up-data-query/test-chat.png":::

   Be aware that with Edge RAG extension version `0.1.5` each question is answered based on retrieved content only and doesn't include the context of the chat history. Chat history is not saved between questions. Treat each question as a new chat.
1. (Optional) [Test the end user experience by using the chat solution app for Edge RAG](test-end-user-app.md).

## View details to refine settings

Use the chat response details to analyze and fine-tune your model and search parameters to optimize your chat responses.

1. Under the chat response, select **View details**. 

   :::image type="content" source="media/set-up-data-query/test-chat-view-details-option.png" alt-text="Screenshot that shows the view details option underneath the chat response." lightbox="media/set-up-data-query/test-chat-view-details-option.png":::
1. Use the chat details to understand the impact of the inferencing parameters on the language model's response to your question. 

   | Field            | Description                                                                                     |
   |------------------------|-------------------------------------------------------------------------------------------------|
   | LLM response           | Response from the large language model (LLM) for the corresponding question.   |
   | User question          | Question asked by user.     |
   | Parameters             | Parameters that are used to search content and generate LLM response.              |
   | System prompt          | System prompt input set by Developer.    |
   | Reranked chunks        | Shows search IDs by reranking score.   |
   | LLM Input chunks       | Relevant chunks passed to LLM as retrieved content; the chunks are selected based on text strictness and image strictness. |
   | Search details         | Shows search details.    |
   | Results from text search | Results from textual search for a query; each result shows reranking score, search distance, text, file path, chunk ID, and last modified date. |
   | Results from vector search | Results from semantic search for a query; each result shows reranking score, search distance, text, file path, chunk ID, and last modified date. |
   | Results from image search | Results from image search for a query, each result shows reranking score, file path, last modified date. |

   :::image type="content" source="media/set-up-data-query/view-details.png" alt-text="Screenshot that shows the details of the chat response including the search type, parameters, and more." lightbox="media/set-up-data-query/view-details.png":::

1. To analyze the **Details**, select **Copy** to paste a JSON version of the text into a text editor.
1. Tune the inferencing parameters to get the type of responses that you want for your ingested data.

## Get the API endpoint

When you're satisfied with the solution, select on **View the endpoint** to get the API endpoint to use in your downstream applications.

## Related content

- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
- [Add data source for chat solution in Edge RAG](add-data-source.md)
