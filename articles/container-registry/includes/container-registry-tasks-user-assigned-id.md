---
title: include file
description: include file
services: container-registry
author: chasedmicrosoft

ms.service: azure-container-registry
ms.topic: include
ms.date: 07/12/2019
ms.author: doveychase
ms.custom: include file, devx-track-azurecli
# Customer intent: As a cloud administrator, I want to create a user-assigned identity using CLI commands, so that I can manage access and permissions for my Azure Container Registry effectively.
---
### Create a user-assigned identity

Create an identity named *myACRTasksId* in your subscription using the [az identity create][az-identity-create] command. You can use the same resource group you used previously to create a container registry, or a different one.

```azurecli
az identity create \
  --resource-group myResourceGroup \
  --name myACRTasksId
```

To configure the user-assigned identity in the following steps, use the [az identity show][az-identity-show] command to store the identity's resource ID, principal ID, and client ID in variables.

```azurecli
# Get resource ID of the user-assigned identity
resourceID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACRTasksId \
  --query id --output tsv)

# Get principal ID of the task's user-assigned identity
principalID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACRTasksId \
  --query principalId --output tsv)

# Get client ID of the user-assigned identity
clientID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACRTasksId \
  --query clientId --output tsv)
```

<!-- LINKS - Internal -->
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-identity-show]: /cli/azure/identity#az_identity_show
