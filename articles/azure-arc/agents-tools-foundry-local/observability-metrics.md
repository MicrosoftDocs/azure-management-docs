---
title: Monitor Agents and Tools with Foundry Local with metrics
description: "Learn which metrics track API performance, ingestion rates, and processing times in Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: reference #Don't change
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: As a cloud administrator or DevOps engineer, I want to use the available metrics for Agents and Tools with Foundry Local to monitor API performance, ingestion rates, and processing times so that I can maintain optimal system performance and quickly address any issues in my infrastructure.
---

# Metrics for Agents and Tools with Foundry Local

The following table lists Azure Monitor metric display names for **Agents and Tools with Foundry Local**. Availability depends on the installed version and features in use. Select the metrics listed for your extension in Azure Monitor or Azure Managed Grafana.

## How to read these metrics

- Each chart shows the metrics of the extension resource that you select in Azure Monitor or Azure Managed Grafana. To compare clusters, add a chart or metric for each extension resource.
- **ResultType** shows whether an operation succeeded or failed: `Success`, `Failure`, or, for some metrics, `CustomerError`. **Result** is `Success` or a short error category. It never contains exception text.
- An agent response can fail even when the HTTP request returns `200`. To check the outcome of a response, read its `status` in the Responses API.
- For duration metrics, **Count** is the number of measurements, **Sum** is the total time, and **Average** is the time per measurement. Most duration metrics use milliseconds. **Call LLM Total Time in Seconds** uses seconds.
- To filter or split charts, use low-cardinality dimensions such as **AppId**, **Operation**, **ResultType**, and **Result**. Don't split charts by identifiers such as agent IDs. Agent, agent version, and response identifiers aren't recommended metric dimensions. To investigate a specific agent, agent version, response, or request, or to follow a request across services, use [application logs](observability.md#check-application-logs).

## Available metrics

| Metric name | Description |
| --- | --- |
| Agent API Duration | Time taken to complete agent operations, in milliseconds |
| Agent Tool Invocations | Number of tool calls |
| Knowledge API Duration | Time taken to complete knowledge service operations, in milliseconds |
| Knowledge File Transfer Duration | File loading time, in milliseconds. Failed operations can include processing time. |
| Knowledge Search Duration | Time taken to retrieve matching content, in milliseconds |
| Knowledge Embedding Duration | Time taken to generate embeddings, in milliseconds |
| Knowledge Insert Duration | Time taken to add vectors to the search index, in milliseconds |
| Knowledge Ingestion Queue Size | Number of ingestion jobs waiting or running |
| Call LLM Total Time in Seconds | Time taken for classic chat language model inference requests, in seconds |
| Get LLM Payload Total Time in Milliseconds | Time taken to prepare a classic chat language model inference request, in milliseconds |

## Related content

[Monitor Agents and Tools with Foundry Local](observability.md)
