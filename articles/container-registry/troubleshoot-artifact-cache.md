---
title: Troubleshoot artifact cache issues in Azure Container Registry
description: Learn how to troubleshoot the most common problems for a registry that uses the Artifact cache feature.
ms.topic: troubleshooting
ms.date: 04/23/2025
author: chasedmicrosoft
ms.author: doveychase
ms.service: azure-container-registry
# Customer intent: As a user managing an Azure Container Registry, I want to troubleshoot common artifact cache issues, so that I can maintain optimal performance and resolve problems effectively.
---

# Troubleshoot artifact cache issues in Azure Container Registry

Learn how to solve the most common problems for a registry enabled with the artifact cache feature by identifying symptoms, causes, and potential solutions.

## Unhealthy credentials

Credentials are a set of Azure Key Vault secrets that operate as a username and password for private repositories. Unhealthy credentials can occur when these secrets are no longer valid. In the Azure portal, you can select the credentials to edit and apply changes.

Verify that secrets in Azure Key Vault are valid, and that access to the Azure Key Vault is assigned.

To assign access to Azure Key Vault, run the following command:

```azurecli-interactive
az keyvault set-policy --name myKeyVaultName --object-id myObjID --secret-permissions get
```

For more information, see [Key Vaults][create-and-store-keyvault-credentials] and [Assigning access to Azure Key Vault][az-keyvault-set-policy].

### Unable to create a cache rule

If you're unable to create a cache rule, you may have hit the cache rule limit, or your rule might have a conflict with an existing rule.

## Cache rule limit issues

You may face issues while creating a cache rule if you exceed the limit of 1,000 cache rules.

If you hit this limit, delete any cache rules that are no longer needed.

## Unable to create cache rule using a wildcard

If a cache rule you're trying to create conflicts with an existing rule, you see an error message explaining there's already a cache rule with a wildcard for the specified target repository.

To resolve this issue, follow these steps:

1. Identify which existing cache rule is causing the conflict. Look for an existing rule that uses a wildcard (*) for the target repository.
1. Delete the conflicting cache rule that overlaps with the source repository and wildcard.
1. Create a new cache rule with the desired wildcard and target repository.
1. Double-check your cache configuration to ensure that the new rule is correctly applied and there are no other conflicting rules.

For more information about wildcards and potential conflicts, see [Wildcard support for artifact cache in Azure Container Registry](wildcards-artifact-cache.md).

## Supported upstream repositories

Be sure your upstream repository is supported. For more information, see [Upstream support](artifact-cache-overview.md#upstream-support).

<!-- LINKS - External -->
[create-and-store-keyvault-credentials]:/azure/key-vault/secrets/quick-create-portal
[az-keyvault-set-policy]: /azure/key-vault/general/assign-access-policy#assign-an-access-policy

## Next steps

- Learn how to use [wildcards](wildcards-artifact-cache.md) to match multiple paths within the container image registry.
- Learn more about [artifact cache](artifact-cache-overview.md).
