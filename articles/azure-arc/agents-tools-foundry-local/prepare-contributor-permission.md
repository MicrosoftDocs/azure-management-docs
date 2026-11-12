---
title: Verify Contributor permissions
description: "Learn how to verify that you have the Azure Contributor role required to deploy Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 10/29/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As an Azure administrator, I want to verify that I have contributor permissions for Agents and Tools with Foundry Local so that I have the necessary access to deploy the resources needed for Agents and Tools with Foundry Local.
---

# Verify Contributor role for Agents and Tools with Foundry Local
To deploy the extension, you need Contributor permissions at the subscription level. Check your Azure role assignments to ensure you have the right permissions. This article is part of the deployment prerequisites checklist.

## Check your role assignment
Verify that you have the [Contributor](/azure/role-based-access-control/built-in-roles/privileged#contributor) role for the subscription you want to use. For more information, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

You need the Contributor role at the subscription level for the following commands:

```powershell
az provider register --namespace Microsoft.KubernetesConfiguration
az feature register --namespace Microsoft.KubernetesConfiguration --name extensions
```

## Next step

> [!div class="nextstepaction"]
> [Choose the right language model](prepare-language-model.md)
