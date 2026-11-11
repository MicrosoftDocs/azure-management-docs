---
title: Deployment Parameter Reference for Agents and Tools with Foundry Local
description: "Configuration parameters, environment variables, and troubleshooting reference for deploying Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a platform administrator, I want a reference for deployment parameters and troubleshooting so that I can configure and maintain Agents and Tools with Foundry Local.
---

# Deployment parameter reference for Agents and Tools with Foundry Local

This article provides the configuration parameter reference, environment variables, and troubleshooting guidance for deploying Agents and Tools with Foundry Local.

## Configuration parameters

Use the following configuration parameters when you install the extension:

| Parameter | Required | Description |
|---|---|---|
| `byom.enabled` | Conditional | Set to `true` when `indexedSourcesEnabled=true` and you use deprecated classic chat in the developer portal. Omit for an agent-only installation. |
| `byom.apiEndpoint` | Conditional | Required only when indexed sources are enabled and you use deprecated classic chat in the developer portal. Full endpoint URL for the deployment-level inference path. |
| `byom.apiModel` | Conditional | Required only when indexed sources are enabled and you use deprecated classic chat in the developer portal. Model name to send in requests (for example, `gpt-oss-20b`). |
| `byom.maxTokensInK` | Conditional | Required only when indexed sources are enabled and you use deprecated classic chat in the developer portal. Maximum tokens in thousands (for example, `16`). |
| `foundryClientId` | Conditional | Foundry app registration client ID. Set it when indexed sources or agents use Foundry Local on this cluster. Agents use it to trust the Foundry Local certificate and as the default managed identity audience. When indexed sources are enabled, it also switches the indexed-source endpoint from `byom-api-key` authentication to managed identity authentication. |
| `serviceUrls` | No | JSON array of approved HTTPS base URLs that models or MCP servers can call with the extension's managed identity (populates `approvedMiDestinations` in the `mi-destinations` ConfigMap in `model-access-arc-rag`). |
| `auth.tenantId` | Yes | Microsoft Entra ID tenant ID. |
| `auth.clientId` | Yes | Agents and Tools with Foundry Local app registration client ID. |
| `isManagedIdentityRequired` | Yes | Always `true`. Enables managed identity token acquisition. |
| `indexedSourcesEnabled` | No | Enables local ingestion, collections, and Indexed Sources MCP. Defaults to `true` in Helm and Azure CLI (the Azure portal template defaults to `false`). Set to `false` to deploy the core Agent Service without the local indexed-source stack. |
| `sharepoint.sharepointIngestionEnabled` | No | Enables SharePoint Server indexed ingestion when `indexedSourcesEnabled=true`. Default: `true`. |
| `kerberos.enabled` | No | Set to `true` to enable NFS with Kerberos (`krb5p`) authentication for indexed sources. Default: `false`. |
| `kerberos.spn` | Conditional | Kerberos service principal name (SPN) in `service/host@REALM` format (for example, `nfs/edgerag-svc@CONTOSO.COM`). Required when `kerberos.enabled=true`. |
| `sharepointMcp.cache.claimName` | No | Existing ReadWriteMany (RWX) PersistentVolumeClaim name for the Microsoft 365 Local SharePoint file cache. |
| `exchangeMcp.cache.claimName` | No | Existing ReadWriteMany (RWX) PersistentVolumeClaim name for the Microsoft 365 Local Exchange file cache. |
| `networking.mode` | No | Ingress mode. Use `envoy` for the built-in Envoy Gateway (default) or `none` for a bring-your-own ingress controller. The removed `nginx` mode is rejected. |
| `networking.domainName` | Yes | Full DNS name for external access (for example, `mycluster.eastus.cloudapp.azure.com`). |
| `networking.tlsSecretName` | No | Kubernetes TLS Secret used by the gateway or your ingress controller. |
| `networking.dnsLabelName` | No | Azure DNS label for the load balancer IP. |
| `networking.envoy.enabled` | No | Installs the Envoy Gateway subchart. Keep `true` with `networking.mode=envoy`. Set `false` with `networking.mode=none`. |
| `gpu_enabled` | No | Set to `true` for GPU clusters. Enables GPU-accelerated embedding models. |
| `skipGPUValidations` | No | Set to `true` to skip pre-install GPU node validation (use when `indexedSourcesEnabled=false` or `gpu_enabled=false`). Default: `false`. |
| `min_gpu_nodes` | No | Minimum GPU nodes required. Default: `2`. |
| `AgentOperationTimeoutInMinutes` | No | Timeout for agent operations. Default: `30`. |

The `byom.*` settings and the `byom-api-key` Secret are used only by classic chat in the developer portal. Classic chat is deprecated, and its APIs accept requests only from the developer portal. Agents don't use this key. They use [model connections](create-manage-agents.md).

When indexed sources use API-key authentication, don't pass the key as a configuration parameter. Store it as a Kubernetes secret (`byom-api-key`) in the `arc-rag` namespace before extension installation.

The Azure CLI accepts both `--config` and `--configuration-settings` for Arc extension parameters. Both syntaxes are equivalent.

## Environment variables

When indexed sources are enabled, Helm templates populate the following environment variables for Inferencing Flow and VectorDB API:

| Variable | Source |
|---|---|
| `BYOM_ENABLED` | `true` when `byom.apiEndpoint` is non-empty; `false` otherwise |
| `BYOM_ENDPOINT` | `byom.apiEndpoint` |
| `BYOM_MODEL` | `byom.apiModel` |
| `BYOM_API_KEY` | `BYOM_API_KEY` in the `byom-api-key` Kubernetes Secret |
| `FOUNDRY_CLIENT_ID` | `foundryClientId` (when configured) |

## Troubleshoot Foundry Local integration

Use the following commands to diagnose Foundry Local issues:

| Command | Purpose |
|---|---|
| `kubectl describe mdep <name>` | Check ModelDeployment status and events. |
| `kubectl logs -f deployment/inference-operator -n foundry-local-operator` | Check operator logs. |
| `kubectl get pods -l app.kubernetes.io/managed-by=inference-operator` | Check inference pod status. |
| `kubectl describe pod <pod_name>` | Get pod details and events. |
| `kubectl get deploy,svc,gateway,httproute -A` | List the workloads and Gateway API resources used for external access. |
| `kubectl get configmap foundry-local-catalog -n foundry-local-operator -o yaml` | Check the model catalog ConfigMap. |

### Common integration issues

| Symptom | Cause | Resolution |
|---|---|---|
| Connection refused or timeout | Foundry Local isn't running or network policy is blocking egress. | Verify Foundry pods are running. Ensure egress from `arc-rag` namespace to Foundry ingress is allowed. |
| `SSL: CERTIFICATE_VERIFY_FAILED` | The Foundry Local CA bundle isn't available to the calling workload, or the endpoint doesn't identify the in-cluster Foundry Local service. | Verify the Foundry Local endpoint, certificate ConfigMap, and certificate mount in the affected deployment. |
| `400 Bad Request: plain HTTP sent to HTTPS port` | Using `http://` instead of `https://` in `byom.apiEndpoint`. | Change the endpoint to `https://`. Foundry Local enables TLS by default. |
| `400 Invalid JSON in request body` | Using the `onnx-genai` runtime with the current Agent Service. | Switch to the `vllm` runtime. The `onnx-genai` runtime doesn't support `tools` or `tool_choice` parameters. |
| `401 Token validation failed` | The token audience isn't the Foundry Local application client ID, or the token is expired or invalid. | Set `foundryClientId` to the Foundry Local application client ID. After you fix it, restart the affected pods so that they get a new token. See [Configure Foundry Local inference authentication](configure-foundry-inference-authentication.md). |
| `403 insufficient_permissions` | The managed identity doesn't have the inference role on the Foundry extension. | Assign `Cognitive Services OpenAI User` in connected deployments, or `Contributor` in disconnected deployments, at the Foundry extension scope. See [Configure Foundry Local inference authentication](configure-foundry-inference-authentication.md). |
| `500 rbac_check_unavailable` | Foundry Local can't check role assignments in Azure Resource Manager. | Assign `Reader` to the connected cluster and Foundry operator managed identities, and verify that Foundry Local can reach Azure Resource Manager. |
| `401 Entra ID authentication is not enabled` | Sending a managed identity token to Foundry with `entraAuth.enabled=false`. | Enable Microsoft Entra authentication on Foundry, or remove the `foundryClientId` extension setting and configure API-key authentication. |
| `401 Invalid API key` | API key rotated after model redeployment. | Re-read the key from the `<model-name>-api-keys` Secret and update `byom-api-key` in the `arc-rag` namespace. |
| `404 Not Found` from Foundry | Model not deployed. | Run `kubectl get mdep -n foundry-local-operator` and verify the model name matches `byom.apiModel`. |
| LLM calls fail but embedding and ingestion work | Expected behavior. Embedding models are local; only LLM inference uses Foundry. | Check Foundry connectivity and model deployment status. |
| Managed identity token acquisition fails | Microsoft Entra ID is unreachable or msi-adapter isn't running. | Check msi-adapter sidecar logs. There's no automatic API-key fallback when `foundryClientId` is configured. Fix managed identity authentication, or remove `foundryClientId` and configure API-key authentication. |
| Pods pending (insufficient resources) | The cluster can't satisfy one or more workload resource requests. | Inspect pending pod events, compare requests with allocatable node capacity, and scale or resize the appropriate node pool. Validate the resulting capacity with your workload. |
| Upgrade from a legacy nginx-based release fails on a stale webhook | A previous release left the `ingress-nginx-admission` `ValidatingWebhookConfiguration`. | Confirm that no other installation uses the webhook, and then delete it before reinstalling: `kubectl delete validatingwebhookconfiguration ingress-nginx-admission`. |

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
