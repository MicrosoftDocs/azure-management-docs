---
title: "Enable artifact cache in your Azure Container Registry with Azure CLI"
description: "Learn how to use Azure CLI to cache container images in Azure Container Registry, improving performance and efficiency."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to
ms.custom: devx-track-azurecli
ms.date: 02/28/2025
ai-usage: ai-assisted
#customer intent: As a developer, I want Artifact cache capabilities so that I can efficiently deliver and serve containerized applications to end-users in real-time.
---

# Enable artifact cache in your Azure Container Registry with Azure CLI

In this article, you learn how to use Azure CLI to enable the [artifact cache feature](artifact-cache-overview.md) in your Azure Container Registry (ACR) with or without authentication.

In addition to the prerequisites listed here, you need an Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## Prerequisites

* Azure CLI. You can use the [Azure Cloud Shell][Azure Cloud Shell] or a local installation of Azure CLI to run the commands in this article. If you want to use it locally, Azure CLI version 2.46.0 or later is required. To confirm your Azure CLI version, run `az --version`. To install or upgrade, see [Install Azure CLI][Install Azure CLI].
* An existing ACR instance. If you don't already have one, [use our quickstart to create a new container registry](/azure/container-registry/container-registry-get-started-azure-cli).
* An existing Key Vault to [create and store credentials][create-and-store-keyvault-credentials].
* Permissions to [set and retrieve secrets from your Key Vault][set-and-retrieve-a-secret].

In this article, we use an example ACR instance named `MyRegistry`.

## Create the credentials

Before configuring the credentials, make sure you're able to [create and store secrets in the Azure Key Vault][create-and-store-keyvault-credentials] and [retrieve secrets from the Key Vault.][set-and-retrieve-a-secret].

1. Run [`az acr credential set create`][az-acr-credential-set-create]:

   ```azurecli-interactive
   az acr credential-set create 
   -r MyRegistry \
   -n MyDockerHubCredSet \
   -l docker.io \ 
   -u https://MyKeyvault.vault.azure.net/secrets/usernamesecret \
   -p https://MyKeyvault.vault.azure.net/secrets/passwordsecret
   ```

1. Run [`az acr credential set update`][az-acr-credential-set-update] to update the username or password Key Vault secret ID on the credential set:

   ```azurecli-interactive
   az acr credential-set update -r MyRegistry -n MyDockerHubCredSet -p https://MyKeyvault.vault.azure.net/secrets/newsecretname
   ```

1. Run [az acr credential-set show][az-acr-credential-set-show] to show credentials:

   ```azurecli-interactive
   az acr credential-set show -r MyRegistry -n MyDockerHubCredSet
   ```

## Create a cache rule

Next, create and configure the cache rule that pulls artifacts from the repository into your cache.

1. To create a new cache rule, run [`az acr cache create`][az-acr-cache-create]:

   ```azurecli-interactive
   az acr cache create -r MyRegistry -n MyRule -s docker.io/library/ubuntu -t ubuntu -c MyDockerHubCredSet
   ```

1. To update credentials on the cache rule, run [`az acr cache update`][az-acr-cache-update]:

    ```azurecli-interactive
    az acr cache update -r MyRegistry -n MyRule -c NewCredSet
    ```

    If you need to remove the credentials, run `az acr cache update -r MyRegistry -n MyRule --remove-cred-set`.

1. To show cache rules, run [`az acr cache show`][az-acr-cache-show]:

    ```azurecli-interactive
     az acr cache show -r MyRegistry -n MyRule
    ```

> [!TIP]
> To create a cache rule without using credentials, use the same command without credentials specified. For example, `az acr cache create -r MyRegistry -n MyRule -s docker.io/library/ubuntu -t ubuntu`. For some sources, such as Docker Hub, credentials are required to create a cache rule.

## Assign permissions to Key Vault using access policies

You can use access policies to assign the appropriate permissions to users so they can access the Azure KeyVault.

1. Get the principal ID of the system identity in use to access Key Vault:

   ```azurecli
   PRINCIPAL_ID=$(az acr credential-set show 
                   -n MyDockerHubCredSet \ 
                   -r MyRegistry  \
                   --query 'identity.principalId' \ 
                   -o tsv) 
   ```

1. Run the `az keyvault set-policy` command to assign access to the Key Vault before pulling the image. For example, to assign permissions for the credentials to access the KeyVault secret:

   ```azurecli
   az keyvault set-policy --name MyKeyVault \
   --object-id $PRINCIPAL_ID \
   --secret-permissions get
   ```

## Pull your image

To pull an image from your cache, use the Docker command and provide the registry sign-in server name, repository name, and its desired tag. For example, to pull an image from the repository `hello-world` with desired tag `latest` for the registry sign-in server `myregistry.azurecr.io`, run:

```azurecli-interactive
 docker pull myregistry.azurecr.io/hello-world:latest
```

## Clean up resources

When no longer needed, delete the cache rule and credentials that you created.

1. To delete the cache rule, run [`az acr cache delete`][az-acr-cache-delete]:

   ```azurecli-interactive
   az acr cache delete -r MyRegistry -n MyRule
   ```

1. To delete the credentials, run [`az acr credential-set delete`][az-acr-credential-set-delete]:

   ```azurecli-interactive
   az acr credential-set delete -r MyRegistry -n MyDockerHubCredSet
   ```

## Next steps

* Learn about [troubleshooting issues with artifact caching](troubleshoot-artifact-cache.md).
* Learn how to [enable artifact cache using the Azure portal](artifact-cache-portal.md).

<!-- LINKS - External -->
[create-and-store-keyvault-credentials]: /azure/key-vault/secrets/quick-create-cli#add-a-secret-to-key-vault
[set-and-retrieve-a-secret]: /azure/key-vault/secrets/quick-create-cli#retrieve-a-secret-from-key-vault
[Install Azure CLI]: /cli/azure/install-azure-cli
[Azure Cloud Shell]: /azure/cloud-shell/quickstart
[az-acr-cache-create]:/cli/azure/acr/cache#az-acr-cache-create
[az-acr-cache-show]:/cli/azure/acr/cache#az-acr-cache-show
[az-acr-cache-delete]:/cli/azure/acr/cache#az-acr-cache-delete
[az-acr-cache-update]:/cli/azure/acr/cache#az-acr-cache-update
[az-acr-credential-set-create]:/cli/azure/acr/credential-set#az-acr-credential-set-create
[az-acr-credential-set-update]:/cli/azure/acr/credential-set#az-acr-credential-set-update
[az-acr-credential-set-show]: /cli/azure/acr/credential-set#az-acr-credential-set-show
[az-acr-credential-set-delete]: /cli/azure/acr/credential-set#az-acr-credential-set-delete
