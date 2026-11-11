---
title: Configure "BYOM" Endpoint Authentication for Agentic Retrieval in Foundry Local
description: "Learn how to configure API-key based authentication for Agentic Retrieval in Foundry Local to securely manage and access resources across environments."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/26/2026
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to configure API-key based authentication for Agentic Retrieval in Foundry Local, so that I can securely manage and access resources across cloud environments.
---

# Configure "BYOM" endpoint authentication for Agentic Retrieval in Foundry Local

This article shows you how to configure API-key based authentication for any cloud-based LLM endpoints that need it. If you configured Agentic Retrieval to use your own language model (BYOM), complete the steps in this article.

This article applies only when using BYOM (cloud) endpoints (`useFoundryLocal=false`). If you use Foundry Local with `foundryClientId`, authentication is handled via managed identity and no API key configuration is needed. For managed identity role assignments after deployment, see [Configure Foundry Local inference authentication for Agentic Retrieval](configure-foundry-inference-authentication.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Set up API Key for authentication (BYOM)

After you install the Agentic Retrieval extension and configure it to use your own language model, get an API key for the model.

1. In your Azure Local node, get the "bring your own model" (BYOM) secret that was created during the extension installation.

   ```powershell
   [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String(
   (.\kubectl.exe get secret byom-api-key -n arc-rag -o jsonpath="{.data.BYOM_API_KEY}")
   
   )) 
 
   Output:  
   byom-secret 
   ```

1. Update the secret value to LLM endpoint API key by deleting and recreating the secret. 

   ```powershell
   kubectl delete secret byom-api-key -n arc-rag 
   
   $apiKey = "<LLM endpoint API key>" 
   
   kubectl create secret generic byom-api-key --from-literal=BYOM_API_KEY=$apiKey -n arc-rag 
    ```

1. Verify if the secret is set to desired API key 

   ```powershell
   [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String( 
   
       ( kubectl get secret byom-api-key -n arc-rag -o jsonpath="{.data.BYOM_API_KEY}" ) 
   
   )) 
 
   Output: 
   <Endpoint api key>
   ```

1. Delete the inferencing flow pod to apply the secret change.

   ```powershell
   kubectl.exe delete pods -n arc-rag -l app=inferencingflow 
   ```

## Related content

- [Deploy the Agentic Retrieval extension](deploy.md)
- [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md)
