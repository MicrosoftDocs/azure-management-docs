---
title: Known Issues in Agents and Tools with Foundry Local
description: "Read about the known issues and fixed issues with Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 05/04/2026
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a cloud solution architect, I want to understand the known issues in Agents and Tools with Foundry Local so that I can make informed decisions during deployment and mitigate potential problems for my team.
---

# Known issues in Agents and Tools with Foundry Local

Before you deploy Agents and Tools with Foundry Local, review the known issues and limitations.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Known issues for public preview

The following table lists the known issues in this release.


|Feature  |Issue |
|---------|---------|
|Automatic evaluation|If you run an automatic evaluation without adding a data source, you receive an error like: "Failed to calculate automatic metrics. Both "answer" and "context must be non-empty strings." To work around this issue, [add a data source](add-data-source.md) before you run an evaluation.|
|Chat feedback    | End users of the chat solution can submit feedback about the chat, but the AI Application Developers/Prompt Engineers that set up the chat solution don't have an easy UI-based way to analyze the feedback. |
|Chat history | With Agents and Tools with Foundry Local extension version 0.1.5 and later each question is answered based on retrieved content only. The answer doesn't include the context of the chat history. Chat history isn't saved between questions. Treat each question as a new chat. |
| Chat UI | Dollar amounts in agent responses (e.g., `$100`) may be rendered as LaTeX mathematical expressions in the agentic chat UI. **Workaround:** Configure agent instructions to format currency without the `$` symbol (e.g., "100 USD"), or use the API directly. |
| Agent behavior | In some cases, the agent might perform arithmetic operations on numerical data from the knowledge base instead of directly quoting the source values. **Workaround:** Add explicit instructions to the agent: "Always quote numerical values exactly as they appear in the source documents. Don't perform calculations unless explicitly asked." |
| Agent configuration | When an agent has no knowledge base or the knowledge base has no connected knowledge sources, the system doesn't display a user-facing warning. The agent responds without grounding, which might produce hallucinated answers. **Workaround:** Ensure all agents have a knowledge base with at least one knowledge source. |
| Collection RBAC | When an `EdgeRAGEndUser` has no app roles matching any collection name, they receive a `403 Forbidden` error with no guidance on what roles are needed. **Workaround:** Assign collection-specific app roles in Entra ID before granting `EdgeRAGEndUser` access. |
| Knowledge Sources | The `indexed_source_ref` field refers to a **collection name** when pointing to the built-in MCP server, not a tool name. This field is documented in the Knowledge Sources API Reference but might cause confusion. |
| Knowledge Base PATCH | When updating a knowledge base by using PATCH, the `knowledge_source_ids` field **replaces** the entire list. It doesn't append. **Workaround:** Read the current list first, then include all desired IDs in the PATCH request. |
| API timestamps | The Agent Manager returns timestamps as ISO 8601 strings (for example, `"2025-01-15T10:30:00Z"`), while the Agents Runtime uses Unix epoch integers (for example, `1699012345`). Ensure your client handles both formats. |

The chat history limitation applies to the legacy `/user` chat endpoint (Inference API). The new agentic chat interface supports full multithread conversations with persistent thread history via the Agents Runtime.

## Security considerations

The following security items are under review:

| Issue | Description | Mitigation |
|---|---|---|
| **PostgreSQL SSL disabled** (GAP-01) | PostgreSQL connections within the cluster don't use SSL by default. In-cluster traffic is unencrypted. | Deploy Agents and Tools with Foundry Local in a network-isolated environment. Use network policies to restrict pod-to-pod communication. |
| **MCP bearer token forwarding** (GAP-02) | When agents connect to external MCP servers by using `auth_type: microsoft_entra_id`, the caller's bearer token is forwarded without re-validation. A malicious MCP server could capture and replay the token. | Only register trusted external MCP servers. Use `auth_type: unauthenticated` for external MCP servers that don't require auth. |


## Related content

- [What you need for Agents and Tools with Foundry Local](requirements.md)
- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
