---
title: Configure Foundry Local inference authentication
description: Learn how to authorize Foundry Local inference for Agents and Tools with Foundry Local by assigning Azure RBAC roles.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
#CustomerIntent: As a cloud administrator, I want to configure Foundry Local inference authentication after deploying Agents and Tools with Foundry Local so that model inference requests are authorized and reliable.
---

# Configure Foundry Local inference authentication

After you deploy the Agents and Tools with Foundry Local extension with `foundryClientId`, assign the Azure roles in this article so that Foundry Local accepts managed identity calls. These steps assume that both extensions run on the same Arc-connected cluster.

Use this article only for Foundry Local managed identity authentication. For bring-your-own-model API key authentication, see [Configure authentication for BYOM](configure-endpoint-authentication.md).

> [!IMPORTANT]
> These grants allow the extension to call a model. They don't replace the API-access roles. For every authenticated deployment, [assign Reader to the extension identity](deploy.md#allow-the-extension-to-check-azure-roles) at the Agents and Tools extension resource scope, and [assign Azure roles to API callers](prepare-authentication.md#assign-azure-roles). Without Reader or an inherited equivalent, Responses and Conversations can return `503` with `Authorization service unavailable`.

## Prerequisites

Before you begin, ensure that you:

- [Register a Foundry Local application](prepare-authentication.md#optional-register-a-foundry-local-application).
- [Deploy the extension for Agents and Tools with Foundry Local](deploy.md).
- Have permissions to create Azure role assignments (`Owner` or `User Access Administrator`) on your target scope.

Use the Foundry Local application's client ID for `foundryClientId`. It identifies the token audience, not the managed identity principal ID of the caller.

For background on how these authorization checks work, see [Required roles for Foundry Local inference](deploy-overview.md#required-roles-for-foundry-local-inference). For more information about Foundry Local authentication, see [Configure authentication for Foundry Local](/azure/azure-sovereign-clouds/private/foundry-local/how-to-configure-authentication).

## Configure Azure RBAC role assignments

Configure Azure role assignments so that:

- The Agents and Tools with Foundry Local extension managed identity can call the Foundry inference endpoint.
- The connected cluster and Foundry operator managed identities can read role assignments for Foundry's ARM RBAC validation.

### Choose when to assign roles

You can assign these roles as you create identities during deployment, or assign them together after you deploy all extensions:

- Assign `Reader` for the connected cluster managed identity after `az connectedk8s connect`.
- Assign `Reader` for the Foundry operator managed identity after you deploy the inference-operator extension.
- Assign `Reader` and the inference role for the Agents and Tools managed identity after you deploy the Agents and Tools with Foundry Local extension.

Use the connected cluster scope for the ARM `Reader` assignments. Use the narrower Foundry extension scope for the inference role.

### Set scopes and principal IDs

Run the following commands in Bash. Replace the placeholder values with values from your environment.

```azurecli
RESOURCE_GROUP="<resource_group>"
CLUSTER_NAME="<cluster_name>"
EXTENSION_NAME="<extension_name>"
FOUNDRY_EXTENSION="<foundry_extension_name>"

CLUSTER_SCOPE=$(az connectedk8s show \
    --resource-group "$RESOURCE_GROUP" \
    --name "$CLUSTER_NAME" \
    --query id -o tsv)
FOUNDRY_SCOPE=$(az k8s-extension show \
    --resource-group "$RESOURCE_GROUP" \
    --cluster-name "$CLUSTER_NAME" \
    --cluster-type connectedClusters \
    --name "$FOUNDRY_EXTENSION" \
    --query id -o tsv)

CLUSTER_PRINCIPAL_ID=$(az connectedk8s show \
    --resource-group "$RESOURCE_GROUP" \
    --name "$CLUSTER_NAME" \
    --query identity.principalId -o tsv)
FOUNDRY_PRINCIPAL_ID=$(az k8s-extension show \
    --resource-group "$RESOURCE_GROUP" \
    --cluster-name "$CLUSTER_NAME" \
    --cluster-type connectedClusters \
    --name "$FOUNDRY_EXTENSION" \
    --query identity.principalId -o tsv)
EXTENSION_PRINCIPAL_ID=$(az k8s-extension show \
    --resource-group "$RESOURCE_GROUP" \
    --cluster-name "$CLUSTER_NAME" \
    --cluster-type connectedClusters \
    --name "$EXTENSION_NAME" \
    --query identity.principalId -o tsv)

if [[ -z "$CLUSTER_SCOPE" || -z "$FOUNDRY_SCOPE" || -z "$CLUSTER_PRINCIPAL_ID" || \
      -z "$FOUNDRY_PRINCIPAL_ID" || -z "$EXTENSION_PRINCIPAL_ID" ]]; then
    echo "Couldn't resolve the resource IDs or managed identity principal IDs." >&2
    exit 1
fi
```

### Assign roles

After you set the scopes and principal IDs, run the following commands:

```azurecli
# Reader for the connected cluster identity
az role assignment create \
    --assignee-object-id "$CLUSTER_PRINCIPAL_ID" \
    --role "Reader" \
    --scope "$CLUSTER_SCOPE" \
    --assignee-principal-type ServicePrincipal

# Reader for the Foundry operator identity
az role assignment create \
    --assignee-object-id "$FOUNDRY_PRINCIPAL_ID" \
    --role "Reader" \
    --scope "$CLUSTER_SCOPE" \
    --assignee-principal-type ServicePrincipal

# Reader for the Agents and Tools with Foundry Local identity
az role assignment create \
    --assignee-object-id "$EXTENSION_PRINCIPAL_ID" \
    --role "Reader" \
    --scope "$CLUSTER_SCOPE" \
    --assignee-principal-type ServicePrincipal

# Use the narrow inference role.
INFERENCE_ROLE=$(az role definition list \
    --name "Cognitive Services OpenAI User" \
    --query "[0].name" -o tsv)

if [ $? -ne 0 ]; then
  echo "Failed to query the inference role." >&2
  exit 1
fi

if [ -z "$INFERENCE_ROLE" ]; then
  echo "Cognitive Services OpenAI User isn't published in this cloud. For disconnected deployment guidance, see the disconnected installation article." >&2
  exit 1
fi

az role assignment create \
    --assignee-object-id "$EXTENSION_PRINCIPAL_ID" \
    --role "$INFERENCE_ROLE" \
    --scope "$FOUNDRY_SCOPE" \
    --assignee-principal-type ServicePrincipal
```

Disconnected deployments use `Contributor` on the Foundry extension resource as the inference role. For those steps, see [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md#grant-inference-permission-managed-identity-auth-only).

## Prepare Agent Service model access

For Agent Service, [create a model connection](create-manage-agents.md#create-a-model-connection) with `ManagedIdentity` authentication, and have a cluster administrator [approve the endpoint](create-manage-agents.md#approve-endpoints-for-managed-identity). Azure role assignments don't replace endpoint approval.

## Handle role propagation delays

Role assignments can take 5 to 30 minutes to propagate across Azure infrastructure. If you still get `401` or `403` errors after assigning roles, wait 10 to 15 minutes and then restart the affected pods.

```bash
kubectl -n foundry-local-operator rollout restart deployment/gpt-oss-20b

for deployment in \
    inferencingflow-deployment \
    vectordb-api-server-deployment \
    agents-management-deployment \
    agents-execution-deployment \
    kb-execution-deployment; do
  if kubectl -n arc-rag get deployment "$deployment" >/dev/null 2>&1; then
    kubectl -n arc-rag rollout restart deployment "$deployment"
    kubectl -n arc-rag rollout status deployment "$deployment"
  fi
done
```

Don't rerun the role assignment commands. The assignments are recorded immediately, but downstream services might not see them until propagation completes.

If calls still fail after propagation, use the error to identify what to check:

| Error | Check |
| --- | --- |
| `401` with `Token validation failed` | The token audience matches the Foundry Local application client ID (`foundryClientId`), and the token isn't expired or invalid. |
| `403` with `insufficient_permissions` | The extension identity has the inference role at the Foundry extension scope: `Cognitive Services OpenAI User` in connected deployments, or `Contributor` in disconnected deployments. |
| `500` with `rbac_check_unavailable` | The connected cluster and Foundry operator identities have `Reader`, and Foundry can reach Azure Resource Manager. |

## Related content

- [Deploy the extension for Agents and Tools with Foundry Local](deploy.md)
- [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md)
- [Configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md)
- [Configure authentication for BYOM](configure-endpoint-authentication.md)
- [Deployment parameter reference and troubleshooting](deploy-reference.md)
