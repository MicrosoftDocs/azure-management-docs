---
title: Prerequisite: Ensure you have "contributor" permission at the subscription level
description: "Verify contributor permissions for Edge RAG deployment."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/06/2025
ai-usage: ai-assisted
---

# Prerequisite: Ensure you have "contributor" permission at the subscription level

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
> [Prerequisite: Choose the right language model](prerequisites-choose-language-model.md)
