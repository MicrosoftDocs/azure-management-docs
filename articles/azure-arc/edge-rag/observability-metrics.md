---
title: Metrics Available For Monitoring Edge RAG
description: "Learn about the metrics you can use to monitor Edge RAG to track API performance, ingestion rates, and processing times for optimal system performance."
author: cwatson-cat
ms.author: cwatson
ms.topic: reference #Don't change
ms.date: 05/13/2025
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator or DevOps engineer, I want to monitor and analyze Edge RAG metrics to track API performance, ingestion rates, and processing times so that I can ensure optimal system performance and quickly identify and resolve issues in my infrastructure.
ms.custom:
  - build-2025
---

# Metrics available for monitoring Edge RAG Preview, enabled by Azure Arc

The following table lists the metrics available for Edge RAG.

| Metric Name | Description |
|---|---|
| API Failure Count | Count of failed API requests |
| API Request Count | Total number of API requests |
| API Request Duration in Seconds | Histogram of request durations |
| API Success Count | Count of successful API requests |
| Evaluation API Request Count | Total number of Evaluation API requests |
| Failed Skipped Count | Failed / Skipped file counter (Ingestion) |
| File Ingestion Rate | Total files ingested per Job |
| Hybrid Search Model API Request Count | Total number of Hybrid Search Model API requests |
| Inference Answer Feedback | The Inference Answer's Feedback |
| Inference API Request Count | Total number of Inference API requests |
| Ingestion Time | Total ingestion time in minutes |
| Ingestion API Request Count | Total number of Ingestion API requests |
| Input Preprocessing Time (Milliseconds) | Input preprocessing time in milliseconds |
| Number of Evaluations | Number of Evaluations |
| Number of Jobs | Number of jobs |
| Call LLM Total Time in Seconds | Total time in seconds for invoking "call_llm" function |
| Embedding Generation Total Time in Seconds | Total time taken to generate embeddings from local model |
| Hybrid Search Embedding Generation Total Time in Seconds | Total time taken to generate Hybrid Search embeddings from local model |
| Reranking Generation Total Time in Seconds | Total time taken to generate Reranking from local model |
| Get Chat History Summary Total Time in Milliseconds | Total time in milliseconds for invoking "get_chat_history_summary" function |
| Get LLM Payload Total Time in Milliseconds | Total time in milliseconds for invoking "get_llm_payload" function |
| Get Hybrid Search Total Time in Milliseconds | Total hybrid search time in milliseconds |
| Inference Total Time in Seconds | Total inference time in seconds |
| Chunks Search Total Time in Milliseconds | Total time in milliseconds for invoking "search_chunks" function |
| Search Total Time in Milliseconds | Total time taken for search |
| Similarity Search Total Time in Milliseconds | Total time taken to search for similar documents |
| Vector DB API Request Count | Total number of API requests to Vector DB |

## Related content

[Monitor Edge RAG](observability.md)