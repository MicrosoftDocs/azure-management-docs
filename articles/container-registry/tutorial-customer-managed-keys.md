---
title: Customer-Managed Keys for Azure Container Registry
description: Learn how to encrypt your Premium container registry by using a customer-managed key stored in Azure Key Vault or Azure Key Vault Managed HSM.
ms.topic: tutorial
ms.date: 10/09/2026
ms.author: kumud
ms.service: azure-container-registry
# Customer intent: "As a cloud administrator, I want to implement customer-managed keys for my container registry so that I can enhance encryption security and maintain compliance with regulatory requirements."
---

# Overview of customer-managed keys

Azure Container Registry automatically encrypts images and other artifacts that you store. By default, Azure automatically encrypts the registry content at rest by using [service-managed keys](/azure/security/fundamentals/encryption-models). By using a customer-managed key, you can supplement default encryption with an additional encryption layer.
  
This article is part one in a four-part tutorial series. The tutorial covers:

> [!div class="checklist"]
> * Overview of customer-managed keys
> * Enable a customer-managed key
> * Rotate and revoke a customer-managed key
> * Troubleshoot a customer-managed key

## About customer-managed keys 

A customer-managed key gives you the ownership to bring your own key in [Azure Key Vault](/azure/key-vault/general/overview) or [Azure Key Vault Managed HSM](/azure/key-vault/managed-hsm/overview). When you enable a customer-managed key, you can manage its rotations, control the access and permissions to use it, and audit its use.

Key features include:

* **Regulatory compliance**: Azure automatically encrypts registry content at rest with [service-managed keys](/azure/security/fundamentals/encryption-models), but customer-managed key encryption helps you meet guidelines for regulatory compliance.

* **Integration with Azure Key Vault**: Customer-managed keys support server-side encryption through integration with [Azure Key Vault](/azure/key-vault/general/overview) and [Azure Key Vault Managed HSM](/azure/key-vault/managed-hsm/overview). You can create or import encryption keys in either key store.

* **Key lifecycle management**: Integrating customer-managed keys with Key Vault or Managed HSM gives you full control and responsibility for the key lifecycle, including rotation and management.

## Before you enable a customer-managed key  

Before you configure Azure Container Registry with a customer-managed key, consider the following information:

* This feature is available in the Premium service tier for a container registry. For more information, see [Azure Container Registry service tiers](container-registry-skus.md).
* You can currently enable a customer-managed key only while creating a registry.
* You can't disable the encryption after you enable a customer-managed key on a registry.
* You have to configure a *user-assigned* managed identity to access the key store. Later, if required, you can enable the registry's *system-assigned* managed identity for key access.
* Azure Container Registry supports only RSA or RSA-HSM keys. Elliptic-curve keys aren't currently supported.
* A Managed HSM must be provisioned and activated before you configure it for registry encryption. For more information, see [Provision and activate a Managed HSM](/azure/key-vault/managed-hsm/quick-create-cli).
* Managed HSM uses local role-based access control (RBAC), not Key Vault access policies. Assign the registry identity the **Managed HSM Crypto Service Encryption User** role, preferably at the scope of the encryption key.
* Managed HSM availability varies by Azure cloud and region. Verify that Managed HSM is available in the location and cloud where you plan to use it.
* In a registry that's encrypted with a customer-managed key, you can retain logs for [Azure Container Registry tasks](container-registry-tasks-overview.md) for only 24 hours. To retain logs for a longer period, see [View and manage task run logs](container-registry-tasks-logs.md#alternative-log-storage).
* [Content trust](container-registry-content-trust.md) is currently not supported in a registry that's encrypted with a customer-managed key.

## Update the customer-managed key version

Azure Container Registry supports both automatic and manual rotation of registry encryption keys when a new key version is available in Key Vault or Managed HSM.

>[!IMPORTANT]
>It's an important security consideration for a registry with customer-managed key encryption to frequently update (rotate) the key versions. Follow your organization's compliance policies to regularly update key versions in the key store.

* **Automatically update the key version**: When a registry is encrypted with a non-versioned key, Azure Container Registry regularly checks the key store for a new key version and updates the customer-managed key within one hour. We suggest that you omit the key version when you enable registry encryption with a customer-managed key. Azure Container Registry will then automatically use and update the latest key version.

* **Manually update the key version**: When a registry is encrypted with a specific key version, Azure Container Registry uses that version for encryption until you manually rotate the customer-managed key. We suggest that you specify the key version when you enable registry encryption with a customer-managed key. Azure Container Registry will then use a specific version of a key for registry encryption.

For details, see [Key rotation](tutorial-enable-customer-managed-keys.md#key-rotation) and [Update key version](tutorial-rotate-revoke-customer-managed-keys.md#create-or-update-the-key-version-by-using-the-azure-cli).

## Next steps

* To enable your container registry with a customer-managed key by using the Azure CLI, the Azure portal, or an Azure Resource Manager template, advance to the next article: [Enable a customer-managed key](tutorial-enable-customer-managed-keys.md).
* Learn more about [encryption at rest in Azure](/azure/security/fundamentals/encryption-atrest).
* Learn how to [secure access to a key vault](/azure/key-vault/general/security-features) or manage [Managed HSM local RBAC roles](/azure/key-vault/managed-hsm/role-management).
