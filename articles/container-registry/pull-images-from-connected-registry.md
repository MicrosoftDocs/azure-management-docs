---
title: Pull Images from a Connected Registry 
description: Learn how to use Azure Container Registry CLI commands to configure a client token and pull images from a connected registry.
ms.topic: quickstart
author: chasedmicrosoft
ms.author: doveychase
ms.date: 05/20/2025
ms.custom: mode-other, devx-track-azurecli
ms.devlang: azurecli
ms.service: azure-container-registry
# Customer intent: As a cloud developer, I want to configure a client token and pull images from a connected container registry, so that I can efficiently manage and deploy containerized applications in my development environment.
---

# Pull images from a connected registry 

To pull images from a [connected registry](intro-connected-registry.md), configure a [client token](overview-connected-registry-access.md#client-tokens) and pass the token credentials to access registry content.

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]
* Connected registry resource in Azure. For deployment steps, see [Quickstart: Create a connected registry using the Azure CLI][quickstart-connected-registry-cli]. In the commands in this article, the connected registry name is stored in the environment variable *$CONNECTED_REGISTRY_RW*.

## Create a scope map

Use the [az acr scope-map create][az-acr-scope-map-create] command to create a scope map for read access to the `hello-world` repository:

```azurecli
# Use the REGISTRY_NAME variable in the following Azure CLI commands to identify the registry
REGISTRY_NAME=<container-registry-name>

az acr scope-map create \
  --name hello-world-scopemap \
  --registry $REGISTRY_NAME \
  --repository hello-world content/read \
  --description "Scope map for the connected registry."
```

## Create a client token

Use the [az acr token create][az-acr-token-create] command to create a client token and associate it with the newly created scope map:

```azurecli
az acr token create \
  --name myconnectedregistry-client-token \
  --registry $REGISTRY_NAME \
  --scope-map hello-world-scopemap
```

This command returns details about the newly generated token, including passwords.

  > [!IMPORTANT]
  > Make sure that you save the generated passwords. These passwords are one-time passwords and can't be retrieved. You can generate new passwords using the [az acr token credential generate][az-acr-token-credential-generate] command.

## Update the connected registry with the client token

Use the az acr connected-registry update command to update the connected registry with the newly created client token. 

```azurecli
az acr connected-registry update \
  --name $CONNECTED_REGISTRY_RW \
  --registry $REGISTRY_NAME \
  --add-client-token myconnectedregistry-client-token
```

## Pull an image from the connected registry

From a machine with access to connected registry on-premises device, use the following example command to sign into the connected registry, using the client token credentials. For best practices to manage login credentials, see the [docker login](https://docs.docker.com/engine/reference/commandline/login/) command reference.

> [!CAUTION]
> If you set up your connected registry as an insecure registry, update the insecure registries list in the Docker daemon configuration to include the IP address or FQDN (Fully Qualified Domain Name) and port of your connected registry on your device. This configuration should only be used for testing purposes. For more information, see [Test an insecure registry](https://docs.docker.com/registry/insecure/).

```
docker login --username myconnectedregistry-client-token \
  --password <token_password> <IP_address_or_FQDN_of_connected_registry>:<port>
```

Then, use the following command to pull the `hello-world` image:

```
docker pull <IP_address_or_FQDN_of_connected_registry>:<port>/hello-world
```

## Next steps

* Learn more about [non-Microsoft Entra repository-scoped tokens](container-registry-token-based-repository-permissions.md).
* Learn more about [accessing a connected registry](overview-connected-registry-access.md).

<!-- LINKS - internal -->
[az-acr-scope-map-create]: /cli/azure/acr/token/#az_acr_token_create
[az-acr-token-create]: /cli/azure/acr/token/#az_acr_token_create
[az-acr-token-credential-generate]: /cli/azure/acr/token/credential#az_acr_token_credential_generate
[az-acr-connected-registry-update]: ./quickstart-connected-registry-cli.md#az_acr_connected_registry_update] 
[container-registry-intro]: container-registry-intro.md
[quickstart-connected-registry-cli]: quickstart-connected-registry-cli.md