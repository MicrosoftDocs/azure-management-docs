---
title: Quickstart - Create Registry in Portal
description: Quickly learn to create a private Azure container registry using the Azure portal, push a container image, and pull and run the image from the registry.
author: chasedmicrosoft
ms.author: doveychase
ms.date: 10/31/2023
ms.topic: quickstart
ms.service: azure-container-registry
ms.custom:
  - mvc
  - mode-ui
  - sfi-image-nochange
# Customer intent: "As a cloud developer, I want to create and manage a private container registry in the portal, so that I can efficiently store and deploy container images for my applications."
---

# Quickstart: Create an Azure container registry using the Azure portal

Azure Container Registry is a private registry service for building, storing, and managing container images and related artifacts. In this quickstart, you create an Azure container registry instance with the Azure portal. Then, use Docker commands to push a container image into the registry, and finally pull and run the image from your registry.

### [Azure CLI](#tab/azure-cli)

To log in to the registry to work with container images, this quickstart requires that you are running the Azure CLI (version 2.0.55 or later recommended). Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][azure-cli-install].

### [Azure PowerShell](#tab/azure-powershell)

To log in to the registry to work with container images, this quickstart requires that you are running the Azure PowerShell (version 7.5.0 or later recommended). Run `Get-Module Az -ListAvailable` to find the version. If you need to install or upgrade, see [Install Azure PowerShell module][azure-powershell-install].

---

You must also have Docker installed locally with the daemon running. Docker provides packages that easily configure Docker on any [Mac][docker-mac], [Windows][docker-windows], or [Linux][docker-linux] system.

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Create a container registry

Select **Create a resource** > **Containers** > **Container Registry**.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-01.png" alt-text="Navigate to container registry in portal":::

### Configure container registry name and SKU

In the **Basics** tab, enter values for **Resource group** and **Registry name**. The registry name must be unique within Azure, and contain 5-50 alphanumeric characters, with dash characters (`-`) not allowed in the registry name. For this quickstart create a new resource group in the `West US 2` location named `myResourceGroup`, and for **SKU**, select `Standard`.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-02.png" alt-text="Create container registry in the portal":::

For more information about different SKU options, see [Azure Container Registry SKUs][container-registry-skus].

### Configure Domain Name Label (DNL) option

The Domain Name Label (DNL) feature strengthens security by preventing subdomain takeover attacks of registry DNS names. These attacks occur when a registry is deleted, and another entity reuses the same registry name, potentially causing downstream references to pull from the registry re-created by the other entity.

DNL addresses this by appending a unique hash to the registry's DNS name. This ensures that even if the same registry name is reused by another entity, the DNS names will differ due to the unique hash. This safeguards your downstream references from inadvertently pointing to the registry re-created by the other entity.

When creating a registry from the Portal, select the **Domain Name Label Scope** from the available options:

- **Unsecure**: Creates the DNS name as-is, based on the registry name (e.g., `contosoacrregistry.azurecr.io`). This option does not include DNL protection.
- **Tenant Reuse**: Appends a unique hash based on the tenant and registry name, ensuring the DNS name is unique within the tenant.
- **Subscription Reuse**: Appends a unique hash based on the subscription, tenant, and registry name, ensuring the DNS name is unique within the subscription.
- **Resource Group Reuse**: Appends a unique hash based on the resource group, subscription, tenant, and registry name, ensuring the DNS name is unique within the resource group.
- **No Reuse**: Generates a unique DNS name with a unique hash every time the registry is created, regardless of other factors, ensuring the DNS name is always unique.

> [!NOTE]
> **Immutable Configuration**: The DNL scope selected during registry creation is permanent and cannot be modified later. This ensures consistent DNS behavior and prevents disruptions to downstream references.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-03.png" alt-text="Configure Domain Name Label option":::

### DNS Name Implications of DNL options

**DNS Name Format**: For all DNL-enabled options except **Unsecure**, the DNS name follows the format `registryname-hash.azurecr.io`, where the dash (`-`) serves as the hash delineator. To avoid conflicts, dash (`-`) is not permitted in the registry name. For instance, a registry named `contosoacrregistry` with the `Tenant Reuse` DNL scope will have a DNS name like `contosoacrregistry-e7ggejfuhzhgedc8.azurecr.io`.

**Downstream References**: The DNS name may differ from the registry name, necessitating updates in downstream files such as Dockerfiles, Kubernetes YAML, and Helm charts to reflect the full DNS name with the DNL hash. For example, if you want your downstream Dockerfile to reference a registry named `contosoacrregistry` with the `Tenant Reuse` DNL scope, you would need to update the reference to `contosoacrregistry-e7ggejfuhzhgedc8.azurecr.io` in your downstream Dockerfile.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-04a.png" alt-text="Screenshot of reviewing the Domain Name Label option and DNS name.":::

### Configure role assignment permissions mode

Configure the "Role assignment permissions mode" of the new registry.
This option determines how Microsoft Entra role-based access control (RBAC) and role assignments are managed for the registry, including the use of Microsoft Entra attribute-based access control (ABAC) for Microsoft Entra repository permissions.

Choose "RBAC Registry + ABAC Repository Permissions" to retain standard Microsoft Entra RBAC role assignments, while optionally applying Microsoft Entra ABAC conditions for fine‑grained, repository‑level access control.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-04b.png" alt-text="Screenshot of of configuring role assignment permissions mode":::

For more information on this option, see [Microsoft Entra attribute-based access control (ABAC) for repository permissions](container-registry-rbac-abac-repository-permissions.md).

### Deploying the container registry

Accept default values for the remaining settings. Then select **Review + create**. After reviewing the settings, select **Create**.

[!INCLUDE [container-registry-quickstart-sku](./includes/container-registry-quickstart-sku.md)]

When the **Deployment succeeded** message appears, select the container registry in the portal. 

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-05.png" alt-text="Container registry Overview in the portal":::

Take note of the registry name and the value of the **Login server**, which is a fully qualified name ending with `azurecr.io` in the Azure cloud. If you selected a DNL option, the login server name will include a unique hash.

Please use the login server in the following steps when you push and pull images with Docker, as well as in downstream references such as Dockerfiles, Kubernetes YAML, and Helm charts.

## Log in to registry

### [Azure CLI](#tab/azure-cli)

Before pushing and pulling container images, you must log in to the registry instance. [Sign into the Azure CLI][get-started-with-azure-cli] on your local machine, then run the [az acr login][az-acr-login] command.

Specify only the registry resource name when logging in with the Azure CLI, such as `az acr login -n registryname`. Don't use the fully qualified login server name, such as `registryname.azurecr.io` or `registryname-hash.azurecr.io` (for DNL-enabled registries).

```azurecli
az acr login --name <registry-name>
```

Example:

```azurecli
az acr login --name contosoacrregistry
```

The command returns `Login Succeeded` once completed.

### [Azure PowerShell](#tab/azure-powershell)

Before pushing and pulling container images, you must log in to the registry instance. [Sign into the Azure PowerShell][get-started-with-azure-powershell] on your local machine, then run the [Connect-AzContainerRegistry][connect-azcontainerregistry] cmdlet. Specify only the registry resource name when logging in with the Azure PowerShell. Don't use the fully qualified login server name.

```azurepowershell
Connect-AzContainerRegistry -Name <registry-name>
```

Example:

```azurepowershell
Connect-AzContainerRegistry -Name contosoacrregistry
```

The command returns `Login Succeeded` once completed. 

---

[!INCLUDE [container-registry-quickstart-docker-push](./includes/container-registry-quickstart-docker-push.md)]

## List container images

To list the images in your registry, navigate to your registry in the portal and select **Repositories**, then select the  **hello-world** repository you created with `docker push`.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-09.png" alt-text="List container images in the portal":::

By selecting the **hello-world** repository, you see the `v1`-tagged image under **Tags**.

[!INCLUDE [container-registry-quickstart-docker-pull](./includes/container-registry-quickstart-docker-pull.md)]

## Clean up resources

To clean up your resources, navigate to the **myResourceGroup** resource group in the portal. Once the resource group is loaded, click on **Delete resource group** to remove the resource group, the container registry, and the container images stored there.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-08.png" alt-text="Delete resource group in the portal":::


## Next steps

In this quickstart, you created an Azure Container Registry with the Azure portal, pushed a container image, and pulled and ran the image from the registry. Continue to the Azure Container Registry tutorials for a deeper look at ACR.

> [!div class="nextstepaction"]
> [Azure Container Registry tutorials][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Azure Container Registry Tasks tutorials][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
[container-registry-skus]: container-registry-skus.md
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-powershell-install]: /powershell/azure/install-az-ps
[get-started-with-azure-cli]: /cli/azure/get-started-with-azure-cli
[get-started-with-azure-powershell]: /powershell/azure/get-started-azureps
[az-acr-login]: /cli/azure/acr#az_acr_login
[connect-azcontainerregistry]: /powershell/module/az.containerregistry/connect-azcontainerregistry
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
