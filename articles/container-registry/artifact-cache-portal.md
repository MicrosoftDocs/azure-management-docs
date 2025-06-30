---
title: "Enable artifact cache in your Azure Container Registry with Azure portal"
description: "Learn how to use the Azure portal to cache container images in Azure Container Registry, improving performance and efficiency."
author: chasedmicrosoft
ms.author: doveychase
ms.service: azure-container-registry
ms.topic: how-to
ms.date: 04/29/2025
ai-usage: ai-assisted
# Customer intent: "As a developer, I want to enable artifact caching in Azure Container Registry so that I can improve the performance and efficiency of delivering containerized applications."
---

# Enable artifact cache in your Azure Container Registry with Azure portal

In this article, you learn how to use the Azure portal to enable the [artifact cache feature](artifact-cache-overview.md) in your Azure Container Registry (ACR).

In addition to the prerequisites listed here, you need an Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## Prerequisites

* An existing ACR instance. If you don't have one, [use our quickstart to create a new container registry](/azure/container-registry/container-registry-get-started-azure-cli).
* An existing Key Vault to [create and store credentials][create-and-store-keyvault-credentials].
* Permissions to [set and retrieve secrets from your Key Vault][set-and-retrieve-a-secret].
* Azure CLI. You can use the [Azure Cloud Shell][Azure Cloud Shell] or a local installation of the Azure CLI to run the commands in this article. Use Azure CLI version 2.46.0 or later to run it locally. To confirm your Azure CLI version, run `az --version`. To install or upgrade, see [Install Azure CLI][Install Azure CLI].

## Configure Artifact cache

To create and configure the cache rule that pulls artifacts from the repository into your cache, follow these steps.

Follow the steps to create a cache rule in the [Azure portal](https://portal.azure.com).

1. Navigate to your Azure Container Registry instance.

1. In the service menu, under **Services**, select **Cache**.

1. Select **Create rule**.

    :::image type="content" source="./media/container-registry-artifact-cache/artifact-cache-create-rule.png" alt-text="Screenshot showing the Create rule command for a container registry in the Azure portal.":::

1. In the **New cache rule** pane, enter a **Rule name**.

1. For **Source**, select a login server.

1. For **Repository Path**, enter the full repository path to the artifacts you want to cache.

1. Depending on your source, **Authentication** might be required. If the **Authentication** box isn't already checked, and you don't want to use authentication, you can skip this section. Otherwise, ensure the box is checked and add your credentials:

   * Select **Create new credentials** to create a new set of credentials to store the username and password for your source registry. For more information, see [create new credentials](tutorial-enable-artifact-cache-auth.md#create-new-credentials).
   * To use existing credentials, choose **Select credentials** from the drop-down menu.

1. For **Destination**, enter the name of the **New ACR repository namespace** to store cached artifacts.

1. Select **Create** to create your cache rule.

   :::image type="content" source="media/container-registry-artifact-cache/artifact-cache-enter-rule-details.png" alt-text="Screenshot showing details entered to create a new cache rule for a container registry in the Azure portal.":::

## Create new credentials

Before configuring the credentials, make sure you're able to [create and store secrets in the Azure Key Vault][create-and-store-keyvault-credentials] and [retrieve secrets from the Key Vault][set-and-retrieve-a-secret].

1. In your container registry's **Cache** pane, select **Credentials**, then select **Create credentials**.

   :::image type="content" source="./media/container-registry-artifact-cache/artifact-cache-create-credentials.png" alt-text="Screenshot of the steps to start adding credentials for a container registry in Azure portal.":::

1. Enter a **Name** for the new credentials for your source registry.
1. Select a **Source Authentication**. Artifact cache currently supports **Select from Key Vault** and **Enter secret URIs**.
1. For the  **Select from Key Vault** option, [create your credentials using Key Vault][create-and-store-keyvault-credentials].
1. Select **Create**.

      :::image type="content" source="./media/container-registry-artifact-cache/artifact-cache-enter-credential-details.png" alt-text="Screenshot showing details entered to create credentials for a container registry in Azure portal.":::

Alternately, you can use Azure RBAC to assign the **Key Vault Secrets User** role (or a custom role that includes the `Microsoft.KeyVault/vaults/secrets/getSecret/action` permission) to the system identity. For more information, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal) and [Grant permission to applications to access an Azure Key Vault using Azure RBAC](/azure/key-vault/general/rbac-guide?tabs=azure-portal).

## Next steps

* Learn about [troubleshooting issues with artifact caching](troubleshoot-artifact-cache.md).
* Learn how to [enable artifact cache using the Azure CLI](artifact-cache-cli.md).

<!-- LINKS - External -->
[create-and-store-keyvault-credentials]: /azure/key-vault/secrets/quick-create-cli#add-a-secret-to-key-vault
[set-and-retrieve-a-secret]: /azure/key-vault/secrets/quick-create-cli#retrieve-a-secret-from-key-vault
[Install Azure CLI]: /cli/azure/install-azure-cli
[Azure Cloud Shell]: /azure/cloud-shell/quickstart
