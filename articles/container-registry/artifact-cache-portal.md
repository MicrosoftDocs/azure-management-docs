---
title: "Enable artifact cache in your Azure Container Registry with Azure portal"
description: "Learn how to use the Azure portal to cache container images in Azure Container Registry, improving performance and efficiency."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to
ms.date: 02/26/2025
ai-usage: ai-assisted
#customer intent: As a developer, I want Artifact cache capabilities so that I can efficiently deliver and serve containerized applications to end-users in real-time.
---

# Enable artifact cache in your Azure Container Registry with Azure portal

In this article, you learn how to use the Azure portal to enable the [artifact cache feature](artifact-cache-overview.md) in your Azure Container Registry (ACR).

In addition to the prerequisites listed here, you need an Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## Prerequisites

* An existing ACR instance. If you don't already have one, [use our quickstart to create a new container registry](/azure/container-registry/container-registry-get-started-azure-cli).
* An existing Key Vault to [create and store credentials][create-and-store-keyvault-credentials].
* Permissions to [set and retrieve secrets from your Key Vault][set-and-retrieve-a-secret].
* Azure CLI. You can use the [Azure Cloud Shell][Azure Cloud Shell] or a local installation of the Azure CLI to run the commands in this article. If you'd like to use it locally, Azure CLI version 2.46.0 or later is required. To confirm your Azure CLI version, run `az --version`. To install or upgrade, see [Install Azure CLI][Install Azure CLI].

## Configure Artifact cache

Follow these steps to create and configure the cache rule that will be used to pull artifacts from the repository into your cache.

Follow the steps to create a cache rule in the [Azure portal](https://portal.azure.com).

1. Navigate to your Azure Container Registry instance.

1. In the service menu, under **Services**, select **Cache**.

    :::image type="content" source="./media/container-registry-artifact-cache/cache-preview-01.png" alt-text="Screenshot showing the Cache option in the service menu of a container registry in the Azure portal.":::

1. Select **Create rule**.

    :::image type="content" source="./media/container-registry-artifact-cache/cache-blade-02.png" alt-text="Screenshot showing the Create rule command for a container registry in the Azure portal.":::

1. A window for **New cache rule** appears.

    :::image type="content" source="./media/container-registry-artifact-cache/new-cache-rule-03.png" alt-text="Screenshot for new Cache Rule in Azure portal.":::

1. Enter the **Rule name**.

1. Select **Source** Registry from the dropdown menu.

1. Enter the **Repository Path** to the artifacts you want to cache.

1. Depending on your source, **Authentication** may be required. If the **Authentication** box isn't already checked, and you don't want to use authentication, you can skip this section. Otherwise, ensure the box is checked and add your credentials:

   * Select **Create new credentials** to create a new set of credentials to store the username and password for your source registry. For more information, see [create new credentials](tutorial-enable-artifact-cache-auth.md#create-new-credentials).
   * To use existing credentials, choose **Select credentials** from the drop-down menu.

1. For **Destination**, enter the name of the **New ACR repository namespace** to store cached artifacts.

    :::image type="content" source="./media/container-registry-artifact-cache/save-cache-rule-04.png" alt-text="Screenshot showing cache rule creation for a container registry in Azure portal.":::

1. Select **Save**.

## Create new credentials

Before configuring the credentials, make sure you're able to [create and store secrets in the Azure Key Vault][create-and-store-keyvault-credentials] and [retrieve secrets from the Key Vault.][set-and-retrieve-a-secret].

1. In the **Cache** pane, select **Credentials**, then select **Create credentials**.

   :::image type="content" source="./media/container-registry-artifact-cache/add-credential-set-05.png" alt-text="Screenshot for adding credentials in Azure portal.":::code language="{language}" source="{source}" range="{range}":::

   :::image type="content" source="./media/container-registry-artifact-cache/create-credential-set-06.png" alt-text="Screenshot for create new credentials in Azure portal.":::

1. Enter a **Name** for the new credentials for your source registry.
1. Select a **Source Authentication**. Artifact cache currently supports **Select from Key Vault** and **Enter secret URIs**.
1. For the  **Select from Key Vault** option, [create your credentials using Key Vault][create-and-store-keyvault-credentials].
1. Select **Create**.

## Next steps

* Learn about troubleshooting issues with artifact caching (troubleshoot-artifact-cache.md).
* Learn how to [enable artifact cache using the Azure CLI](artifact-cache-cli.md).

<!-- LINKS - External -->
[create-and-store-keyvault-credentials]: /azure/key-vault/secrets/quick-create-cli#add-a-secret-to-key-vault
[set-and-retrieve-a-secret]: /azure/key-vault/secrets/quick-create-cli#retrieve-a-secret-from-key-vault
[Install Azure CLI]: /cli/azure/install-azure-cli
[Azure Cloud Shell]: /azure/cloud-shell/quickstart
