---
title: Access Network-Restricted Registry By Trusted Azure Service
description: Enable a trusted Azure service instance to securely access a network-restricted container registry to pull or push images 
ms.topic: how-to
author: chasedmicrosoft
ms.service: azure-container-registry
ms.author: doveychase
ms.date: 02/28/2025

# Customer intent: As a developer, I want to enable trusted Azure services to access my network-restricted container registry, so that I can securely pull or push images without compromising my registry's network security settings.
---

# Allow trusted services to securely access a network-restricted container registry

With Azure Container Registry, you can allow select trusted Azure services to access a registry that's configured with network access rules. When you allow trusted services, a trusted service instance can securely bypass the registry's network rules and perform operations such as pulling or pushing images. This article explains how to enable and use trusted services with a network-restricted Azure container registry.

Use the Azure Cloud Shell or a local installation of the Azure CLI to run the command examples in this article. Use version 2.18 or later to run it locally. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Limitations

* Certain registry access scenarios with trusted services require a [managed identity for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview). Except where noted that a user-assigned managed identity is supported, only a system-assigned identity can be used.
* Allowing trusted services doesn't apply to a container registry configured with a [service endpoint](container-registry-vnet.md). The feature only affects registries that are restricted with a [private endpoint](container-registry-private-link.md) or that have [public IP access rules](container-registry-access-selected-networks.md) applied.

## About trusted services

Azure Container Registry has a layered security model that supports multiple network configurations to restrict access to a registry. These configurations include:

* [Private endpoint with Azure Private Link](container-registry-private-link.md). When configured, a registry's private endpoint is accessible only to resources within the virtual network, using private IP addresses.  
* [Registry firewall rules](container-registry-access-selected-networks.md), which allow access to the registry's public endpoint only from specific public IP addresses or address ranges. You can also configure the firewall to block all access to the public endpoint when using private endpoints.

When you deploy a registry in a virtual network or configure it with firewall rules, it denies access to users or services from outside those sources.

Several multitenant Azure services operate from networks that you can't include in these registry network settings. As a result, these services can't perform operations such as pulling or pushing images to the registry. By designating certain service instances as "trusted", a registry owner can allow select Azure resources to securely bypass the registry's network settings to perform registry operations.

### Trusted services

Instances of the following services can access a network-restricted container registry if the registry's **allow trusted services** setting is enabled (the default). More services will be added over time.

Where indicated, access by the trusted service requires additional configuration of a managed identity in a service instance, assignment of an [RBAC role](container-registry-rbac-built-in-roles-overview.md), and authentication with the registry. For example steps, see [Trusted services workflow](#trusted-services-workflow), later in this article.

|Trusted service  |Supported usage scenarios  | Configure managed identity with RBAC role |
|---------|---------|------|
| Azure Container Instances | [Deploy to Azure Container Instances from Azure Container Registry using a managed identity](/azure/container-instances/using-azure-container-registry-mi) | Yes, either system-assigned or user-assigned identity |
| Microsoft Defender for Cloud | Vulnerability scanning by [Microsoft Defender for container registries](scan-images-defender.md) | No |
| Machine Learning | [Deploy](/azure/machine-learning/how-to-deploy-custom-container) or [train](/azure/machine-learning/how-to-train-with-custom-image) a model in a Machine Learning workspace using a custom Docker container image | Yes |
| Azure Container Registry | [Import images](container-registry-import-images.md) to or from a network-restricted Azure container registry | No |

> [!NOTE]
> Currently, enabling the `allow trusted services` setting doesn't apply to App Service.

## Allow trusted services - CLI

By default, the allow trusted services setting is enabled in a new Azure container registry. Disable or enable the setting by running the [az acr update](/cli/azure/acr#az-acr-update) command.

To disable:

```azurecli
az acr update --name myregistry --allow-trusted-services false
```

To enable the setting in an existing registry or a registry where it's already disabled:

```azurecli
az acr update --name myregistry --allow-trusted-services true
```

## Allow trusted services - portal

By default, the allow trusted services setting is enabled in a new Azure container registry.

To disable or re-enable the setting in the portal:

1. In the portal, navigate to your container registry.
1. Under **Settings**, select **Networking**.
1. In **Allow public network access**, select **Selected networks** or **Disabled**.
1. Do one of the following steps:
    * To disable access by trusted services, under **Firewall exception**, uncheck **Allow trusted Microsoft services to access this container registry**.
    * To allow trusted services, under **Firewall exception**, check **Allow trusted Microsoft services to access this container registry**.
1. Select **Save**.

## Trusted services workflow

Here's a typical workflow to enable an instance of a trusted service to access a network-restricted container registry. This workflow is needed when you use a service instance's managed identity to bypass the registry's network rules.

1. Enable a managed identity in an instance of one of the [trusted services](#trusted-services) for Azure Container Registry.
1. Assign the identity an [Azure role](container-registry-rbac-built-in-roles-overview.md) to your registry. For example, assign either `Container Registry Repository Reader` (for [ABAC-enabled registries](container-registry-rbac-abac-repository-permissions.md)) or `AcrPull` (for non-ABAC registries).
1. Configure the setting in the network-restricted registry to allow access by trusted services.
1. Use the identity's credentials to authenticate with the network-restricted registry.
1. Pull images from the registry, or perform other operations allowed by the role.

## Next steps

* To restrict access to a registry using a private endpoint in a virtual network, see [Configure Azure Private Link for an Azure container registry](container-registry-private-link.md).
* To set up registry firewall rules, see [Configure public IP network rules](container-registry-access-selected-networks.md).
