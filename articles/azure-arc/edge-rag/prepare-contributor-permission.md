---
title: Verify "Contributor" Role for Edge RAG Deployment
description: "Learn how to verify you have the necessary Contributor role in Azure for Edge RAG deployment. Make sure you have proper permissions before continuing."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/21/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As an Azure administrator, I want to verify that I have contributor permissions for Edge RAG so that I have the necessary access to deploy the resources needed for Edge RAG.
---

# Verify "contributor" role at the subscription level

In this article, verify that you have contributor permissions at the subscription level for your Edge RAG deployment by checking your Azure role assignments and running required registration commands. This article is part of the deployment prerequisites checklist.

## Verify contributor permissions

Verify that you have [Contributor](/azure/role-based-access-control/built-in-roles/privileged#contributor) permissions for the subscription you want to use. See [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

Contributor role is required at the subscription level for the following commands:

```powershell
az provider register --namespace Microsoft.KubernetesConfiguration
az feature register --namespace Microsoft.KubernetesConfiguration --name extensions
```

## Next step

> [!div class="nextstepaction"]
> [Choose the right language model for Edge RAG deployment](prepare-choose-language-model.md)