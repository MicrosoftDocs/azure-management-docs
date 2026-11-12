---
title: Data parsing for Agents and Tools with Foundry Local
description: "Learn how data parsing in Agents and Tools with Foundry Local extracts structured data, tables, and images for more accurate search and chat results."
ms.reviewer: assafakiva
ms.topic: concept-article #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a solution architect or data engineer, I want to understand data parsing in Agents and Tools with Foundry Local so that I can extract structured information, tables, and images from complex documents for accurate, context-rich search and chat experiences.
---
# Agents and Tools with Foundry Local data parsing

The Agents and Tools with Foundry Local extension uses advanced data parsing to help you extract more value from your documents by capturing structure, tables, and images for more accurate, context-rich search and chat experiences. This article explains how data parsing works, what it extracts, and how to optimize your content for better retrieval results.

## Key capabilities

Data parsing in the extension provides the following capabilities:

- **Enhanced document understanding**: Extracts headings, tables, images, and formatting for richer context.
- **Semantic chunking**: Splits text into meaningful sections, preserving context and document hierarchy.
- **Detailed metadata**: Captures paragraph headings, page numbers, and other structural details.
- **Intelligent table processing**: Extracts table grids directly, groups up to 10 rows per chunk with column headers (`ColName: value`), and records table index, row range, and page numbers.
- **Improved retrieval accuracy**: Enables more relevant and precise search results.

## When data parsing adds the most value

Data parsing is built in and is especially valuable when your documents include:

- Tables, charts, or images that are important for your use case.
- Complex structure, such as reports, scientific papers, or financial statements.
- A need for precise search, filtering, or attribution (such as page numbers or section headings).

## How data parsing works

Data parsing analyzes your documents to identify and preserve structure, including:

- Headings, paragraphs, lists, and hierarchy.
- Tables and images.
- Multiple file formats, including PDF, Word, PowerPoint, Excel, HTML, Markdown, and common image types.

The process splits text into context-aware chunks that align with natural boundaries, such as sentences or sections. Each chunk includes metadata like headings and page numbers, making it easier to trace information back to its source.

### Supported file formats

Data parsing supports the following file types:

- .txt (text files)
- .pdf (PDF documents)
- .docx (Microsoft Word documents)
- .pptx (Microsoft PowerPoint presentations)
- .xlsx (Microsoft Excel workbooks)
- .csv (comma-separated values files)
- .html (HTML documents)
- .md (Markdown files)
- .png, .jpg, .jpeg (images)

Each file can be up to 30 MB. The ingestion service skips files that exceed this limit.

### Text chunking and metadata

Data parsing uses semantic chunking to create meaningful sections. Text chunks include metadata such as headings, page numbers, and unique chunk IDs. Table chunks group up to 10 rows per chunk with column headers (`ColName: value`) and record the table index, row range, and page numbers. This approach helps preserve context and improves retrieval relevance.

### Table extraction and processing

Data parsing detects tables across pages, including tables in scanned documents. It extracts table grids directly and groups up to 10 rows per chunk with column headers (`ColName: value`). Each table chunk records the table index, row range, and page numbers. This information makes tables searchable and retrievable with source context.

Malformed or deeply nested tables might be only partially parsed. Review the extracted content before relying on these tables for search.

### Image extraction and processing

Data parsing automatically detects images across all pages and stores each image at full quality with its source page number metadata. This information supports full-quality image display and source page context during inference.

## How data parsing improves results

Data parsing directly improves the quality of retrieval-augmented generation (RAG) responses. It helps you get better context retrieval, enhanced table understanding, and improved accuracy. For example, heading information helps identify relevant sections quickly, and page numbers allow precise source attribution. Semantic chunks align with query intent, and structured queries about tabular data are more accurate. These improvements help you deliver more accurate and trustworthy results to your users.

## Performance considerations

Data parsing can take longer for large, complex documents because it extracts richer structure and metadata. It can also increase storage requirements per chunk. Split files that exceed the 30-MB limit before ingestion.

## Best practices

To get the best results from data parsing, follow these recommendations:

- Use native digital formats (such as PDFs with selectable text) when possible.
- Ensure tables are well-formatted and consistent.
- Use clear, consistent headings to help the parser identify document structure.

Review your documents before ingestion to ensure they meet these guidelines.

## Troubleshooting

If you notice missing or incomplete tables, check that your source documents are well-structured and legible. For unexpected results, review the metadata and chunk details to verify that headings and page numbers are captured. If ingestion fails, try simplifying the document or splitting it into smaller sections.

## Related content

- [Agentic Retrieval overview](knowledge-layer-overview.md)
- [Add a data source](add-data-source.md)
- [Set up the data query](set-up-data-query.md)
- [Supported data sources](requirements.md#supported-data-sources)
