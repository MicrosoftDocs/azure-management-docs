---
title: KnowledgeBase CRD reference - Agents and Tools with Foundry Local
description: Field reference for the KnowledgeBase custom resource definition in Agents and Tools with Foundry Local.
author: Lea1702
ms.author: leabenero
ms.topic: reference
ms.date: 11/05/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
#CustomerIntent: As a platform engineer, I want a field reference for the KnowledgeBase custom resource so that I can inspect and troubleshoot knowledge bases.
---

# KnowledgeBase CRD reference

The `KnowledgeBase` custom resource stores the definition managed by the [Knowledge Bases REST API](knowledge-bases-api-reference.md).

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

> [!IMPORTANT]
> Create and modify knowledge bases through the REST API, not by applying YAML directly. The API validates the referenced knowledge sources and model connection, enforces the public request contract, and creates the managed `kb-<knowledge-base-name>` connection used for execution. Applying a custom resource directly skips those actions.

## Resource information

| Property | Value |
|---|---|
| **API group** | `foundrylocal.azure.com` |
| **Kind** | `KnowledgeBase` |
| **Version** | `v1` |
| **Scope** | Namespaced |
| **Plural** | `knowledgebases` |
| **Short name** | `kb` |

```bash
kubectl get kb -n arc-rag
```

## Name

`metadata.name` is the resource identity.

| Rule | Detail |
|---|---|
| **Characters** | Lowercase letters, digits, and hyphens only. |
| **First and last character** | Must be a letter or digit. |
| **Length** | 1-40 characters. |
| **Rename** | Unsupported. |

## spec

| Field | Required by CRD | Type | Description |
|---|---|---|---|
| `knowledgeSources` | Yes | Array | Knowledge source references by name. Maximum 100. |
| `description` | No | String | Description. Maximum 512 characters. The REST API requires a non-empty value. |
| `model` | No | String | Active connection/model binding in the form `<connection-name>/<model-name>`. The REST API requires a ready binding. |
| `retrievalInstructions` | No | String | Guidance for source selection and searching. Maximum 8,192 characters. |
| `answerInstructions` | No | String | Guidance for composing the grounded answer. Maximum 8,192 characters. |
| `outputMode` | No | String | CRD values: `answerSynthesis` or `extractiveData`. The current REST API accepts only `answerSynthesis`; current execution doesn't use this field. |
| `retrievalReasoningEffort` | No | Object | CRD values: `minimal`, `low`, or `medium`. The current REST API accepts only `low` or `medium`; current execution doesn't use this field. |
| `metadata` | No | Object | Up to 16 customer string tags. This field isn't exposed by the current REST request model. |
| `models` | No | Array | Reserved legacy-shaped model field. The current REST API writes the singular `model` field instead. |

### spec.knowledgeSources[]

| Field | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | String | Knowledge source name. 1-40 lowercase DNS-style characters. |

The CRD can't verify that a referenced knowledge source exists. The REST API performs that validation.

### spec.model

The active model binding uses `<connection-name>/<model-name>`, for example:

```yaml
model: cloud-model/gpt-4o
```

The REST API accepts an Azure OpenAI model object and stores it in this singular field. The referenced model connection must exist, be ready, and contain the selected model.

### spec.metadata

| Constraint | Value |
|---|---|
| Maximum pairs | 16 |
| Maximum key length | 64 characters |
| Maximum value length | 512 characters |

## status

The schema permits these status fields:

| Field | Type | Allowed values |
|---|---|---|
| `provisioningState` | String | `Provisioning` or `Succeeded` |
| `observedGeneration` | Integer | 0 or greater |

The current Foundry-compatible Knowledge Bases REST API doesn't write or return these status fields. Don't use them to determine whether an API write succeeded. Use the HTTP status, response body, and ETag.

## Service RBAC

The Agents Management service uses Role `agent-custom-resources-agents-management`:

| Resource | Verbs |
|---|---|
| `knowledgebases` | `get`, `list`, `create`, `patch`, `delete` |
| `knowledgesources` | `get`, `list` |
| `connections` | `get`, `list`, `create`, `patch`, `delete` |
| `connections/status` | `patch` |
| `agents` | `get`, `list`, `create`, `patch`, `delete` |
| `agents/status` | `patch` |
| `agentversions` | `get`, `list`, `create`, `delete` |
| Core `services` and `configmaps` | `get` |

The Knowledge Base Execution service uses Role `kb-execution-reader`:

| Resource | Verbs |
|---|---|
| `knowledgebases` | `get`, `list` |
| `knowledgesources` | `get`, `list` |
| `connections` | `get`, `list` |
| Core `services` and `configmaps` | `get` |

The Agents Execution service (`agents-execution`) uses Role `agent-custom-resources-agents-execution`:

| Resource | Verbs |
|---|---|
| `knowledgebases` | `get` |
| `knowledgesources` | `get` |
| `connections` | `get`, `list` |
| `agents` and `agentversions` | `get`, `list` |
| Core `services` and `configmaps` | `get` |

Inspect the roles and bindings:

```bash
kubectl get role agent-custom-resources-agents-management -n arc-rag -o yaml
kubectl get rolebinding agent-custom-resources-agents-management-binding -n arc-rag -o yaml
kubectl get role kb-execution-reader -n arc-rag -o yaml
kubectl get rolebinding kb-execution-reader-binding -n arc-rag -o yaml
kubectl get role agent-custom-resources-agents-execution -n arc-rag -o yaml
kubectl get rolebinding agent-custom-resources-agents-execution-binding -n arc-rag -o yaml
```

## Stored resource example

The following example shows the shape stored after a REST API request. Don't apply it directly.

```yaml
apiVersion: foundrylocal.azure.com/v1
kind: KnowledgeBase
metadata:
  name: docs-kb
  namespace: arc-rag
spec:
  description: Product documentation for the sales team.
  knowledgeSources:
    - name: my-docs-search
  model: cloud-model/gpt-4o
  retrievalInstructions: Prefer current product documentation.
  answerInstructions: Answer with citations.
  outputMode: answerSynthesis
  retrievalReasoningEffort:
    kind: low
```

## Related content

- [Knowledge bases REST API reference](knowledge-bases-api-reference.md)
- [Manage knowledge bases](../knowledge-bases-guide.md)