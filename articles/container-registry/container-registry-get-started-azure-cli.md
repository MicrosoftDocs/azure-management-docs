---
title: Quickstart - Create Registry - Azure CLI
description: Learn how to create a private Docker container registry using the Azure CLI, push a container image, and pull and run the image from the registry.
ms.topic: quickstart
author: rayoef
ms.author: rayoflores
ms.date: 10/31/2023
ms.service: azure-container-registry
ms.custom: H1Hack27Feb2017, mvc, devx-track-azurecli, mode-api
#customer intent: As a developer, I want to create a private Docker container registry using the Azure CLI so that I can manage container images efficiently.
---
# Quickstart: Create a private container registry using the Azure CLI

Azure Container Registry is a private registry service for building, storing, and managing container images and related artifacts. In this quickstart, you create an Azure container registry instance with the Azure CLI. Then, use Docker commands to push a container image into the registry, and finally pull and run the image from your registry.

This quickstart requires that you are running the Azure CLI (version 2.0.55 or later recommended). Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][azure-cli].

You must also have Docker installed locally. Docker provides packages that easily configure Docker on any [macOS][docker-mac], [Windows][docker-windows], or [Linux][docker-linux] system.

Because the Azure Cloud Shell doesn't include all required Docker components (the `dockerd` daemon), you can't use the Cloud Shell for this quickstart.

## Create a resource group

Create a resource group with the [az group create][az-group-create] command. An Azure resource group is a logical container into which Azure resources are deployed and managed.

The following example creates a resource group named *myResourceGroup* in the *eastus* location.

```azurecli
az group create --name myResourceGroup --location eastus
```

## Configure parameters for a container registry

In this quickstart you create a *Standard* registry, which is sufficient for most Azure Container Registry workflows. For details on available service tiers, see [Container registry service tiers][container-registry-skus].

Create an ACR instance using the [az acr create][az-acr-create] command. The registry name must be unique within Azure, and contain 5-50 lowercase alphanumeric characters. In the following example, *mycontainerregistry* is used. Update this to a unique value.

### Configure role assignment permissions mode

You can optionally use the `--role-assignment-mode` parameter to specify the role assignment mode of the registry.
This option determines how Microsoft Entra role-based access control (RBAC) and role assignments are managed for the registry, including the use of Microsoft Entra attribute-based access control (ABAC) for Microsoft Entra repository permissions.

Specify `rbac-abac` for this parameter to retain standard Microsoft Entra RBAC role assignments, while optionally applying Microsoft Entra ABAC conditions for fine‑grained, repository‑level access control.

For more information on this option, see [Microsoft Entra attribute-based access control (ABAC) for repository permissions](container-registry-rbac-abac-repository-permissions.md).

## Create a container registry

```azurecli
az acr create --resource-group myResourceGroup \
  --name mycontainerregistry --sku Standard \
  --role-assignment-mode 'rbac-abac'
```

When the registry is created, the output is similar to the following:

```json
{
  "adminUserEnabled": false,
  "creationDate": "2019-01-08T22:32:13.175925+00:00",
  "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/mycontainerregistry",
  "location": "eastus",
  "loginServer": "mycontainerregistry.azurecr.io",
  "name": "mycontainerregistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Standard",
    "tier": "Standard"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

Take note of `loginServer` in the output, which is the fully qualified registry name (all lowercase). Throughout the rest of this quickstart `<registry-name>` is a placeholder for the container registry name, and `<login-server>` is a placeholder for the registry's login server name.

[!INCLUDE [container-registry-quickstart-sku](./includes/container-registry-quickstart-sku.md)]

## Log in to registry

Before pushing and pulling container images, you must log in to the registry. To do so, use the [az acr login][az-acr-login] command. Specify only the registry resource name when logging in with the Azure CLI. Don't use the fully qualified login server name. 

```azurecli
az acr login --name <registry-name>
```

Example:

```azurecli
az acr login --name mycontainerregistry
```

The command returns a `Login Succeeded` message once completed.

[!INCLUDE [container-registry-quickstart-docker-push](./includes/container-registry-quickstart-docker-push.md)]

## List container images

The following example lists the repositories in your registry:

```azurecli
az acr repository list --name <registry-name> --output table
```

Output:

```
Result
----------------
hello-world
```

The following example lists the tags on the **hello-world** repository.

```azurecli
az acr repository show-tags --name <registry-name> --repository hello-world --output table
```

Output:

```
Result
--------
v1
```

[!INCLUDE [container-registry-quickstart-docker-pull](./includes/container-registry-quickstart-docker-pull.md)]

## Clean up resources

When no longer needed, you can use the [az group delete][az-group-delete] command to remove the resource group, the container registry, and the container images stored there.

```azurecli
az group delete --name myResourceGroup
```

## Next steps

In this quickstart, you created an Azure Container Registry with the Azure CLI, pushed a container image to the registry, and pulled and ran the image from the registry. Continue to the Azure Container Registry tutorials for a deeper look at ACR.

> [!div class="nextstepaction"]
> [Azure Container Registry tutorials][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Azure Container Registry Tasks tutorials][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-group-create]: /cli/azure/group#az_group_create
[az-group-delete]: /cli/azure/group#az_group_delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
