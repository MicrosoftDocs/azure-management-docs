---
title: include file
description: include file
services: container-registry
author: rayoef

ms.service: azure-container-registry
ms.topic: include
ms.date: 07/12/2019
ms.author: rayoflores
ms.custom: include file
# Customer intent: "As a cloud administrator, I want to configure a User Assigned identity in my container registry tasks, so that I can securely manage access and authentication for my applications."
---
In the command output, the `identity` section shows the identity of type `UserAssigned` is set in the task:

```console
[...]
"identity": {
    "principalId": null,
    "tenantId": null,
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACRTasksId": {
        "clientId": "00001111-aaaa-2222-bbbb-3333cccc4444",
        "principalId": "aaaaaaaa-bbbb-cccc-1111-222222222222"
      }
[...]
``` 
<!-- LINKS - Internal -->
