---
title: Set up the Data Query for Agents and Tools with Foundry Local Chat Solution
description: "Learn how to set up your data query with Agents and Tools with Foundry Local to configure model settings and create effective AI-driven chat solutions."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/27/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to set up the data query and configure model settings for the Agents and Tools with Foundry Local chat solution, so that I can create effective AI-driven chat experiences tailored to my application's requirements.
---

# Set up the data query for chat solution in Agents and Tools with Foundry Local

Configure data queries and model settings for your Agents and Tools with Foundry Local chat solution to optimize your chat results. Adjust search types, tune model parameters, and refine your chat experience in the Agents and Tools with Foundry Local developer portal.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- Make sure you are in **Knowledge-based** chat mode.
- Review [Search types in Agents and Tools with Foundry Local](search-types.md) to understand the available search types and when to use them.
- Review [Knowledge layer configuration](knowledge-layer-overview.md) to plan for data ingestion and choose the right prompt and model parameters.
- [Add data source for the chat solution in Agents and Tools with Foundry Local](add-data-source.md)
- To access the developer portal, you must have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Configure model settings

To get started, configure the model settings.

1. Go to the local portal using the domain name provided at deployment and app registration. For example, `https://arcrag.contoso.com`.
1. Sign in with developer credentials that have both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned. If you have the right access configured, you're automatically  redirected to the developer portal.
1. Select the **Chat** tab to get to the **Chat playground**.
1. In the **Data inferencing** pane, select the **Search parameters** section.

1. Select the **Type of search**. All four search types (hybrid text, text, vector, and hybrid multimodal) are available. For guidance on choosing a search type, see [Search types in Agents and Tools with Foundry Local](search-types.md).

     :::image type="content" source="media/set-up-data-query/search-types.png" alt-text="Screenshot of the available search types under the search parameters.":::

1. Under **Model parameters**, at the top of the **Data inferencing** pane, adjust the model parameters for **Temperature** and **Top P** as needed.
   1. Under **Parameters**, adjust the model parameters for **Top-N documents** and **Text strictness** as needed.
   
   :::image type="content" source="media/set-up-data-query/data-inferencing.png" alt-text="Screenshot of the data inferencing section of the chat window that allows you to change the model parameters.":::
   1. Review and update the **System prompt** as needed for your solution.

1. Any changes that you make are applied when you submit a new question in the chat.

## Test chat results

Next, test the chat endpoint.

1. In the chat window, enter a question that uses a simple question and answer format. Queries that require summarization across multiple documents might not return accurate answers.

   :::image type="content" source="media/set-up-data-query/test-chat.png" alt-text="Screenshot of the chat playground with a question in the chat window." lightbox="media/set-up-data-query/test-chat.png":::

   Be aware that with Agents and Tools with Foundry Local extension version `0.1.5` and later each question is answered based on retrieved content only. The answer doesn't include the context of the chat history. Chat history isn't saved between questions. Treat each question as a new chat.
1. (Optional) To see how the language model responds without using your ingested data, switch the chat mode to **Model-only** and enter your question. Switch back to **Knowledge-based** chat to keep refining your solution with your ingested data.
1. (Optional) [Test the end user experience by using the chat solution app for Agents and Tools with Foundry Local](test-end-user-app.md).

## View details to refine settings

Use the chat response details to analyze and fine-tune your model and search parameters to optimize your chat responses.

1. Under the chat response, select **View details**.

   :::image type="content" source="media/set-up-data-query/test-chat-view-details-option.png" alt-text="Screenshot that shows the view details option underneath the chat response." lightbox="media/set-up-data-query/test-chat-view-details-option.png":::
1. Use the chat details to understand the impact of the inferencing parameters on the language model's response to your question. 

   | Field            | Description                                                                                     |
   |------------------------|-------------------------------------------------------------------------------------------------|
   | LLM response           | Response from the large language model (LLM) for the corresponding question.   |
   | User question          | Question asked by user.     |
   |Search type|The method used to find relevant information for your question, such as hybrid, text, vector, or hybrid multimodal search.|
   | Parameters             | Parameters that are used to search content and generate LLM response.              |
   | System prompt          | The custom instructions set by the developer to guide the language model\u2019s responses.    |
   | Reranked chunks        | Shows search IDs by reranking score.  |
   | LLM Input chunks       | Relevant chunks passed to LLM as retrieved content; the chunks are selected based on text strictness and image strictness.  |
   | Search details         | Shows search details.    |
   | Results from text search | Results from textual search for a query; each result shows reranking score, search distance, text, file path, chunk ID, and last modified date. |
   | Results from vector search | Results from semantic search for a query; each result shows reranking score, search distance, text, file path, chunk ID, and last modified date. |
   | Results from image search | Results from image search for a query, each result shows reranking score, file path, last modified date. |

   :::image type="content" source="media/set-up-data-query/view-details.png" alt-text="Screenshot that shows the details of the chat response including the search type, parameters, and more." lightbox="media/set-up-data-query/view-details.png":::

1. To analyze the **Details**, select **Copy** to paste a JSON version of the text into a text editor.
1. Tune the inferencing parameters to get the type of responses that you want for your ingested data.

## Get the API endpoint

When you're satisfied with the solution, select on **View the endpoint** to get the API endpoint to use in your downstream applications.

## Next step

> [!div class="nextstepaction"]
> [Test the chat solution](test-end-user-app.md)

## Related content

-[Search types in Agents and Tools with Foundry Local](search-types.md)
- [Knowledge layer configuration](knowledge-layer-overview.md)
- [Add data source for chat solution in Agents and Tools with Foundry Local](add-data-source.md)
