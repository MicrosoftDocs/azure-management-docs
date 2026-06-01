---
title: Configure Foundry Local Inference Authentication for Agentic Retrieval
description: Learn how to configure post-deployment managed identity authentication for Foundry Local inference in Agentic Retrieval by assigning app roles and Azure RBAC roles.
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/01/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to configure Foundry Local inference authentication after deploying Agentic Retrieval so that model inference requests are authorized and reliable.
---

# Configure Foundry Local inference authentication for Agentic Retrieval

After you deploy the Agentic Retrieval extension with `foundryClientId`, configure the Foundry Local managed identity authorization layers in this article.

Use this article only for Foundry Local managed identity authentication. For bring-your-own-model API key authentication, see [Configure authentication for BYOM](configure-endpoint-authentication.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, make sure you:

- [Register a Foundry Local application](prepare-authentication.md#optional-register-a-foundry-local-application).
- [Deploy the extension for Agentic Retrieval in Foundry Local](deploy.md).
- Have permissions to create Azure role assignments (`Owner` or `User Access Administrator`) on your target scope.
- Permissions to assign app roles in Microsoft Entra ID for the Foundry Local app registration.

For background on how these authorization checks work, see [Foundry Local inference authentication layers](deploy-overview.md#foundry-local-inference-authentication-layers).

## Assign the Foundry app role (Layer 1)

After you deploy the Agents and Tools extension and its managed identity principal ID exists, assign the `FoundryInferenceAccess` app role in your Foundry Local app registration to the extension managed identity.

For detailed app role assignment steps, see [Configure authentication for Foundry Local](/azure/azure-sovereign-clouds/private/foundry-local/how-to-configure-authentication).

## Configure Azure RBAC role assignments (Layers 2 and 3)

Configure Azure role assignments so that:

- The Agents and Tools extension managed identity can call the Foundry inference endpoint.
- The Foundry operator can perform ARM RBAC validation.

### Choose when to assign roles

You can assign these roles as you create identities during deployment, or assign them together after you deploy all extensions:

- Assign Layer 2 (`Reader` for the connected cluster managed identity) after `az connectedk8s connect`.
- Assign Layer 3 (`Reader` for the Foundry operator managed identity) after you deploy the inference-operator extension.
- Assign Layer 3 (`Cognitive Services OpenAI User` and `Reader` for the Agents and Tools managed identity) after you deploy the Agents and Tools extension.

You can apply these role assignments at either subscription or resource group scope for broader coverage, or at connected cluster resource scope for least privilege. Choose the scope option that best fits your deployment in the following sections.

### Set scope and principal IDs

Set your scope and principal IDs by using one of the following options.
Replace placeholder values in angle brackets, such as `<subscription_id>`, `<resource_group>`, and `<cluster_name>`, with values from your environment where applicable.

#### Option 1: Subscription or resource group scope

Use this option for broader coverage and simpler setup.

```azurecli
# Set subscription or resource group scope
SCOPE="/subscriptions/<subscription_id>"

# Or use resource group scope
# SCOPE="/subscriptions/<subscription_id>/resourceGroups/<resource_group>"

# Set managed identity principal IDs
FOUNDRY_PRINCIPAL_ID="<foundry_app_principal_id>"
EXTENSION_PRINCIPAL_ID="<agents_and_tools_app_principal_id>"
```

#### Option 2: Connected cluster resource scope (least privilege)

Use this option for least-privilege access scoped to one connected cluster.

```azurecli
# Set connected cluster resource scope
SCOPE=$(az connectedk8s show -g <resource_group> -n <cluster_name> --query "id" -o tsv)

# Get managed identity principal IDs
FOUNDRY_PRINCIPAL_ID=$(az k8s-extension show -g <resource_group> -c <cluster_name> \
    -t connectedClusters --name inference-operator --query "identity.principalId" -o tsv)
EXTENSION_PRINCIPAL_ID=$(az k8s-extension show -g <resource_group> -c <cluster_name> \
    -t connectedClusters --name <extension_name> --query "identity.principalId" -o tsv)
```

### Assign roles by using the selected scope

After you set `SCOPE`, `FOUNDRY_PRINCIPAL_ID`, and `EXTENSION_PRINCIPAL_ID`, run the following commands:

```azurecli
# Assign roles at the selected scope

# 1. Reader role for Foundry operator identity (required for ARM RBAC validation)
az role assignment create \
    --assignee-object-id $FOUNDRY_PRINCIPAL_ID \
    --role "Reader" \
    --scope $SCOPE \
    --assignee-principal-type ServicePrincipal

# 2. Cognitive Services OpenAI User role for Agents and Tools identity (for model inference)
az role assignment create \
    --assignee-object-id $EXTENSION_PRINCIPAL_ID \
    --role "Cognitive Services OpenAI User" \
    --scope $SCOPE \
    --assignee-principal-type ServicePrincipal

# 3. Reader role for Agents and Tools identity (for managed identity token ARM RBAC checks)
az role assignment create \
    --assignee-object-id $EXTENSION_PRINCIPAL_ID \
    --role "Reader" \
    --scope $SCOPE \
    --assignee-principal-type ServicePrincipal
```

## Handle role propagation delays

Role assignments can take 5 to 30 minutes to propagate across Azure infrastructure. If you still get `401` or `403` errors after assigning roles, wait 10 to 15 minutes and then restart the affected pods.

```bash
kubectl -n arc-rag delete azureclusteridentityrequests --all
kubectl -n arc-rag rollout restart deployment \
    inferencingflow-deployment agents-runtime-deployment \
    agents-manager-deployment
```

Don't rerun the role assignment commands. The assignments are recorded immediately, but downstream services might not see them until propagation completes.

## Related content

- [Deploy the extension for Agentic Retrieval in Foundry Local](deploy.md)
- [Configure authentication for Agentic Retrieval in Foundry Local](prepare-authentication.md)
- [Configure authentication for BYOM](configure-endpoint-authentication.md)
- [Deployment parameter reference and troubleshooting](deploy-reference.md)
