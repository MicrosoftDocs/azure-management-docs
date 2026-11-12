---
title: Set up the Data Query for Agentic Retrieval in Agents and Tools with Foundry Local
description: "Learn how to set up the data query for indexed sources in Agentic Retrieval, and configure model settings for grounded chat answers in Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to set up the data query and configure model settings for indexed sources in Agentic Retrieval, so that I can create effective AI-driven chat experiences tailored to my application's requirements.
---

# Set up the data query for Agentic Retrieval in Agents and Tools with Foundry Local

Configure data queries and model settings for indexed sources in Agentic Retrieval to optimize your chat results. Adjust search types, tune model parameters, and refine your results in the Agents and Tools with Foundry Local developer portal.

## Prerequisites

Before you begin:

- Ensure you're in **Knowledge-base** chat mode.
- Review [Search types](search-types.md) to understand the available search types and when to use them.
- Review the [Agentic Retrieval overview](knowledge-layer-overview.md) to plan for data ingestion and choose the right prompt and model parameters.
- [Add a data source](add-data-source.md).
- To access the developer portal, you need the `EdgeRAGDeveloper` app role in Microsoft Entra ID. For the Local chat UI, assign **Reader** and **Cognitive Services OpenAI User** at the extension scope. In disconnected deployments, use **Contributor**. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).
- Local ingestion and the developer portal are available only when `indexedSourcesEnabled=true`.

## Configure inference model settings

To get started, configure the inference model settings in the developer portal.

1. Go to the local portal using the domain name provided at deployment and app registration. For example, `https://arcrag.contoso.com`.
1. Sign in with developer credentials. Sign-in opens **Chat**, not the developer portal.
1. Open `https://<domain>/developer`, using your deployment domain. This address opens the **Data** page.
1. Select the **Chat** tab to go to the **Chat playground**.
1. In the **Data inferencing** pane, under **Model parameters**, adjust the model parameters for **Temperature (low=precise, high=creative)** and **Top-P (nucleus sampling)** as needed.

   :::image type="content" source="media/set-up-data-query/model-parameters.png" alt-text="Screenshot of the data inferencing section of the chat window that allows you to change the model parameters." lightbox="media/set-up-data-query/model-parameters.png":::

1. Select the **Search parameters** section.
1. Select the **Type of search** you want to use.

    | Search type | Description |
    |---|---|
      | Hybrid text search | Combines keyword (text) search and vector (contextual) search. |
      | Text search | Looks for exact words or phrases in documents. |
      | Vector search | Looks for contextual similarity rather than exact matching keywords. |
      | Hybrid multimodal search (Preview, BYOM-only) | Combines multiple modalities of text and image simultaneously. Requires GPU mode (`gpu_enabled=true` so image embedding collections are provisioned) and, in the developer portal, a configured `byom.apiEndpoint` with a vision-capable model. |

1. Under **Parameters**, adjust the search parameters for **Top-N documents**, **Text strictness**, and **Image strictness** (for hybrid multimodal search) as needed.

   :::image type="content" source="media/set-up-data-query/search-parameters.png" alt-text="Screenshot of the types of search, top-N documents, and text strictness fields under the search parameters." lightbox="media/set-up-data-query/search-parameters.png":::
1. Select the **System prompt** section. Review and update the prompt as needed for your solution.
1. Your changes take effect when you submit a new question in the chat.

## Test chat results

Next, test the chat endpoint.

1. In the chat window, enter a question that uses a simple question and answer format. Queries that require summarization across multiple documents might not return accurate answers.

   Be aware that with Agents and Tools with Foundry Local extension version 0.1.5 and later, each question is answered based on retrieved content only. The answer doesn't include the context of the chat history. Chat history isn't saved between questions. Treat each question as a new chat.

1. (Optional) To see how the language model responds without using your ingested data, switch the chat mode to **Model-only** and enter your question. Switch back to **Knowledge-base** chat to keep refining your solution with your ingested data.
1. (Optional) After you create an enabled agent, [test the end-user experience in the Local chat UI (`/chat`)](test-end-user-app.md). **Chat playground** (`/developer/chat`) tests classic RAG over `edgeragapp` inside the developer portal only and doesn't configure the Local chat UI.

## View details to refine settings

Use the chat response details to analyze and fine-tune your model and search parameters to optimize your chat responses.

1. Under the chat response, select **View details**.
1. Use the chat details to understand the impact of the inferencing parameters on the language model's response to your question. 

   | Field            | Description                                                                                     |
   |------------------------|-------------------------------------------------------------------------------------------------|
   | LLM response           | Response from the large language model (LLM) for the corresponding question.   |
   | User question          | Question asked by user.     |
   |Search type|The method used to find relevant information for your question, such as hybrid, text, vector, or hybrid multimodal search.|
   | Parameters             | Parameters that are used to search content and generate LLM response.              |
   | System prompt          | The custom instructions set by the developer to guide the language model's responses.    |
   | Reranked chunks        | Shows search IDs by reranking score.  |
   | LLM input chunks       | Relevant chunks passed to LLM as retrieved content; the chunks are selected based on text strictness and image strictness.  |
   | Search details         | Shows search details.    |
   | Results from Text Search | Results from textual search for a query; each result shows reranking score, search distance, text, file path, chunk ID, page numbers, and updated at (`Updated at`). |
   | Results from Vector Search | Results from semantic search for a query; each result shows reranking score, search distance, text, file path, chunk ID, page numbers, and updated at (`Updated at`). |
   | Results from Image Search | Results from image search for a query; each result shows reranking score, file path, page numbers, and updated at (`Updated at`). |

1. To analyze the **Details**, select **Copy** to paste a JSON version of the text into a text editor.
1. Tune the inferencing parameters to get the type of responses that you want for your ingested data.

## Get the API endpoint

In the developer portal, **View the endpoint** displays the deprecated classic inference URL (`POST /chat/completions`). This endpoint is restricted to the developer portal browser origin and returns `403 Forbidden` for external callers.

To integrate line-of-business applications with your indexed data, use Agent Service (`/openai/v1/responses` and `/openai/v1/conversations`) or Indexed Sources MCP (`/mcp`). See [Run agents with the Responses API](run-agents-responses-api.md) and [MCP Server overview](mcp-server-overview.md).

## Next step

The **Chat playground** (`/developer/chat`) tests classic RAG over `edgeragapp` in the developer portal and doesn't configure the Local chat UI (`/chat`). To test the Local chat UI, create an enabled agent and then open `/chat`:

> [!div class="nextstepaction"]
> [Test the Local chat UI](test-end-user-app.md)

## Related content

- [Search types in Agents and Tools with Foundry Local](search-types.md)
- [Agentic Retrieval overview](knowledge-layer-overview.md)
- [Add a data source for Agents and Tools with Foundry Local](add-data-source.md)
