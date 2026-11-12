---
title: Known Issues in Agents and Tools with Foundry Local
description: "Read about known issues in Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: concept-article #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a cloud solution architect, I want to understand the known issues in Agents and Tools with Foundry Local so that I can make informed decisions during deployment and mitigate potential problems for my team.
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
---

# Known issues in Agents and Tools with Foundry Local

Before you deploy Agents and Tools with Foundry Local, review the known issues and limitations.

## Current known issues

The following table lists the known issues in this release.


|Feature  |Issue |
|---------|---------|
| Chat feedback (deprecated developer-portal classic chat) | In the deprecated developer-portal classic chat experience, users can submit feedback, but developers don't have a built-in UI to analyze that feedback. Built-in end-user chat at `/chat` doesn't use this endpoint, and `POST /chat/feedback` is restricted to the developer portal. |
| Local chat UI | Dollar amounts in agent responses (for example, `$100`) might be rendered as LaTeX mathematical expressions in the local chat UI. **Workaround:** Configure agent instructions to format currency without the `$` symbol (for example, "100 USD"), or use the API directly. |
| Agent behavior | In some cases, the agent might perform arithmetic operations on numerical data from the knowledge base instead of directly quoting the source values. **Workaround:** Add explicit instructions to the agent: "Always quote numerical values exactly as they appear in the source documents. Don't perform calculations unless explicitly asked." |
| Agent configuration | When an agent has no tools, or its knowledge base has no reachable knowledge sources, the system doesn't display a user-facing warning. The agent responds without grounding, which might produce hallucinated answers. **Workaround:** Add at least one knowledge base or MCP tool to each agent, and tell the agent in its instructions to search before it answers. |
| App-role collection access | When an `EdgeRAGEndUser` relies on app-role access and has no app roles matching a collection name, queries return `403 Forbidden` without guidance on the required role. **Workaround:** Assign the matching collection app role in Microsoft Entra ID. This limit doesn't apply to callers allowed by an Azure role or the `EdgeRAGDeveloper` app role. |
| Ingestion chunk settings | The portal shows chunk size and overlap fields, but these values aren't applied in the current release. The service uses Docling HybridChunker with the BGE-M3 tokenizer to split text into chunks of about 512 tokens. |
| Knowledge sources | The `indexed_source_ref` field refers to a **collection name** when it points to the built-in MCP server, not a tool name. This field is documented in the Knowledge Sources API reference but might still cause confusion. |
| Knowledge Base PUT | `PUT /knowledgebases('<name>')` **replaces** the entire knowledge base, including the `knowledgeSources` list. It doesn't append. **Workaround:** Read the current knowledge base first, then send the full list with an `If-Match` header. |
| API timestamps | The Knowledge Sources API returns timestamps as ISO 8601 strings (for example, `"2025-01-15T10:30:00Z"`), while the Agents and Responses APIs use Unix epoch integers (for example, `1699012345`). Ensure your client handles both formats. |
| Table parsing | Documents that contain malformed or deeply nested tables (particularly large HTML files larger than 500 KB with many nested HTML table elements) might be only partially parsed. Table content can be silently dropped and unavailable to search. |

## Security considerations

Keep the following security boundary in mind when you register an external MCP server:

| Issue | Description | Mitigation |
|---|---|---|
| **Authenticated MCP servers are trusted destinations** | Authenticated external MCP connections require HTTPS, and the extension blocks credential forwarding to destinations that aren't trusted. The registered server still receives the delegated credential, and its tool output is untrusted input to the agent. | Register only MCP servers that you operate or trust. Use `auth_type: unauthenticated` when the server doesn't require delegated authentication. |


## Related content

- [What you need for Agents and Tools with Foundry Local](requirements.md)
- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
