---
title: Deployment Parameter Reference for Agents and Tools with Foundry Local
description: "Configuration parameters, environment variables, and troubleshooting reference for deploying Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/18/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a platform administrator, I want a reference for deployment parameters and troubleshooting so that I can configure and maintain Agents and Tools with Foundry Local.
---

# Deployment parameter reference for Agents and Tools with Foundry Local

This article provides the configuration parameter reference, environment variables, and troubleshooting guidance for deploying Agents and Tools with Foundry Local.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Configuration parameters

The following configuration parameters are used when you install the Agents and Tools with Foundry Local extension:

| Parameter | Required | Description |
|---|---|---|
| `byom.enabled` | Yes | Always `true`. BYOM is the only language model path. |
| `byom.apiEndpoint` | Yes | Full endpoint URL. For Foundry Local: `https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions`. For Microsoft Foundry: `https://<resource>.cognitiveservices.azure.com/openai/deployments/<model>/chat/completions?api-version=<version>`. |
| `byom.apiModel` | Yes | Model name to send in requests (for example, `gpt-oss-20b`). |
| `byom.maxTokensInK` | Yes | Maximum tokens in thousands (for example, `16`). |
| `foundryClientId` | Conditional | Required only when using a Foundry Local model source with `useFoundryLocal=true`. Not required for non-Foundry Local BYOM endpoints. |
| `auth.tenantId` | Yes | Microsoft Entra ID tenant ID. |
| `auth.clientId` | Yes | Agents and Tools app registration client ID. |
| `isManagedIdentityRequired` | Yes | Always `true`. Enables managed identity token acquisition. |
| `layerSelection` | Yes | `combined`, `agentic`, or `knowledge`. |
| `ingress.domainname` | Yes | Full DNS name for external access (for example, `mycluster.eastus.cloudapp.azure.com`). |
| `gpu_enabled` | No | Set to `true` for GPU clusters. Enables GPU-accelerated embedding models. |
| `min_gpu_nodes` | No | Minimum GPU nodes required. Default: `2`. |
| `AgentOperationTimeoutInMinutes` | No | Timeout for agent operations. Default: `30`. |
| `model` | No | Always `byom`. No other option. |
| `llm.dapr.accessControl.defaultAction` | No | Dapr access control. Set to `allow`. |
| `embeddingmodel.image.gpu.repository` | No | GPU embedding model image repository. |
| `embeddingmodel.image.gpu.tag` | No | GPU embedding model image tag. |

The BYOM API key is not passed as a configuration parameter. It's stored as a Kubernetes secret (`byom-api-key`) in the `arc-rag` namespace before extension installation.

The Azure CLI accepts both `--config` and `--configuration-settings` for Arc extension parameters. Both syntaxes are equivalent.

## Environment variables

Helm templates populate the following environment variables for all inferencing pods:

| Variable | Source |
|---|---|
| `BYOM_ENABLED` | Always `true` |
| `BYOM_ENDPOINT` | `byom.apiEndpoint` |
| `BYOM_MODEL` | `byom.apiModel` |
| `BYOM_API_KEY` | `byom.apiKey` |
| `FOUNDRY_CLIENT_ID` | `foundryClientId` (when configured) |

## Troubleshoot Foundry Local integration

Use the following commands to diagnose Foundry Local issues:

| Command | Purpose |
|---|---|
| `kubectl describe mdep <name>` | Check ModelDeployment status and events. |
| `kubectl logs -f deployment/inference-operator -n foundry-local-operator` | Check operator logs. |
| `kubectl get pods -l app.kubernetes.io/managed-by=inference-operator` | Check inference pod status. |
| `kubectl describe pod <pod_name>` | Get pod details and events. |
| `kubectl get deploy,svc,ing -l foundry.azure.com/deployment=<name>` | List all resources created by a deployment. |
| `kubectl get configmap foundry-local-catalog -n foundry-local-operator -o yaml` | Check the model catalog ConfigMap. |

### Common integration issues

| Symptom | Cause | Resolution |
|---|---|---|
| Connection refused or timeout | Foundry Local not running or network policy blocking egress. | Verify Foundry pods are running. Ensure egress from `arc-rag` namespace to Foundry ingress is allowed. |
| `SSL: CERTIFICATE_VERIFY_FAILED` | `foundryClientId` not set in extension configuration. | Set `foundryClientId` — this gates the CA bundle mount. Without it, the Foundry self-signed certificate isn't trusted. |
| `400 Bad Request: plain HTTP sent to HTTPS port` | Using `http://` instead of `https://` in `byom.apiEndpoint`. | Change the endpoint to `https://`. Foundry Local enables TLS by default. |
| `400 Invalid JSON in request body` | Using `onnx-genai` runtime with agentic or combined mode. | Switch to `vllm` runtime. The `onnx-genai` runtime doesn't support `tools` or `tool_choice` parameters. |
| `401 Token validation failed` | RBAC roles not assigned for the managed identity. | Assign `Reader` + `Cognitive Services OpenAI User` + `FoundryInferenceAccess` app role. See [Configure authentication](prepare-authentication.md). |
| `401 Entra ID authentication is not enabled` | Sending managed identity token to Foundry with `entraAuth.enabled=false`. | Either enable Microsoft Entra authentication on Foundry, or clear `FOUNDRY_CLIENT_ID` so Agents and Tools uses API key authentication. |
| `401 Invalid API key` | API key rotated after model redeployment. | Re-read the key from the `gpt-oss-20b-api-keys` secret and update `byom-api-key` in the `arc-rag` namespace. |
| `404 Not Found` from Foundry | Model not deployed. | Run `kubectl get mdep -n foundry-local-operator` and verify the model name matches `byom.apiModel`. |
| LLM calls fail but embedding and ingestion work | Expected behavior. Embedding models are local; only LLM inference uses Foundry. | Check Foundry connectivity and model deployment status. |
| Managed identity token acquisition fails | Microsoft Entra ID unreachable or msi-adapter not running. | Check msi-adapter sidecar logs. The request falls back to API key authentication only. |
| Pods pending (insufficient resources) | Cluster too small for combined mode (60+ pods). | Combined mode requires at least 3x Standard_D8s_v3 (24 vCPU, 96 GB RAM) worker nodes + 1 GPU node. Scale node pools with `az aksarc nodepool scale`. |
| Extension install: stale nginx webhook | Previous install left `ValidatingWebhookConfiguration`. | Run `kubectl delete validatingwebhookconfiguration ingress-nginx-admission` before reinstalling. |

## Foundry Local operator parameters

You can set these optional parameters during Foundry inference operator installation:

| Parameter | Description |
|---|---|
| `entraAuth.enabled` | When enabled, Microsoft Entra Auth SDK and msi-adapter sidecars are injected into inference pods for JWT validation and ARM RBAC authorization. When disabled, `tenantId` and `clientId` are optional. Default: `true`. |
| `watch.namespaces` | Configure if the operator should manage resources across multiple namespaces. Default: `foundry-local-operator`. Pass as: `--config watch.namespaces[0]="<namespace_1>" --config watch.namespaces[1]="<namespace_2>"`. |

## Foundry Local key management

You can retrieve and rotate API keys by using the Foundry Local inference service:

| Endpoint | Description |
|---|---|
| `GET /namespaces/<namespace>/deployments/<name>/keys` | Retrieve both primary and secondary keys. |
| `POST /namespaces/<namespace>/deployments/<name>/keys/{primary\|secondary}/rotate` | Rotate a specific key. |

## Related content

- [Deploy Agents and Tools with Foundry Local](deploy.md)
- [Configure BYOM endpoint authentication](configure-endpoint-authentication.md)
- [Create a BYOM endpoint](prepare-model-endpoint.md)
