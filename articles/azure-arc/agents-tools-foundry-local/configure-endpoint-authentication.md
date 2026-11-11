---
title: Configure "BYOM" Endpoint Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure API-key based authentication for Agents and Tools with Foundry Local to securely manage and access resources across environments."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to configure API-key based authentication for Agents and Tools with Foundry Local, so that I can securely manage and access resources across cloud environments.
---

# Configure "BYOM" endpoint authentication for Agents and Tools with Foundry Local

This article shows you how to configure API key–based authentication for a model endpoint. Complete these steps when Agents and Tools with Foundry Local uses the shared `byom-api-key` Secret.

Use API-key authentication for an external BYOM endpoint or for Foundry Local when `foundryClientId` isn't configured. If you configure `foundryClientId`, managed identity authentication is required and the service doesn't fall back to the API key. For managed identity role assignments after deployment, see [Configure Foundry Local inference authentication for Agents and Tools with Foundry Local](configure-foundry-inference-authentication.md).

> [!IMPORTANT]
> The `byom.*` settings and the `byom-api-key` Secret are used only by classic chat in the developer portal. Classic chat is deprecated, and its APIs accept requests only from the developer portal. Agents don't use this key. They use [model connections](create-manage-agents.md). Configure agent model credentials in each model connection, not with this procedure.

## Set up API key for authentication (BYOM)

After you install the Agents and Tools with Foundry Local extension and configure it to use your own language model, get an API key for the model.

1. Set the new API key in your PowerShell session.

   ```powershell
   $apiKey = "<LLM-endpoint-API-key>"
   ```

1. Update the Secret without printing the key.

   ```powershell
   kubectl create secret generic byom-api-key `
     --from-literal=BYOM_API_KEY=$apiKey `
     --namespace arc-rag `
     --dry-run=client `
     --output yaml |
     kubectl apply --filename -
   ```

1. Verify that the stored value matches the key without displaying either value.

   ```powershell
   $storedValue = kubectl get secret byom-api-key `
     --namespace arc-rag `
     --output jsonpath="{.data.BYOM_API_KEY}"
   $expectedValue = [Convert]::ToBase64String(
     [Text.Encoding]::UTF8.GetBytes($apiKey)
   )

   if ($storedValue -ne $expectedValue) {
     throw "The byom-api-key Secret doesn't contain the expected value."
   }
   ```

1. Restart the deployments that read the Secret, and wait for each rollout to finish.

   ```powershell
   $deployments = @(
     "inferencingflow-deployment",
     "vectordb-api-server-deployment"
   )

   foreach ($deployment in $deployments) {
     if (kubectl get deployment $deployment --namespace arc-rag --ignore-not-found) {
       kubectl rollout restart deployment/$deployment --namespace arc-rag
       kubectl rollout status deployment/$deployment --namespace arc-rag
     }
   }
   ```

## Related content

- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
- [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md)
