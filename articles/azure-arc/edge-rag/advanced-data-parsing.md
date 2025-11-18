---
title: Advanced data parsing for Edge RAG
description: "Learn about advanced data parsing in Edge RAG to extract structured data, tables, and images for more accurate search and chat results."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 11/10/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a solution architect or data engineer, I want to use advanced data parsing in Edge RAG to extract structured information, tables, and images from complex documents so that I can enable more accurate, context-rich search and chat experiences for my users.
---
# Advanced data parsing for Edge RAG preview enabled by Azure Arc

Edge RAG offers an advanced data parsing option that helps you extract more value from your documents by capturing structure, tables, and images for more accurate, context-rich search and chat experiences. This article explains how advanced data parsing works in Edge RAG, when to use it, and how it can help you get the most from your data.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Key capabilities

Advanced data parsing provides several enhancements over basic parsing. The following list summarizes the main benefits:

- **Enhanced document understanding**: Extracts headings, tables, images, and formatting for richer context.
- **Advanced chunking**: Splits text into meaningful sections, preserving context and document hierarchy.
- **Rich metadata**: Captures paragraph headings, page numbers, and other structural details.
- **Intelligent table processing**: Detects, merges, and indexes tables—even those spanning multiple pages.
- **Improved retrieval accuracy**: Enables more relevant and precise search results.

## When to use advanced parsing

Choose advanced parsing when your documents include:

- Tables, charts, or images that are important for your use case.
- Complex structure, such as reports, scientific papers, or financial statements.
- A need for precise search, filtering, or attribution (such as page numbers or section headings).

If you only need to extract free-form text quickly, basic parsing might be sufficient.

## How advanced parsing works

Advanced parsing analyzes your documents to identify and preserve structure, including:

- Headings, paragraphs, lists, and hierarchy.
- Tables and images.
- Multiple file formats, including PDF, Word, PowerPoint, HTML, Markdown, and common image types.

Text is split into context-aware chunks that align with natural boundaries, such as sentences or sections. Each chunk includes metadata like headings and page numbers, making it easier to trace information back to its source.

### Supported file formats

Advanced parsing supports the following file types:

- .txt (text files)
- .pdf (PDF documents)
- .docx (Microsoft Word documents)
- .pptx (Microsoft PowerPoint presentations)
- .html (HTML documents)
- .md (Markdown files)
- .png, .jpg, .jpeg (images)

### Text chunking and metadata

Instead of simple character-based splitting, advanced parsing uses advanced chunking to create semantically meaningful sections. Each chunk includes metadata such as headings, page numbers, and unique chunk IDs. Table chunks include more details like table index, shape, and a preview of the content. This approach helps preserve context and improves retrieval relevance.

### Table extraction and processing

Advanced parsing automatically detects tables across all pages, including tables in scanned documents. It merges tables that span multiple pages, restores column headers, and ensures consistent column structure. Each table chunk includes metadata such as table index, shape, page numbers, section headings, and a preview of the table. This information makes tables searchable and retrievable with full context.

### Image extraction and processing

Advanced parsing automatically detects images across all pages. Each image is stored as the original source in full quality along with source page number. This information allows for full quality image display and source page context during inference.

## How advanced parsing improves results

Advanced parsing directly improves the quality of retrieval-augmented generation (RAG) responses. It helps you get better context retrieval, enhanced table understanding, and improved accuracy. For example, heading information helps identify relevant sections quickly, and page numbers allow precise source attribution. Semantic chunks align with query intent, and structured queries about tabular data are more accurate. These improvements help you deliver more accurate and trustworthy results to your users.

## Migrating from basic to advanced parsing

If you're switching from basic to advanced parsing, follow these steps:

1. Update your configuration by deleting existing ingested documents and selecting "Advanced" mode in the ingestion job configuration.
2. Re-ingest your documents. Documents processed in basic mode should be re-ingested to take advantage of richer chunks and metadata. Historical data remains accessible during migration.
3. Verify results by testing retrieval quality with sample queries, checking table extraction, and confirming that page numbers and headings appear in results.

## Performance considerations

Advanced parsing takes longer per document than basic parsing, especially for large documents. It also stores more metadata per chunk, which might increase storage requirements. However, the improved accuracy and context typically outweigh the extra processing time and storage needs.

## Best practices

To get the best results from advanced parsing, follow these recommendations:

- Use native digital formats (such as PDFs with selectable text) when possible.
- Ensure tables are well-formatted and consistent.
- Use clear, consistent headings to help the parser identify document structure.

Review your documents before ingestion to ensure they meet these guidelines.

## Troubleshooting

If you notice missing or incomplete tables, check that your source documents are well-structured and legible. For unexpected results, review the metadata and chunk details to verify that headings and page numbers are captured. If ingestion fails, try simplifying the document or splitting it into smaller sections.

## Related content

- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- [Set up the data query for Edge RAG chat solution](set-up-data-query.md)
- [Supported data sources](requirements.md#supported-data-sources)

