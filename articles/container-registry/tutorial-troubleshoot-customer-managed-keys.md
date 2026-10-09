---
title: Troubleshoot Customer-Managed Keys in Azure Container Registry
description: Learn how to troubleshoot the most common problems for a registry that's enabled with a customer-managed key.
author: KumudD
ms.topic: tutorial
ms.date: 10/09/2026
ai-usage: ai-assisted
ms.author: kumud
ms.service: azure-container-registry
# Customer intent: As a cloud administrator, I want to troubleshoot customer-managed keys in my container registry so that I can resolve common issues and ensure secure access to my images.
---

# Troubleshoot a customer-managed key 

This article is part four in a four-part tutorial series. [Part one](tutorial-customer-managed-keys.md) provides an overview of customer-managed keys, their features, and considerations before you enable one on your registry. In [part two](tutorial-enable-customer-managed-keys.md), you learn how to enable a customer-managed key by using the Azure CLI, the Azure portal, or an Azure Resource Manager template. In [part three](tutorial-rotate-revoke-customer-managed-keys.md), you learn how to rotate, update, and revoke a customer-managed key. This article helps you troubleshoot and resolve common problems with customer-managed keys.

## Error when you're removing a managed identity

If you try to remove a user-assigned or system-assigned managed identity that you used to configure encryption for your registry, you might see an error:
 
```
Azure resource '/subscriptions/xxxx/resourcegroups/myGroup/providers/Microsoft.ContainerRegistry/registries/myRegistry' does not have access to identity 'xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx' Try forcibly adding the identity to the registry <registry name>. For more information on bring your own key, please visit 'https://aka.ms/acr/cmk'.
```
 
You're unable to change (rotate) the encryption key. The resolution steps depend on the type of identity that you used for encryption.

### Removing a user-assigned identity

If you get the error when you try to remove a user-assigned identity, follow these steps: 
 
1. Reassign the user-assigned identity by using the [az acr identity assign](/cli/azure/acr/identity/#az-acr-identity-assign) command. 
2. Pass the user-assigned identity's resource ID, or use the identity's name when it's in the same resource group as the registry. 

   For example:

   ```azurecli
   az acr identity assign -n myRegistry \
       --identities "/subscriptions/mysubscription/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentity"
   ```
        
3. Change the key and assign a different identity.
4. Now, you can remove the original user-assigned identity.

### Removing a system-assigned identity

If you get the error when you try to remove a system-assigned identity, [create an Azure support ticket](https://azure.microsoft.com/support/create-ticket/) for assistance in restoring the identity.

## Error after you enable a key store firewall

If you enable a Key Vault or Managed HSM firewall, virtual network, or private endpoint after creating an encrypted registry, you might see HTTP 403 or other errors with image import or automated key rotation. Ensure that the key store's network configuration allows Azure Container Registry to reach the key. For Managed HSM networking options, see [Network security for Managed HSM](/azure/key-vault/managed-hsm/network-security).

If Azure Container Registry can access the Managed HSM but the portal key picker can't list keys, ensure that your client is inside the configured network boundary or use **Enter key URI**. Trusted-services bypass doesn't grant portal clients access through the Managed HSM firewall.

After you correct the network configuration, reconfigure the managed identity and key that you initially used for encryption. See the steps in [Rotate a customer-managed key](tutorial-rotate-revoke-customer-managed-keys.md#rotate-a-customer-managed-key).

If the problem persists, contact Azure Support.

## Error when you use a Managed HSM key

Check the following requirements if registry creation, key rotation, or image operations fail when the encryption key is stored in Managed HSM:

* The Managed HSM is provisioned and activated. Data-plane operations, including key creation and role assignment, are unavailable until activation is complete.
* The key is an RSA-HSM key that supports the `wrapKey` and `unwrapKey` operations.
* The registry's managed identity has the **Managed HSM Crypto Service Encryption User** local RBAC role. For the least privilege, assign the role at `/keys/<key-name>`.
* The key URI uses HTTPS and has the format `https://<managed-hsm-name>.<managed-hsm-dns-suffix>/keys/<key-name>` or `https://<managed-hsm-name>.<managed-hsm-dns-suffix>/keys/<key-name>/<version>`.
* The URI identifies a key. URIs for secrets, certificates, or other Managed HSM object paths aren't supported.

To verify the role assignment for a specific key, run:

```azurecli
az keyvault role assignment list \
  --hsm-name <managed-hsm-name> \
  --assignee-object-id <identity-principal-id> \
  --role "Managed HSM Crypto Service Encryption User" \
  --scope /keys/<key-name>
```

Managed HSM local RBAC changes can take several minutes to propagate. If the role assignment is correct but validation still fails, wait for propagation and retry the registry operation.

## Identity expiry error

The identity attached to a registry is set for autorenewal to avoid expiry. If you disassociate an identity from a registry, an error message occurs explaining to you can't remove the identity in use for CMK. Attempting to remove the identity jeopardizes the autorenewal of identity. The artifact pull/push operations work until the identity expires (Usually three months). After the identity expiration, you'll see the HTTP 403 with an error message "The identity associated with the registry is inactive. This could be due to attempted removal of the identity. Reassign the identity manually". 

You have to reassign the identity back to registry explicitly.

1. Run the [az acr identity assign](/cli/azure/acr/identity/#az-acr-identity-assign) command to reassign the identity manually.

    - For example,
   
    ```azurecli-interactive
    az acr identity assign -n myRegistry \
    --identities "/subscriptions/mysubscription/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentity"
    ``` 

## Accidental deletion of a key store or key

Deletion of the Key Vault or Managed HSM resource, or the key that's used to encrypt a registry, makes the registry's content inaccessible. If [soft delete](/azure/key-vault/general/soft-delete-overview) is enabled in a key vault, you can recover the deleted vault or key vault object and resume registry operations. Managed HSM also supports soft-delete recovery, but it continues to incur charges while it's in a soft-deleted state.

## Next steps

For deletion and recovery scenarios, see:

* [Azure Key Vault recovery management with soft delete and purge protection](/azure/key-vault/general/key-vault-recovery)
* [Managed HSM soft-delete and purge protection](/azure/key-vault/managed-hsm/recovery)
