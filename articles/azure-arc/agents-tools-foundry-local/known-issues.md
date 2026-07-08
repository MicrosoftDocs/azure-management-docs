---
title: Known Issues in Agentic Retrieval in Foundry Local
description: "Read about known issues in Agentic Retrieval in Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 07/07/2026
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a cloud solution architect, I want to understand the known issues in Agentic Retrieval in Foundry Local so that I can make informed decisions during deployment and mitigate potential problems for my team.
---

# Known issues in Agentic Retrieval in Foundry Local

Before you deploy Agentic Retrieval, review the known issues and limitations.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Known issues for public preview

The following table lists the known issues in this release.


|Feature  |Issue |
|---------|---------|
|Automatic evaluation|If you run an automatic evaluation without adding a data source, you receive an error like: "Failed to calculate automatic metrics. Both \"answer\" and \"context\" must be non-empty strings." To work around this issue, [add a data source](add-data-source.md) before you run an evaluation.|
|Chat feedback    | End users of the chat solution can submit feedback, but the AI application developers and prompt engineers who set up the chat solution don't have an easy UI-based way to analyze that feedback. |
| Chat UI | Dollar amounts in agent responses (e.g., `$100`) may be rendered as LaTeX mathematical expressions in the agentic chat UI. **Workaround:** Configure agent instructions to format currency without the `$` symbol (e.g., "100 USD"), or use the API directly. |
| Agent behavior | In some cases, the agent might perform arithmetic operations on numerical data from the knowledge base instead of directly quoting the source values. **Workaround:** Add explicit instructions to the agent: "Always quote numerical values exactly as they appear in the source documents. Don't perform calculations unless explicitly asked." |
| Agent configuration | When an agent has no knowledge base or the knowledge base has no connected knowledge sources, the system doesn't display a user-facing warning. The agent responds without grounding, which might produce hallucinated answers. **Workaround:** Ensure all agents have a knowledge base with at least one knowledge source. |
| Collection RBAC | When an `EdgeRAGEndUser` has no app roles matching any collection name, they receive a `403 Forbidden` error with no guidance on what roles are needed. **Workaround:** Assign collection-specific app roles in Entra ID before granting `EdgeRAGEndUser` access. |
| Knowledge sources | The `indexed_source_ref` field refers to a **collection name** when it points to the built-in MCP server, not a tool name. This field is documented in the Knowledge Sources API reference but might still cause confusion. |
| Knowledge Base PATCH | When updating a knowledge base by using PATCH, the `knowledge_source_ids` field **replaces** the entire list. It doesn't append. **Workaround:** Read the current list first, then include all desired IDs in the PATCH request. |
| API timestamps | The Knowledge Base Manager returns timestamps as ISO 8601 strings (for example, `"2025-01-15T10:30:00Z"`), while the Agents Runtime uses Unix epoch integers (for example, `1699012345`). Ensure your client handles both formats. |
| Table parsing | Documents that contain malformed or deeply nested tables (particularly large HTML files larger than 500 KB with many nested `<table>` elements) might be only partially parsed. Table content can be silently dropped and unavailable to search. |
| Bring-your-own ingress controller | Agentic Retrieval bundles its own `ingress-nginx` controller, so bringing your own ingress controller isn't supported. When you install Foundry Local, skip the optional `ingress-nginx` step. A separate `ingress-nginx` release conflicts with the bundled one and causes the install to fail. **Workaround:** If one is already installed, remove it first with `helm uninstall ingress-nginx -n ingress-nginx`. |
| First request after pod start (disconnected/ALDO) | On Azure Local Disconnected Operations (ALDO) or air-gapped clusters that authenticate to Foundry Local using a managed identity token (`foundryClientId`), the first inference or agent request after any pod start, restart, rollout, or scale-up might fail with an HTTP 500 error: "MI token acquisition timed out". On disconnected stamps, the token is minted by the Azure Arc cluster-identity operator on a fixed 30-second poll cadence, so a freshly started pod can take up to about 60 seconds to obtain its first token. The subsequent request succeeds because the token is cached for the lifetime of the pod. The failure is transient and self-clearing. This behavior doesn't occur with BYOM/API-key authentication. **Workaround:** For disconnected deployments where predictable first-request latency is important, use BYOM/API-key authentication to Foundry Local, or retry the first request after a service restart or scale event. |

## Security considerations

The following security items are under review:

| Issue | Description | Mitigation |
|---|---|---|
| **PostgreSQL SSL disabled** (GAP-01) | PostgreSQL connections within the cluster don't use SSL by default. In-cluster traffic is unencrypted. | Deploy Agentic Retrieval in a network-isolated environment. Use network policies to restrict pod-to-pod communication. |
| **MCP bearer token forwarding** (GAP-02) | When agents connect to external MCP servers by using `auth_type: microsoft_entra_id`, the caller's bearer token is forwarded without re-validation. A malicious MCP server could capture and replay the token. | Only register trusted external MCP servers. Use `auth_type: unauthenticated` for external MCP servers that don't require auth. |


## Related content

- [What you need for Agentic Retrieval](requirements.md)
- [Complete Agentic Retrieval deployment prerequisites](complete-prerequisites.md)
- [Deploy the Agentic Retrieval extension](deploy.md)
