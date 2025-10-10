---
title: include file
description: include file
services: container-registry
author: rayoef

ms.service: azure-container-registry
ms.topic: include
ms.date: 07/06/2020
ms.author: rayoflores
ms.custom: include file
# Customer intent: "As a cloud administrator, I want to retrieve the principal ID of a system-assigned identity from Azure Container Registry tasks, so that I can use it for subsequent command executions and manage access effectively."
---
In the command output, the `identity` section shows an identity of type `SystemAssigned` is set in the task. The `principalId` is the principal ID of the task identity:

```console
[...]
  "identity": {
    "principalId": "aaaaaaaa-bbbb-cccc-1111-222222222222",
    "tenantId": "aaaabbbb-0000-cccc-1111-dddd2222eeee",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
  },
  "location": "eastus",
[...]
``` 
Use the [az acr task show][az-acr-task-show] command to store the principalId in a variable, to use in later commands. Substitute the name of your task and your registry in the following command:

```azurecli
principalID=$(az acr task show \
  --name <task_name> --registry <registry_name> \
  --query identity.principalId --output tsv)
```

<!-- LINKS - Internal -->
[az-acr-task-show]: /cli/azure/acr/task#az_acr_task_show
