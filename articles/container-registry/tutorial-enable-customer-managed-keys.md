---
title: Enable a Customer-Managed Key for Azure Container Registry
description: In this tutorial, learn how to encrypt your Premium registry with a customer-managed key stored in Azure Key Vault or Azure Key Vault Managed HSM.
ms.topic: tutorial
ms.date: 10/09/2026
ai-usage: ai-assisted
ms.author: kumud
ms.service: azure-container-registry
ms.custom:
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: "As a DevOps engineer, I want to enable a customer-managed key for an Azure Container Registry, so that I can ensure data security by utilizing my own key for encryption and manage access effectively."
---

# Enable a customer-managed key

This article is part two in a four-part tutorial series. [Part one](tutorial-customer-managed-keys.md) provides an overview of customer-managed keys, their features, and considerations before you enable one on your registry. This article walks you through the steps of enabling a customer-managed key by using the Azure CLI, the Azure portal, or an Azure Resource Manager template.

## Prerequisites

* [Install the Azure CLI][azure-cli] or prepare to use [Azure Cloud Shell](/azure/cloud-shell/quickstart).
* Sign in to the [Azure portal](https://portal.azure.com/). 

## Enable a customer-managed key by using the Azure CLI

### Create a resource group

Run the [az group create][az-group-create] command to create a resource group that will hold your key store, container registry, and other required resources:

```azurecli
az group create --name <resource-group-name> --location <location>
```

### Create a user-assigned managed identity

Configure a user-assigned [managed identity](/azure/active-directory/managed-identities-azure-resources/overview) for the registry so that it can access the encryption key:

1. Run the [az identity create][az-identity-create] command to create the managed identity:

   ```azurecli
   az identity create \
     --resource-group <resource-group-name> \
     --name <managed-identity-name>
   ```

2. In the command output, take note of the `id` and `principalId` values to configure registry access to the key:

   ```JSON
   {
     "clientId": "00001111-aaaa-2222-bbbb-3333cccc4444",
     "clientSecretUrl": "https://control-eastus.identity.azure.net/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentityname/credentials?tid=aaaabbbb-0000-cccc-1111-dddd2222eeee&oid=aaaaaaaa-0000-1111-2222-bbbbbbbbbbbb&aid=00001111-aaaa-2222-bbbb-3333cccc4444",
     "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myresourcegroup",
     "location": "eastus",
     "name": "myidentityname",
     "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "resourceGroup": "myresourcegroup",
     "tags": {},
     "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
   }
   ```

3. For convenience, store the `id` and `principalId` values in environment variables:

   ```azurecli
   identityID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'id' --output tsv)

   identityPrincipalID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'principalId' --output tsv)
   ```

### Create and configure a key vault

1. Run the [az keyvault create][az-keyvault-create] command to create a key vault where you can store a customer-managed key for registry encryption. 

2. By default, the new key vault automatically enables the *soft delete* setting. To prevent data loss from accidental deletion of keys or key vaults, we recommend enabling the *purge protection* setting:

   ```azurecli
   az keyvault create --name <key-vault-name> \
     --resource-group <resource-group-name> \
     --enable-purge-protection
   ```

3. For convenience, take a note of the key vault's resource ID and store the value in environment variables:

   ```azurecli
   keyvaultID=$(az keyvault show --resource-group <resource-group-name> --name <key-vault-name> --query 'id' --output tsv)
   ```

#### Enable trusted services to access the key vault

If the key vault is in protection with a firewall or virtual network (private endpoint), you must enable the network settings to allow access by [trusted Azure services](/azure/key-vault/general/overview-vnet-service-endpoints#trusted-services). For more information, see [Configure Azure Key Vault networking settings](/azure/key-vault/general/how-to-azure-key-vault-network-security?tabs=azure-cli).

#### Enable managed identities to access the key vault

There are two ways to enable managed identities to access your key vault.

The first option is to configure the access policy for the key vault and set key permissions for access with a user-assigned managed identity:

1. Run the [az keyvault set policy][az-keyvault-set-policy] command. Pass the previously created and stored environment variable value of `principalID`.
 
2. Set key permissions to `get`, `unwrapKey`, and `wrapKey`:  

   ```azurecli
   az keyvault set-policy \
     --resource-group <resource-group-name> \
     --name <key-vault-name> \
     --object-id $identityPrincipalID \
     --key-permissions get unwrapKey wrapKey

   ```

The second option is to use [Azure role-based access control (RBAC)](/azure/key-vault/general/rbac-guide) to assign permissions to the user-assigned managed identity and access the key vault. Run the [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) command and assign the `Key Vault Crypto Service Encryption User` role to a user-assigned managed identity:

```azurecli
az role assignment create --assignee $identityPrincipalID \
  --role "Key Vault Crypto Service Encryption User" \
  --scope $keyvaultID
```

### Create a key in the key vault and get the key ID

1. Run the [az keyvault key create][az-keyvault-key-create] command to create a key in the key vault:

   ```azurecli
   az keyvault key create \
     --name <key-name> \
     --vault-name <key-vault-name>
   ```

2. In the command output, take note of the key ID (`kid`): 

   ```output
   [...]
     "key": {
       "crv": null,
       "d": null,
       "dp": null,
       "dq": null,
       "e": "AQAB",
       "k": null,
       "keyOps": [
         "encrypt",
         "decrypt",
         "sign",
         "verify",
         "wrapKey",
         "unwrapKey"
       ],
       "kid": "https://mykeyvault.vault.azure.net/keys/mykey/<version>",
       "kty": "RSA",
   [...]
   ```

3. For convenience, store the format that you choose for the key ID in the `$keyID` environment variable. You can use a key ID with or without a version.

#### Key rotation

You can choose manual or automatic key rotation.

Encrypting a registry with a customer-managed key that has a key version will allow only manual key rotation in Azure Container Registry. This example stores the key's `kid` property:

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)
```

Encrypting a registry with a customer-managed key by omitting a key version will enable automatic key rotation to detect a new key version in Azure Key Vault. This example removes the version from the key's `kid` property:

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)

keyID=$(echo $keyID | sed -e "s/\/[^/]*$//")
```

### Create and configure a Managed HSM

A Managed HSM must be provisioned and activated before you can create keys or assign data-plane roles. Follow [Quickstart: Provision and activate a Managed HSM by using the Azure CLI](/azure/key-vault/managed-hsm/quick-create-cli), and then return to this article.

If the Managed HSM firewall is enabled, configure trusted services or other network access before you create the registry. For more information, see [Network security for Managed HSM](/azure/key-vault/managed-hsm/network-security).

The user who creates the encryption key needs a Managed HSM role that permits key creation, such as **Managed HSM Crypto User**. For more information, see [Managed HSM data-plane role management](/azure/key-vault/managed-hsm/role-management).

Create an RSA-HSM key that supports key wrapping:

```azurecli
az keyvault key create \
  --hsm-name <managed-hsm-name> \
  --name <key-name> \
  --kty RSA-HSM \
  --ops wrapKey unwrapKey
```

Managed HSM uses local RBAC instead of Key Vault access policies. Assign the **Managed HSM Crypto Service Encryption User** role to the user-assigned identity. The following example limits the assignment to the key that the registry uses:

```azurecli
az keyvault role assignment create \
  --hsm-name <managed-hsm-name> \
  --role "Managed HSM Crypto Service Encryption User" \
  --assignee-object-id $identityPrincipalID \
  --assignee-principal-type MSI \
  --scope /keys/<key-name>
```

Store the key ID with its version for manual rotation:

```azurecli
keyID=$(az keyvault key show \
  --hsm-name <managed-hsm-name> \
  --name <key-name> \
  --query 'key.kid' --output tsv)
```

To enable automatic key rotation, remove the version from the key ID:

```azurecli
keyID=$(echo $keyID | sed -e "s/\/[^/]*$//")
```

> [!NOTE]
> A Managed HSM key URI uses the format `https://<managed-hsm-name>.managedhsm.azure.net/keys/<key-name>/<version>`. The DNS suffix can vary by Azure cloud.

### Create a registry with a customer-managed key

1. Run the [az acr create][az-acr-create] command to create a registry in the *Premium* service tier and enable the customer-managed key. 

2. Pass the managed identity ID (`id`) and key ID (`kid`) values stored in the environment variables in previous steps:

   ```azurecli
   az acr create \
     --resource-group <resource-group-name> \
     --name <container-registry-name> \
     --identity $identityID \
     --key-encryption-key $keyID \
     --sku Premium
   ```

### Show encryption status

Run the [az acr encryption show][az-acr-encryption-show] command to show the status of the registry encryption with a customer-managed key:

```azurecli
az acr encryption show --name <container-registry-name>
```

Depending on the key that's used to encrypt the registry, the output is similar to:

```console
{
  "keyVaultProperties": {
    "identity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "keyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
    "keyRotationEnabled": true,
    "lastKeyRotationTimestamp": xxxxxxxx
    "versionedKeyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
  },
  "status": "enabled"
}
```

## Enable a customer-managed key by using the Azure portal

### Create a user-assigned managed identity

To create a user-assigned [managed identity for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) in the Azure portal: 

1. Follow the steps to [create a user-assigned identity](/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal#create-a-user-assigned-managed-identity).

2. Save the identity's name to use it in later steps.

:::image type="content" source="media/container-registry-customer-managed-keys/create-managed-identity.png" alt-text="Screenshot of the options for creating a user-assigned identity in the Azure portal.":::

### Create a key vault

1. Follow the steps in [Quickstart: Create a key vault using the Azure portal](/azure/key-vault/general/quick-create-portal).

2. When you're creating a key vault for a customer-managed key, on the **Basics** tab, enable the **Purge protection** setting. This setting helps prevent data loss from accidental deletion of keys or key vaults.

   :::image type="content" source="media/container-registry-customer-managed-keys/create-key-vault.png" alt-text="Screenshot of the options for creating a key vault in the Azure portal.":::

#### Enable trusted services to access the key vault

If the key vault is in protection with a firewall or virtual network (private endpoint), enable the network setting to allow access by [trusted Azure services](/azure/key-vault/general/overview-vnet-service-endpoints#trusted-services). For more information, see [Configure Azure Key Vault networking settings](/azure/key-vault/general/how-to-azure-key-vault-network-security?tabs=azure-portal).

#### Enable managed identities to access the key vault

There are two ways to enable managed identities to access your key vault.

The first option is to configure the access policy for the key vault and set key permissions for access with a user-assigned managed identity:

1. Go to your key vault.
2. Select **Settings** > **Access policies > +Add Access Policy**.
3. Select **Key permissions**, and then select **Get**, **Unwrap Key**, and **Wrap Key**.
4. In **Select principal**, select the resource name of your user-assigned managed identity.  
5. Select **Add**, and then select **Save**.

:::image type="content" source="media/container-registry-customer-managed-keys/add-key-vault-access-policy.png" alt-text="Screenshot of options for creating a key vault access policy.":::

The other option is to assign the `Key Vault Crypto Service Encryption User` RBAC role to the user-assigned managed identity at the key vault scope. For detailed steps, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

### Create and configure a Managed HSM

1. Follow [Quickstart: Provision and activate a Managed HSM by using the Azure portal](/azure/key-vault/managed-hsm/quick-create-portal).
1. Ensure that the user who creates the encryption key has a Managed HSM role that permits key creation, such as **Managed HSM Crypto User**.
1. Under **Settings**, select **Keys**, and then create or import an RSA-HSM key that supports the `wrapKey` and `unwrapKey` operations.
1. Under **Settings**, select **Local RBAC**.
1. Add the **Managed HSM Crypto Service Encryption User** role assignment for the user-assigned identity. For the least privilege, assign the role at the scope of the encryption key. For more information, see [Managed HSM data-plane role management](/azure/key-vault/managed-hsm/role-management).
1. If the Managed HSM firewall is enabled, configure trusted services or other network access. For more information, see [Network security for Managed HSM](/azure/key-vault/managed-hsm/network-security).

> [!IMPORTANT]
> Azure Container Registry doesn't create or update Managed HSM role assignments. Configure the role assignment before you create the registry or change its encryption key.

### Create a key in a key vault

Create a key in the key vault and use it to encrypt the registry. Follow these steps if you want to select a specific key version as a customer-managed key. You might also need to create a key before creating the registry if key vault access is restricted to a private endpoint or selected networks. 

1. Go to your key vault.
1. Select **Settings** > **Keys**.
1. Select **+Generate/Import** and enter a unique name for the key.
1. Accept the remaining default values, and then select **Create**.
1. After creation, select the key and then select the current version. Copy the **Key identifier** for the key version.

### Create a container registry

1. Select **Create a resource** > **Containers** > **Container Registry**.
1. On the **Basics** tab, select or create a resource group, and then enter a registry name. In **SKU**, select **Premium**.
1. On the **Encryption** tab, for **Customer-managed key**, select **Enabled**.
1. For **Identity**, select the managed identity that you created.
1. For **Encryption**, choose one of the following options:
    * Choose **Select from Key Vault**, and then select an existing key vault or Managed HSM and a key. You can select **Create new** to create a key vault and key. The key that you select is unversioned and enables automatic key rotation.
    * Select **Enter key URI**, and provide the identifier of an existing Key Vault or Managed HSM key. You can provide either a versioned key URI (for a key that must be rotated manually) or an unversioned key URI (which enables automatic key rotation).

   If a Managed HSM firewall prevents the portal from listing keys, allow your client IP address or select **Enter key URI**. Allowing trusted services enables Azure Container Registry to reach the Managed HSM, but it doesn't allow a client outside the configured network boundary to use the key picker.
1. Select **Review + create**.
1. Select **Create** to deploy the registry instance.

:::image type="content" source="media/container-registry-customer-managed-keys/create-encrypted-registry.png" alt-text="Screenshot that shows options for creating an encrypted registry in the Azure portal.":::

### Show the encryption status

To see the encryption status of your registry in the portal, go to your registry. Under **Settings**, select **Encryption**.

## Enable a customer-managed key by using a Resource Manager template

You can use a Resource Manager template to create a container registry and enable encryption with a customer-managed key stored in Key Vault or Managed HSM.

Before you deploy the template, create the user-assigned identity and configure its access to the key:

* For Key Vault, grant the identity the `get`, `wrapKey`, and `unwrapKey` permissions by using an access policy or the **Key Vault Crypto Service Encryption User** Azure RBAC role.
* For Managed HSM, assign the identity the **Managed HSM Crypto Service Encryption User** local RBAC role at the key scope.

The template declares the existing identity so that the registry can depend on it. Deploy the template to the same resource group as the identity.

1. Copy the following content of a Resource Manager template to a new file and save it as *CMKtemplate.json*:

   ```json
   {
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "registry_name": {
         "type": "String"
       },
       "identity_name": {
         "type": "String"
       },
       "kek_id": {
         "type": "String"
       }
     },
     "resources": [
       {
         "type": "Microsoft.ContainerRegistry/registries",
         "apiVersion": "2019-12-01-preview",
         "name": "[parameters('registry_name')]",
         "location": "[resourceGroup().location]",
         "sku": {
           "name": "Premium",
           "tier": "Premium"
         },
         "identity": {
           "type": "UserAssigned",
           "userAssignedIdentities": {
             "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]": {}
           }
         },
         "dependsOn": [
           "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
         ],
         "properties": {
           "adminUserEnabled": false,
           "encryption": {
             "status": "enabled",
             "keyVaultProperties": {
               "identity": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').clientId]",
               "keyIdentifier": "[parameters('kek_id')]"
             }
           },
           "networkRuleSet": {
             "defaultAction": "Allow",
             "virtualNetworkRules": [],
             "ipRules": []
           },
           "policies": {
             "quarantinePolicy": {
               "status": "disabled"
             },
             "trustPolicy": {
               "type": "Notary",
               "status": "disabled"
             },
             "retentionPolicy": {
               "days": 7,
               "status": "disabled"
             }
           }
         }
       },
       {
         "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
         "apiVersion": "2018-11-30",
         "name": "[parameters('identity_name')]",
         "location": "[resourceGroup().location]"
       }
     ]
   }
   ```

2. Follow the steps in the previous sections to create the following resources:

   * User-assigned managed identity, identified by name
   * Key Vault or Managed HSM key, identified by key ID
   * Required Key Vault permissions or Managed HSM local RBAC role assignment

3. Run the [az deployment group create][az-deployment-group-create] command to create the registry by using the preceding template file. Provide a new registry name, the existing user-assigned managed identity name, and the key ID that you created.

   ```azurecli
   az deployment group create \
     --resource-group <resource-group-name> \
     --template-file CMKtemplate.json \
     --parameters \
       registry_name=<registry-name> \
       identity_name=<managed-identity> \
       kek_id=<key-id>
   ```

4. Run the [az acr encryption show][az-acr-encryption-show] command to show the status of registry encryption:

   ```azurecli
   az acr encryption show --name <registry-name>
   ```

## Next steps

Advance to the [next article](tutorial-rotate-revoke-customer-managed-keys.md) to walk through rotating customer-managed keys, updating key versions, and revoking a customer-managed key. 

<!-- LINKS - internal -->

[azure-cli]: /cli/azure/install-azure-cli
[az-group-create]: /cli/azure/group#az-group-create
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-keyvault-create]: /cli/azure/keyvault#az-keyvault-create
[az-keyvault-key-create]: /cli/azure/keyvault/key#az-keyvault-key-create
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-encryption-show]: /cli/azure/acr/encryption#az-acr-encryption-show
