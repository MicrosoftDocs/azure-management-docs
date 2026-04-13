---
title: Enable artifact cache to cache artifacts from another Azure Container Registry
description: Learn how to cache container images from one Azure Container Registry in another using managed identity authentication with Azure CLI and Bicep.
author:      toddysm # GitHub alias
ms.author:   memladen # Microsoft alias
ms.service: azure-container-registry
ms.topic: how-to
ms.custom: devx-track-azurecli, devx-track-bicep
ms.date:     04/06/2026
---

# Enable artifact cache to cache artifacts from another Azure Container Registry

In this article, you will learn how to enable the [artifact cache feature](../artifact-cache-overview.md) in your Azure Container Registry (ACR) to cache images from another Azure Container Registry. The downstream registry authenticates to the upstream registry using a managed identity, so you don't need to store credentials in Azure Key Vault.

In addition to the prerequisites listed here, you need an Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).

## Prerequisites

* An existing downstream ACR instance (referred to as `MyRegistry` in this article). If you don't already have one, [create a new container registry](/azure/container-registry/container-registry-get-started-azure-cli).
* An existing upstream ACR instance (referred to as `UpstreamRegistry` in this article) that contains the artifacts you want to cache.
* Azure CLI version 2.85.0 or later. You can use the [Azure Cloud Shell][Azure Cloud Shell] or a local installation. To confirm your version, run `az --version`. To install or upgrade, see [Install Azure CLI][Install Azure CLI].
* [Bicep tools](/azure/azure-resource-manager/bicep/install) (for Bicep deployments).

## Configure artifact cache

### [Azure CLI](#tab/azure-cli)

#### Create a user-assigned managed identity

ACR-to-ACR caching authenticates to the upstream registry using a user-assigned managed identity instead of username and password credentials stored in Key Vault. You need to create a managed identity and grant it pull access to the upstream registry.

1. Create a user-assigned managed identity:

   ```azurecli-interactive
   az identity create \
     --name MyACRCacheIdentity \
     --resource-group MyResourceGroup
   ```

1. Get the principal ID and resource ID of the managed identity:

   ```azurecli-interactive
   IDENTITY_PRINCIPAL_ID=$(az identity show \
     --name MyACRCacheIdentity \
     --resource-group MyResourceGroup \
     --query 'principalId' \
     -o tsv)

   IDENTITY_RESOURCE_ID=$(az identity show \
     --name MyACRCacheIdentity \
     --resource-group MyResourceGroup \
     --query 'id' \
     -o tsv)
   ```

#### Assign pull permissions on the upstream registry

The upstream registry must have [ABAC (Attribute-Based Access Control) enabled](../container-registry-rbac-abac-repository-permissions.md) so you can assign fine-grained repository permissions. The managed identity needs the **Container Registry Repository Reader** role on the upstream registry, scoped to the specific repository you want to cache.

> [!NOTE]
> If the upstream registry doesn't have ABAC enabled yet, run `az acr update --name UpstreamRegistry --role-assignment-mode rbac-abac`.

1. Get the resource ID of the upstream registry:

   ```azurecli-interactive
   UPSTREAM_ID=$(az acr show \
     --name UpstreamRegistry \
     --query 'id' \
     -o tsv)
   ```

1. Assign the **Container Registry Repository Reader** role to the managed identity on the upstream registry:

   ```azurecli-interactive
   az role assignment create \
     --role "Container Registry Repository Reader" \
     --assignee "$IDENTITY_PRINCIPAL_ID" \
     --scope "$UPSTREAM_ID/repositories/myapp"
   ```

#### Create a cache rule

Create a cache rule that pulls artifacts from the upstream registry into your downstream registry.

1. Run [`az acr cache create`][az-acr-cache-create] to create a cache rule. Use the `--identity` parameter to specify the user-assigned managed identity for authentication with the upstream registry:

   ```azurecli-interactive
   az acr cache create \
     -r MyRegistry \
     -n MyRule \
     -s upstreamregistry.azurecr.io/myapp \
     -t myapp \
     --identity "$IDENTITY_RESOURCE_ID"
   ```

1. Run [`az acr cache show`][az-acr-cache-show] to verify the cache rule:

   ```azurecli-interactive
   az acr cache show -r MyRegistry -n MyRule
   ```

### [Bicep](#tab/bicep)

#### Define the Bicep template

Create a Bicep file that defines a user-assigned managed identity and cache rule for ACR-to-ACR caching.

```bicep
@description('Name of the downstream (cache) registry.')
param registryName string

@description('Login server of the upstream registry (for example, upstreamregistry.azurecr.io).')
param upstreamLoginServer string

@description('Repository path on the upstream registry to cache (for example, myapp).')
param sourceRepository string

@description('Repository name in the downstream registry for cached artifacts.')
param targetRepository string

@description('Name of the user-assigned managed identity.')
param identityName string = 'acrCacheIdentity'

@description('Name of the cache rule.')
param cacheRuleName string = 'cacheRule'

@description('Location for resources.')
param location string = resourceGroup().location

resource registry 'Microsoft.ContainerRegistry/registries@2025-11-01' existing = {
  name: registryName
}

resource managedIdentity 'Microsoft.ManagedIdentity/userAssignedIdentities@2023-01-31' = {
  name: identityName
  location: location
}

resource cacheRule 'Microsoft.ContainerRegistry/registries/cacheRules@2026-01-01-preview' = {
  name: '${registry.name}/${cacheRuleName}'
  identity: {
    type: 'UserAssigned'
    userAssignedIdentities: {
      '${managedIdentity.id}': {}
    }
  }
  properties: {
    sourceRepository: '${upstreamLoginServer}/${sourceRepository}'
    targetRepository: targetRepository
  }
}

@description('Principal ID of the managed identity. Grant this identity Container Registry Repository Reader on the upstream registry.')
output identityPrincipalId string = managedIdentity.properties.principalId
```

Save this file as `acr-cache.bicep`.

#### Deploy the template

1. Deploy the Bicep template to create the managed identity and cache rule:

   ```azurecli-interactive
   az deployment group create \
     --resource-group MyResourceGroup \
     --template-file acr-cache.bicep \
     --parameters \
       registryName=MyRegistry \
       upstreamLoginServer=upstreamregistry.azurecr.io \
       sourceRepository=myapp \
       targetRepository=myapp
   ```

1. Note the `identityPrincipalId` value from the deployment output.

1. Assign the **Container Registry Repository Reader** role to the managed identity on the upstream registry:

   > [!NOTE]
   > The upstream registry must have [ABAC enabled](../container-registry-rbac-abac-repository-permissions.md). If it doesn't, run `az acr update --name UpstreamRegistry --role-assignment-mode rbac-abac` first.

   ```azurecli-interactive
   UPSTREAM_ID=$(az acr show --name UpstreamRegistry --query 'id' -o tsv)

   az role assignment create \
     --role "Container Registry Repository Reader" \
     --assignee "<identityPrincipalId>" \
     --scope "$UPSTREAM_ID/repositories/myapp"
   ```

---

## Verify the cache

After you configure the cache rule and assign the required permissions, pull an image from your downstream registry to verify that caching works correctly.

```azurecli-interactive
docker pull myregistry.azurecr.io/myapp:latest
```

The first pull retrieves the image from the upstream registry and caches it in your downstream registry. Subsequent pulls are served directly from the downstream registry cache.

## Clean up resources

When the cache resources are no longer needed, delete the cache rule, role assignment, and managed identity.

1. Delete the cache rule by running [`az acr cache delete`][az-acr-cache-delete]:

   ```azurecli-interactive
   az acr cache delete -r MyRegistry -n <cache-rule-name>
   ```

   Replace `<cache-rule-name>` with the name you used when creating the cache rule (for example, `MyRule` for the Azure CLI tab or `cacheRule` for the Bicep tab).

1. Remove the role assignment on the upstream registry:

   ```azurecli-interactive
   UPSTREAM_ID=$(az acr show --name UpstreamRegistry --query 'id' -o tsv)

   IDENTITY_PRINCIPAL_ID=$(az identity show \
     --name MyACRCacheIdentity \
     --resource-group MyResourceGroup \
     --query 'principalId' \
     -o tsv)

   az role assignment delete \
     --role "Container Registry Repository Reader" \
     --assignee "$IDENTITY_PRINCIPAL_ID" \
     --scope "$UPSTREAM_ID/repositories/myapp"
   ```

1. Delete the user-assigned managed identity:

   ```azurecli-interactive
   az identity delete \
     --name MyACRCacheIdentity \
     --resource-group MyResourceGroup
   ```

   > [!NOTE]
   > Make sure the identity is not in use by any other resources before deleting it.

## Next steps

* Learn about [troubleshooting issues with artifact caching](../troubleshoot-artifact-cache.md).
* Learn about [artifact cache overview](../artifact-cache-overview.md).
* Learn how to [enable artifact cache using the Azure portal](../artifact-cache-portal.md).

<!-- LINKS - External -->
[Install Azure CLI]: /cli/azure/install-azure-cli
[Azure Cloud Shell]: /azure/cloud-shell/quickstart
[az-acr-cache-create]:/cli/azure/acr/cache#az-acr-cache-create
[az-acr-cache-show]:/cli/azure/acr/cache#az-acr-cache-show
[az-acr-cache-delete]:/cli/azure/acr/cache#az-acr-cache-delete
[az-acr-cache-update]:/cli/azure/acr/cache#az-acr-cache-update
