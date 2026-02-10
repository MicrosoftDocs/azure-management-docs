---
title: include file
description: include file
services: container-registry
author: rayoef

ms.service: azure-container-registry
ms.topic: include
ms.date: 02/09/2026
ms.author: rayoflores
ms.custom: include file, devx-track-azurecli
# Customer intent: As a cloud administrator, I want to create and configure a service principal for my container registry, so that I can securely manage access and permissions for my applications and services.
---

## Create a service principal

To create a service principal with access to your container registry, run the following script in the [Azure Cloud Shell](/azure/cloud-shell/overview) or a local installation of the [Azure CLI](/cli/azure/install-azure-cli). The script is formatted for the Bash shell.

Before running the script, update the `ACR_NAME` variable with the name of your container registry. The `SERVICE_PRINCIPAL_NAME` value must be unique within your Microsoft Entra tenant. If you receive an "`'http://acr-service-principal' already exists.`" error, specify a different name for the service principal.

You can optionally modify the `--role` value in the [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] command to grant different permissions. For a complete list of roles, see [Azure Container Registry Entra permissions and roles overview](../container-registry-rbac-built-in-roles-overview.md).

After you run the script, take note of the service principal's **ID** and **password**. Once you have its credentials, you can configure your applications and services to authenticate to your container registry as the service principal.

<!-- https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/create-registry/create-registry-service-principal-assign-role.sh -->
:::code language="azurecli" source="~/azure_cli_scripts/container-registry/create-registry/create-registry-service-principal-assign-role.sh" id="Create":::

### Use an existing service principal

To grant registry access to an existing service principal, you must assign a new role to the service principal. As with creating a new service principal, you must perform role assignments to grant permissions to the service principal. See [Azure Container Registry Entra permissions and roles overview](../container-registry-rbac-built-in-roles-overview.md).

The following script uses the [az role assignment create][az-role-assignment-create] command to grant pull permissions to a service principal you specify in the `SERVICE_PRINCIPAL_ID` variable. Adjust the `--role` value if you'd like to grant a different level of access.

<!-- https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/create-registry/create-registry-service-principal-assign-role.sh -->
:::code language="azurecli" source="~/azure_cli_scripts/container-registry/create-registry/create-registry-service-principal-assign-role.sh" id="Assign":::

<!-- LINKS - Internal -->
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
