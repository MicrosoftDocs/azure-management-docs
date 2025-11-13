---
title: Edge Retrieval Augmented Generation (RAG) Overview
description: "Learn about the Azure Arc-enabled Kubernetes extension Edge RAG used to search on-premises data with generative AI."
author: cwatson-cat
ms.author: cwatson
ms.topic: overview #Don't change
ms.date: 10/30/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a business decision maker, I want to evaluate the Edge RAG solution for on-premises data retrieval, so that I can determine if it aligns with our regulatory compliance and AI application needs while enhancing insights and decision-making processes.
---
# What is Edge Retrieval Augmented Generation (RAG)?

Edge RAG Preview is an [Azure Arc-enabled Kubernetes extension](/azure/azure-arc/kubernetes/extensions-release) that enables you to search on-premises data with generative AI, using Retrieval Augmented Generation (RAG). RAG is an industry-standard architecture that augments the capabilities of a language model with private data.

Edge RAG Preview, enabled by Azure Arc is a turnkey solution that packages everything that's necessary to allow customers to build custom chat assistants and derive insights from their private data, including:

- A choice of Generative AI (GenAI) language models running locally with support for both CPU and GPU hardware.
- A turnkey data ingestion and RAG pipeline that keeps all data local, with Azure role-based access controls (Azure RBAC) to prevent unauthorized access.  
- An out-of-the-box prompt engineering and evaluation tool to find build, evaluate, and deploy custom chat solutions.
- Azure-equivalent APIs to integrate into business applications, and a prepackaged UI to get started quickly.

Although Edge RAG is capable of ingesting and retrieving relevant images to be used as contextual references alongside text, it's important to note that it isn't a visual language model (VLM).

Edge RAG is supported and validated on Azure Arc-enabled Kubernetes on Azure Local (formerly Azure Stack HCI) infrastructure and as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).

For more information, see [Azure Arc](/azure/azure-arc/overview), [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/), and [Azure Arc extensions](/azure/azure-arc/kubernetes/conceptual-extensions). 

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Customer scenarios and use cases

For customers across verticals like manufacturing, financial services, healthcare, government, and defense, valuable data is generated and stored locally. This happens outside of the hyperscale cloud due to regulation, latency, business continuity, or the sheer volume of data generated in real time. Customers want to use generative AI applications to get insights from this on-premises data.

Edge RAG supports Q&A capabilities that allow customers to query on-premises data via a custom chat bot for scenarios like: 

- A government customer wants to derive insights from sensitive on-premises data to enable quicker decision making, summarize large datasets, create training materials, and more.

- A regional bank wants to use data that must remain on-premises due to regulatory constraints for use cases like compliance checks, customer assistance, and personalized sales pitch generation.

- A global manufacturer wants to create factory floor assistants to reduce time to issue resolution and aid troubleshooting, using data that needs to stay local to comply with organization policies.

## Why Edge RAG?

Use Edge RAG to:

- **Reduce time to market** with a turnkey experience that accelerates the development and deployment of AI applications on local data. 
- **Simplify operations** and **end-to-end management** with an enterprise quality solution that delivers the same standard of security, compliance, and manageability you to expect from Microsoft, including lifecycle and version management of all components and Microsoft Entra integration for Azure RBAC.  
- **Remove the need for separate developer skillsets** with cloud-consistent developer experiences 
- Stay on top of this rapidly evolving space with **continuous innovation from Microsoft**, the leader in AI technologies, and continue to focus on delivering business value.

## Key concepts

Review the following key concepts for Edge RAG:

- **Chunking** splits large documents into smaller, manageable text blocks (chunks).
  - Chunk size: Chunking divides large documents into smaller units, with settings like chunk size (for example, 1000-2000 characters) and chunk overlap (for example, 100-500 characters) controlling their granularity and continuity. Smaller chunks improve retrieval precision but might lose context, while larger chunks ensure comprehensive context at the cost of precision.
  - Chunk overlap: Overlapping chunks maintain context across boundaries but increase storage and computation requirements.
  
  Optimal chunk settings depend on the use case, balancing accuracy, efficiency, and performance.

- **Data ingestion** is process of importing and preparing external content, such as documents or images, to be used for retrieval. This includes preprocessing steps like cleaning, formatting, and organizing data.

- **Embedding models** transform text, images, or other data into dense numerical vectors (embeddings) that capture semantic meaning. These vectors represent relationships between inputs, allowing for similarity comparisons and clustering.

- **Inferencing** refers to the process of using a trained model to generate predictions or outputs based on new input data. In language models, inferencing involves tasks like completing text, answering questions, or generating summaries.

- **Language models** are AI systems trained to understand, generate, and manipulate human language. They predict text based on input, enabling tasks like text generation, translation, summarization, and question answering. Examples include GPT, Phi, and Mistral.

- **Model parameters** control how the language model generates text, such as the creativity, diversity, and focus of responses. Common parameters include Temperature, and Top-p. Model parameters don't affect which documents are retrieved, only how the model generates its response. For more information, see [Search type parameters in Edge RAG](search-types.md#search-type-parameters).

- **Query** is the input provided to a language model to elicit a response or perform a specific task. It can be a question, a prompt, or a set of instructions, depending on the use case.

- **Retrieval Augmented Generation (RAG)** combines a retrieval system with a generative language model to produce responses enriched by external knowledge. It retrieves relevant context from a database or document store to augment the model's generation capabilities, ensuring accurate and up-to-date information.

- **Search parameters** are settings that control how Edge RAG retrieves, filters, and ranks documents from your indexed data before passing them to the language model. These parameters help you fine-tune the relevance, precision, and scope of the information used to answer user queries. For more information, see [Search type parameters in Edge RAG](search-types.md#search-type-parameters).

- **Search type**: A search type is the method Edge RAG uses to find and rank information from your indexed data. It determines how the system retrieves relevant content to answer user questions, such as by matching keywords, using semantic similarity, or combining multiple approaches. Edge RAG supports several search methods for retrieving information, including deep search, full text search, hybrid search, hybrid multimodal search, and vector search. For more information, see [Types of search in Edge RAG](search-types.md).

- **System prompt** are predefined instructions or messages provided to a language model at the start of a conversation or task to influence its behavior. These prompts define the model's role, tone, or task-specific context. For example, "You're a helpful assistant" or "Provide concise technical explanations." By shaping the initial context, system prompts ensure that the model generates responses aligned with the desired objective or persona.

- **Vector database** is a specialized database to store vector embeddings. It's designed to handle high-dimensional vectors and enables fast and scalable similarity searches.

- **Vectorization** means transforming text into numerical representations, or embeddings, using an embedding model such as Sentence Transformers. These embeddings capture the semantic meaning of text, enabling efficient and accurate comparisons.

## Compare with Azure AI services

Edge RAG runs on customer infrastructure outside the public cloud, allowing customers to search their on-premises data using Retrieval Augmented Generation (RAG). The data plane, including all customer data and the language model, is hosted locally.

In contrast, Azure AI services such as Azure AI Search and Azure AI Foundry also provide RAG capabilities but are hosted in hyperscale cloud regions, requiring customers to bring their data and applications to Azure infrastructure.

Edge RAG provides local developer UI experiences that are aligned to Azure AI Foundry experiences.

## Data on-premises versus cloud

Edge RAG sends only system metadata and organizational identifiable Information like subscription ID and cluster names to Microsoft. All customer content always stays in the on-premises infrastructure within the network boundaries defined by customers.

## User roles

The Edge RAG solution has three distinct user roles:

- **Lifecycle management of the extension**: Users are responsible for managing the lifecycle of the Edge RAG Arc extension. This includes tasks such as setting up the necessary infrastructure, deploying the extension, performing updates, monitoring its performance, and handling its eventual deletion. Typically, these responsibilities fall to an IT administrator with access to the underlying Azure Local and Azure Kubernetes (AKS) on Azure Local infrastructure.
- **Development and evaluation of chat endpoint**: The user responsibilities in this workflow include providing the data source, customizing the RAG pipeline settings, providing custom system prompts, evaluating, monitoring, and updating the chat solution. This role is typically carried out by a prompt engineer or an AI application developer.
- **Consuming the endpoint to query the on-premises data**: The user responsibilities in this workflow can include integration of the chat endpoint into line-of-business applications and using a chat interface, custom, or the one provided out-of-the-box, to query on-premises data.

## Related content

- [What you need for Edge RAG](requirements.md)
- [Arc Jumpstart](https://jumpstart.azure.com/)
