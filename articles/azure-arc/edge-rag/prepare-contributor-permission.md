---
title: Verify Contributor Role for Agents and Tools with Foundry Local
description: "Learn how to verify you have the necessary Contributor role in Azure for Agents and Tools with Foundry Local deployment. Make sure you have proper permissions before continuing."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 06/20/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As an Azure administrator, I want to verify that I have contributor permissions for Agents and Tools with Foundry Local so that I have the necessary access to deploy the resources needed for Agents and Tools with Foundry Local.
---

# Verify contributor role for Agents and Tools with Foundry Local

Make sure you have contributor permissions at the subscription level for your Agents and Tools with Foundry Local deployment by checking your Azure role assignments. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Verify contributor permissions

Verify that you have [Contributor](/azure/role-based-access-control/built-in-roles/privileged#contributor) permissions for the subscription you want to use. See [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

Contributor role is required at the subscription level for the following commands:

```powershell
az provider register --namespace Microsoft.KubernetesConfiguration
az feature register --namespace Microsoft.KubernetesConfiguration --name extensions
```

## Next step

> [!div class="nextstepaction"]
> [Choose the right language model](prepare-language-model.md)