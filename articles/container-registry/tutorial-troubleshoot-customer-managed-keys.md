---
title: Troubleshoot Customer-Managed Keys in Azure Container Registry
description: Learn how to troubleshoot the most common problems for a registry that's enabled with a customer-managed key.
author: KumudD
ms.topic: tutorial
ms.date: 10/31/2023
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

## Error after you enable a key vault firewall

If you enable a key vault firewall or virtual network after creating an encrypted registry, you might see HTTP 403 or other errors with image import or automated key rotation. To correct this problem, reconfigure the managed identity and key that you initially used for encryption. See the steps in [Rotate a customer-managed key](tutorial-rotate-revoke-customer-managed-keys.md#rotate-a-customer-managed-key).

If the problem persists, contact Azure Support.

## Identity expiry error

The identity attached to a registry is set for autorenewal to avoid expiry. If you disassociate an identity from a registry, an error message occurs explaining to you can't remove the identity in use for CMK. Attempting to remove the identity jeopardizes the autorenewal of identity. The artifact pull/push operations work until the identity expires (Usually three months). After the identity expiration, you'll see the HTTP 403 with an error message "The identity associated with the registry is inactive. This could be due to attempted removal of the identity. Reassign the identity manually". 

You have to reassign the identity back to registry explicitly.

1. Run the [az acr identity assign](/cli/azure/acr/identity/#az-acr-identity-assign) command to reassign the identity manually.

    - For example,
   
    ```azurecli-interactive
    az acr identity assign -n myRegistry \
    --identities "/subscriptions/mysubscription/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentity"
    ``` 

## Accidental deletion of a key vault or key

Deletion of the key vault, or the key, that's used to encrypt a registry with a customer-managed key makes the registry's content inaccessible. If [soft delete](/azure/key-vault/general/soft-delete-overview) is enabled in the key vault (the default option), you can recover a deleted vault or key vault object and resume registry operations.

## Azure Policy remediation fails on a customer-managed key registry

An Azure Policy assignment with a **Modify** effect (for example, the built-in policy [Configure container registries to disable ARM audience token authentication](container-registry-disable-authentication-as-arm.md)) can fail to automatically remediate a registry that's encrypted with a customer-managed key. The remediation task fails, and the registry stays noncompliant. The remediation deployment reports a `BadRequest` error similar to the following:

```output
Invalid encryption key vault property specified for registry <registry-name>. Property Identity and KeyIdentifier must be set when encryption is enabled. For more information on customer-managed keys, please visit 'https://aka.ms/acr/cmk'.
```

This is a known limitation: automatic **Modify** remediation isn't currently supported on registries that are encrypted with a customer-managed key. The remediation can't preserve the protected key vault settings when it resubmits the registry configuration, so the registry rejects the update with the preceding error and stays noncompliant.

To bring the registry into compliance, set the property that the policy targets directly instead of relying on automatic remediation. Setting the property directly changes only that one setting and leaves the encryption configuration untouched, so the registry accepts the update. Use the command that corresponds to the policy you're remediating.

For the **Configure container registries to disable ARM audience token authentication** policy, run:

```azurecli
az acr config authentication-as-arm update \
    --registry myRegistry \
    --resource-group myResourceGroup \
    --status Disabled
```

After you run the command, trigger a policy compliance scan or wait for the next evaluation. The registry then reports as compliant. The automatic remediation task might still show a *Failed* status for registries that are encrypted with a customer-managed key. That status doesn't affect the registry's compliance after you set the property directly.

## Next steps

For key vault deletion and recovery scenarios, see [Azure Key Vault recovery management with soft delete and purge protection](/azure/key-vault/general/key-vault-recovery).
