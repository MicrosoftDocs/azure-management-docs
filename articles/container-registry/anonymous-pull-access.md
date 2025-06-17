---
title: Enable Unauthenticated Anonymous Pull Access in Azure Container Registry
description: Optionally enable unauthenticated anonymous pull access to make content in your Azure container registry publicly available
ms.topic: how-to
author: chasedmicrosoft
ms.author: doveychase
ms.service: azure-container-registry
ms.date: 02/28/2025
# Customer intent: As a cloud administrator, I want to enable unauthenticated anonymous pull access in Azure Container Registry so that I can distribute public container images without requiring user authentication.
---

# Unauthenticated anonymous pull access

By default, access to pull or push content from an Azure container registry is only available to [authenticated](container-registry-authentication.md) users. Enabling anonymous (unauthenticated) pull access makes all registry content publicly available for read (pull) actions. Use anonymous pull access in scenarios that don't require user authentication, such as distributing public container images.

Anonymous pull access is a preview feature, available in the Standard and Premium [service tiers](container-registry-skus.md). To configure anonymous pull access, update a registry using the Azure CLI (version 2.21.0 or later). For information about installing or upgrading, see [Install Azure CLI](/cli/azure/install-azure-cli).

- Enable anonymous pull access by updating the properties of an existing registry.
- After enabling anonymous pull access, you can disable that access at any time.
- Only data-plane operations are available to unauthenticated clients.
- The registry might throttle a high rate of unauthenticated requests.
- If you previously authenticated to the registry, make sure you clear the credentials before attempting an anonymous pull operation.

> [!WARNING]
> Anonymous pull access currently applies to all repositories in the registry. If you manage repository access using [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md) or [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md), all users can still pull from those repositories in a registry enabled for unauthenticated anonymous pull. Please be aware of this when enabling unauthenticated anonymous pull access.

## Configure anonymous pull access 

Users can enable, disable, and query the status of anonymous pull access using the Azure CLI. The following examples demonstrate how to enable, disable, and query the status of anonymous pull access.

### Enable anonymous pull access

Update a registry using the [az acr update](/cli/azure/acr#az-acr-update) command and pass the `--anonymous-pull-enabled` parameter. By default, anonymous pull is disabled in the registry.
          
```azurecli
az acr update --name myregistry --anonymous-pull-enabled
``` 

> [!IMPORTANT]
> If you previously authenticated to the registry with Docker credentials, run `docker logout` to ensure that you clear the existing credentials before attempting anonymous pull operations. Otherwise, you might see an error message similar to "pull access denied".
> Remember to always specify the fully qualified registry name (all lowercase) when using `docker login` and tagging images for pushing to your registry. In the examples provided, the fully qualified name is `myregistry.azurecr.io`.

If you previously authenticated to the registry with Docker credentials, run the following command to clear existing credentials or any previous authentication.
 
   ```azurecli
      docker logout myregistry.azurecr.io
   ```

This step helps you attempt an anonymous pull operation. If you encounter any issues, you might see an error message similar to "pull access denied."


### Disable anonymous pull access

Disable anonymous pull access by setting `--anonymous-pull-enabled` to `false`.

```azurecli
az acr update --name myregistry --anonymous-pull-enabled false
```

### Query the status of anonymous pull access

You can query the status of "anonymous-pull" using the [az acr show command][az-acr-show] with the `--query` parameter. Here's an example:

```azurecli-interactive
az acr show -n <registry_name> --query anonymousPullEnabled
```

The command returns a boolean value indicating whether "Anonymous Pull" is enabled (`true`) or disabled (`false`). This command streamlines the process of verifying the status of features within ACR.

## Next steps

* Learn about using [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
* Learn about using [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md).
* Learn about options to [authenticate](container-registry-authentication.md) to an Azure container registry.


[az-acr-show]: /cli/azure/acr#az-acr-show