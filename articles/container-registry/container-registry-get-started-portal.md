---
title: Quickstart - Create Registry in Portal
description: Quickly learn to create a private Azure container registry using the Azure portal, push a container image, and pull and run the image from the registry.
author: rayoef
ms.author: rayoflores
ms.date: 12/9/2025
ms.topic: quickstart
ms.service: azure-container-registry
ms.custom:
  - mvc
  - mode-ui
  - sfi-image-nochange
# Customer intent: "As a cloud developer, I want to create and manage a private container registry in the portal, so that I can efficiently store and deploy container images for my applications."
---

# Quickstart: Create an Azure container registry by using the Azure portal

[Azure Container Registry](container-registry-intro.md) is a private registry service for building, storing, and managing container images and related artifacts. In this quickstart, you create an Azure container registry instance with the Azure portal. Next, you use Docker commands to push a container image into the registry. Finally, you  pull and run the image from your registry.

## Prerequisites

If you don't have an Azure account, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

### [Azure CLI](#tab/azure-cli)

To sign in to the registry to work with container images, this quickstart requires that you run the Azure CLI, preferably the most recent version. If you need to install or upgrade, see [How to install the Azure CLI][azure-cli-install].

### [Azure PowerShell](#tab/azure-powershell)

To sign in to the registry to work with container images, this quickstart requires that you run Azure PowerShell (version 7.5.0 or later recommended). Run `Get-Module Az -ListAvailable` to find the version. If you need to install or upgrade, see [How to install Azure PowerShell][azure-powershell-install].

---

You must also have Docker installed locally with the daemon running. Docker provides packages that easily configure Docker on any [Mac][docker-mac], [Windows][docker-windows], or [Linux][docker-linux] system.

## Create a container registry

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Select **Create a resource** > **Infrastructure services** > **Container Registry** > **Create**.

   :::image type="content" source="media/container-registry-get-started-portal/create-resource-container-registry.png" alt-text="Screenshot of the option to create a new container registry resource in the Azure portal.":::

1. In the **Basics** tab, select the subscription where you want to create the container registry.
1. Select **Create new** to create a new resource group, and enter `myResourceGroup` for the resource group name.
1. Enter a **Registry name**. The registry name must be unique within Azure, and contain 5-50 alphanumeric characters, excluding dash characters (`-`). This name is part of the fully qualified DNS name of the registry.
1. Select **West US 2** for **Location**, and for [**Pricing plan**][container-registry-skus], select `Standard`.
1. For **Domain name label scope**, select **Tenant Reuse**, or choose another option as described in the [Configure Domain Name Label (DNL) option](#configure-domain-name-label-dnl-option) section.
1. For **Role assignment permissions mode**, select **RBAC Registry + ABAC Repository Permissions** to retain standard Microsoft Entra role-based access control (RBAC) role assignments, while optionally applying [Microsoft Entra attribute-based access control (ABAC) conditions](container-registry-rbac-abac-repository-permissions.md) for fine‑grained, repository‑level access control.

  :::image type="content" source="media/container-registry-get-started-portal/configure-container-registry-options.png" alt-text="Screenshot showing the container registry creation settings in the portal":::

1. Leave the other options set to their default values, and select **Review + create**. After reviewing the settings, select **Create**.

[!INCLUDE [container-registry-quickstart-sku](./includes/container-registry-quickstart-sku.md)]

When the **Deployment succeeded** message appears, select **Go to resource** to view your new container registry.

:::image type="content" source="media/container-registry-get-started-portal/container-registry-overview.png" alt-text="Screenshot of the overview page of a container registry in the Azure portal.":::

Take note of the registry name and the value of the **Login server**, which is a fully qualified name ending with `azurecr.io` in the Azure cloud.

Use the login server in the following steps when you push and pull images with Docker, as well as in downstream references such as Dockerfiles, Kubernetes YAML, and Helm charts.

## Sign in to registry

### [Azure CLI](#tab/azure-cli)

Before pushing and pulling container images, you must sign in to the registry instance. [Sign in to the Azure CLI][get-started-with-azure-cli] on your local machine, then run the [az acr login][az-acr-login] command.

Specify only the registry resource name when signing in with the Azure CLI, such as `az acr login -n registryname`. Don't use the fully qualified login server name, such as `registryname.azurecr.io` or `registryname-hash.azurecr.io` (for DNL-enabled registries).

```azurecli
az acr login --name <registry-name>
```

Example:

```azurecli
az acr login --name contosoacrregistry
```

The command returns `Login Succeeded` when it completes.

### [Azure PowerShell](#tab/azure-powershell)

Before pushing and pulling container images, you must sign in to the registry instance. [Sign in to Azure PowerShell][get-started-with-azure-powershell] on your local machine, then run the [Connect-AzContainerRegistry][connect-azcontainerregistry] cmdlet. Specify only the registry resource name when signing in with Azure PowerShell. Don't use the fully qualified login server name.

```azurepowershell
Connect-AzContainerRegistry -Name <registry-name>
```

Example:

```azurepowershell
Connect-AzContainerRegistry -Name contosoacrregistry
```

The command returns `Login Succeeded` when it completes. 

---

[!INCLUDE [container-registry-quickstart-docker-push](./includes/container-registry-quickstart-docker-push.md)]

## List container images

To list the images in your registry, go to your registry in the portal. Under **Services**, select **Repositories**, then select the **hello-world** repository you created with `docker push`.

:::image type="content" source="media/container-registry-get-started-portal/list-container-images.png" alt-text="Screenshot showing the container images for a container registry in the Azure portal.":::

When you select the **hello-world** repository, you see the `v1`-tagged image under **Tags**.

[!INCLUDE [container-registry-quickstart-docker-pull](./includes/container-registry-quickstart-docker-pull.md)]

## Clean up resources

To remove the resources you created, go to the **myResourceGroup** resource group in the Azure portal. Select **Delete resource group** to remove the resource group, the container registry, and the container images.

## Configure Domain Name Label (DNL) option

The Domain Name Label (DNL) feature strengthens security by preventing subdomain takeover attacks of registry DNS names. These attacks occur when a registry is deleted, and another entity reuses the same registry name, potentially causing downstream references to pull from the registry re-created by the other entity.

DNL addresses this issue by appending a unique hash to the registry's DNS name. This approach ensures that even if another entity reuses the same registry name, the DNS names differ because of the unique hash. This safeguard prevents your downstream references from inadvertently pointing to the registry re-created by the other entity.

When you create a registry from the Azure portal, select the **Domain Name Label Scope** from the available options:

- **Unsecure**: Creates the DNS name as-is, based on the registry name (for example, `contosoacrregistry.azurecr.io`). This option doesn't include DNL protection.
- **Tenant Reuse**: Appends a unique hash based on the tenant and registry name, ensuring the DNS name is unique within the tenant.
- **Subscription Reuse**: Appends a unique hash based on the subscription, tenant, and registry name, ensuring the DNS name is unique within the subscription.
- **Resource Group Reuse**: Appends a unique hash based on the resource group, subscription, tenant, and registry name, ensuring the DNS name is unique within the resource group.
- **No Reuse**: Generates a unique DNS name with a unique hash every time you create the registry, regardless of other factors, ensuring the DNS name is always unique.

> [!IMPORTANT]
> The DNL scope you select during registry creation is permanent and can't be modified later. This choice ensures consistent DNS behavior and prevents disruptions to downstream references.

For all DNL-enabled options except **Unsecure**, the DNS name follows the format `registryname-hash.azurecr.io`, where the dash (`-`) serves as the hash delineator. For instance, a registry named `contosoacrregistry` with the `Tenant Reuse` DNL scope has a DNS name like `contosoacrregistry-abc123.azurecr.io`. To avoid conflicts, the dash character (`-`) isn't permitted in the registry name.

If the DNS name differs from the registry name, you need to update downstream files such as Dockerfiles, Kubernetes YAML, and Helm charts to reflect the full DNS name with the DNL hash. For example, if you want your downstream Dockerfile to reference a registry named `contosoacrregistry` with the `Tenant Reuse` DNL scope, you need to update the reference to the full value such as `contosoacrregistry-abc123.azurecr.io` in your downstream Dockerfile.

## Next steps

In this quickstart, you created an Azure Container Registry with the Azure portal, pushed a container image, and pulled and ran the image from the registry. To continue working with Azure Container Registry, see the Azure Container Registry tutorials.

> [!div class="nextstepaction"]
> [Azure Container Registry tutorials][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Azure Container Registry Tasks tutorials][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/install
[docker-mac]: https://docs.docker.com/desktop/setup/install/mac-install/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/desktop/setup/install/windows-install/

<!-- LINKS - internal -->
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
[container-registry-skus]: container-registry-skus.md
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-powershell-install]: /powershell/azure/install-az-ps
[get-started-with-azure-cli]: /cli/azure/get-started-with-azure-cli
[get-started-with-azure-powershell]: /powershell/azure/get-started-azureps
[az-acr-login]: /cli/azure/acr#az-acr-login
[connect-azcontainerregistry]: /powershell/module/az.containerregistry/connect-azcontainerregistry
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
