---
title: Configure Registry Acceptance of Microsoft Entra Authentication Scopes
description: Configure Azure Container Registry to accept ARM-scoped Microsoft Entra authentication (broad) or ACR-scoped Microsoft Entra authentication (narrow) for enhanced security.
ms.author: rayoflores
ms.service: azure-container-registry
ms.custom: devx-track-arm-template, devx-track-azurecli
ms.topic: tutorial  #Don't change.
ms.date: 10/31/2023
# Customer intent: As a DevOps engineer, I want to configure my Azure Container Registry to control which Microsoft Entra authentication scopes are accepted, so that I can enhance security by restricting to ACR-scoped Microsoft Entra authentication only.
---

# Configure registry acceptance of Microsoft Entra authentication scopes

Microsoft Entra authentication with Azure Container Registry (ACR) follows a two-hop process. First, identities authenticate with Microsoft Entra ID to obtain an authentication token (for example, `az login`) with a specific authentication audience scope. Second, identities then authenticate with the registry using that Microsoft Entra authentication token (for example, `az acr login`). By default, in the second step during registry authentication, registries accept both ARM-scoped Microsoft Entra authentication (broad Azure Resource Manager access) and ACR-scoped Microsoft Entra authentication (narrow registry-specific access). This article explains how to configure your registry to accept only ACR-scoped Microsoft Entra authentication for enhanced security.

> [!NOTE]
> Some Azure services and integrations might not work when your registry is configured to accept only ACR-scoped Microsoft Entra authentication. Test compatibility in nonproduction environments before enforcing this configuration in production.

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Configure the registry for ACR-scoped Microsoft Entra authentication using Azure CLI.
> * Configure the registry for ACR-scoped Microsoft Entra authentication using the Azure portal.

## Prerequisites

- [Install or upgrade Azure CLI](/cli/azure/install-azure-cli) version 2.40.0 or later. To find the version, run `az --version`.
- Sign in to the [Azure portal](https://portal.azure.com).

## Understanding Microsoft Entra authentication scopes for ACR

This section explains the two-hop authentication flow, the difference between authentication scopes, and how registry configuration controls access.

### Two-hop authentication flow

Authentication with ACR involves two distinct steps that apply to Azure CLI users, Azure services, and programmatic code using Software Development Kits (SDKs):

1. **First hop - Authenticate with Microsoft Entra ID**: Run `az login` to authenticate with Microsoft Entra ID and obtain a Microsoft Entra authentication token. This token contains an audience scope that determines what Azure resources it can access:
   - By default, `az login` obtains an ARM-scoped Microsoft Entra authentication token (broad Azure Resource Manager access).
   - With `az login --scope https://containerregistry.azure.net/.default`, you obtain an ACR-scoped Microsoft Entra authentication token (narrow registry-specific access).

1. **Second hop - Authenticate with the ACR service**: Run `az acr login` to authenticate with the ACR service using the Microsoft Entra authentication token from the first hop. At this point, the registry's configuration determines whether to accept or reject your registry authentication attempt based on the scope of your Microsoft Entra authentication token.

### Microsoft Entra authentication scope types

**ARM-scoped Microsoft Entra authentication**

- Provides broad access to the Azure Resource Manager (ARM), the control plane for managing all Azure resources.
- This scope type is the default when you run `az login` with no other parameters.
- ARM-scoped Microsoft Entra authentication tokens are overly permissive for container registry operations as they grant access beyond what you need for registry authentication.

**ACR-scoped Microsoft Entra authentication**

- Provides narrow, registry-specific access limited to ACR operations only.
- Requires explicitly specifying `--scope https://containerregistry.azure.net/.default` during `az login`.
- Follows the principle of least privilege by granting only the permissions necessary for container registry operations.

### Registry configuration options

Your registry configuration (controlled by the `azureADAuthenticationAsArmPolicy` property) determines what happens during the second authentication hop (`az acr login`):

- **When enabled** (default): The registry accepts both ARM-scoped and ACR-scoped Microsoft Entra authentication.
- **When disabled** (recommended): The registry accepts only ACR-scoped Microsoft Entra authentication and rejects ARM-scoped authentication.

### Why ACR recommends ACR-scoped authentication only

Configuring your registry to accept only ACR-scoped Microsoft Entra authentication provides several benefits:

- **Enhanced security**: Limits authentication to narrowly scoped Microsoft Entra authentication tokens, reducing the attack surface.
- **Least privilege access**: Ensures Microsoft Entra authentication tokens used for registry authentication have only the permissions necessary for container registry operations.
- **Compliance alignment**: Helps meet security and compliance requirements that mandate minimal privilege access patterns.
- **Best practice**: Aligns with Azure security best practices for identity and access management.

## Configure registry for ACR-scoped Microsoft Entra authentication - Azure CLI

You can configure your registry to control which Microsoft Entra authentication scopes are accepted during the second authentication hop. The `azureADAuthenticationAsArmPolicy` property controls this configuration.

Requires Azure CLI version 2.40.0 or later. To find your version, run `az --version`.

### Check current registry configuration

Run the following command to view your registry's current configuration:

```azurecli
az acr config authentication-as-arm show -r <registry>
```

The status value indicates the current configuration:

- **enabled** (default): Registry accepts both ARM-scoped and ACR-scoped Microsoft Entra authentication.
- **disabled** (recommended): Registry accepts only ACR-scoped Microsoft Entra authentication.

### Update registry configuration

To configure your registry to accept only ACR-scoped Microsoft Entra authentication, set the status to `disabled`:

```azurecli
az acr config authentication-as-arm update -r <registry> --status disabled
```

To revert to the default configuration that accepts both ARM-scoped and ACR-scoped Microsoft Entra authentication:

```azurecli
az acr config authentication-as-arm update -r <registry> --status enabled
```

## Authenticate using ACR-scoped Microsoft Entra authentication

This section demonstrates how to perform the two-hop authentication flow using ACR-scoped Microsoft Entra authentication. This authentication method works with both registry configurations, but is required when your registry is configured to accept only ACR-scoped Microsoft Entra authentication.

### First hop: Obtain ACR-scoped Microsoft Entra authentication token

To obtain an ACR-scoped Microsoft Entra authentication token, specify the `--scope` parameter when running `az login`:

```azurecli
az login --scope https://containerregistry.azure.net/.default
```

This command authenticates you with Microsoft Entra ID and obtains a Microsoft Entra authentication token with ACR-specific scope. The authentication token is stored in your local cache.

> [!NOTE]
> You must specify `https://containerregistry.azure.net/.default` to obtain a Microsoft Entra authentication token scoped for the ACR service.
> You can't specify `https://registryname.azurecr.io/` as the scope, as Microsoft Entra ID and ACR don't support registry-specific Microsoft Entra authentication audiences.

### Second hop: Authenticate with the registry

After obtaining the ACR-scoped Microsoft Entra authentication token in the first hop, authenticate with your registry:

```azurecli
az acr login -n <registry>
```

During this second hop, the registry validates your Microsoft Entra authentication token. If your registry is configured to accept only ACR-scoped authentication, it rejects ARM-scoped tokens and only accepts the ACR-scoped Microsoft Entra authentication token you obtained in the first hop.

The ACR-scoped Microsoft Entra authentication token can be used to authenticate with all ACR registries that you have permissions to access.

## Configure registry for ACR-scoped Microsoft Entra authentication - Azure portal

You can use Azure Policy to configure your registries to accept only ACR-scoped Microsoft Entra authentication. Assigning a built-in policy automatically configures the registry property for current and future registries within the policy scope. You can apply the policy at either the Resource Group level or the Subscription level.

Azure Container Registry provides two built-in policy definitions for configuring ACR-scoped Microsoft Entra authentication:

- **`Container registries should have ARM audience token authentication disabled.`** - This policy reports and blocks noncompliant resources, and can automatically update noncompliant registries to the recommended configuration.
- **`Configure container registries to disable ARM audience token authentication.`** - This policy offers remediation capabilities and updates noncompliant registries to accept only ACR-scoped authentication.

For more information about ACR's built-in policy definitions, see [azure-container-registry-built-in-policy definitions](policy-reference.md).

### Assign a built-in policy definition

To configure your registries to accept only ACR-scoped Microsoft Entra authentication using Azure Policy:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Navigate to your **Azure Container Registry** > **Resource Group** > **Settings** > **Policies**.

   :::image type="content" source="media/container-registry-enable-conditional-policy/01-azure-policies.png" alt-text="Screenshot showing how to navigate Azure policies.":::

1. Navigate to  **Azure Policy**, On the **Assignments** tab, select **Assign policy**.

   :::image type="content" source="media/container-registry-enable-conditional-policy/02-Assign-policy.png" alt-text="Screenshot showing how to assign a policy.":::

1. On the **Assign policy** tab, use filters to search and find the **Scope**, **Policy definition**, and **Assignment name**.

   :::image type="content" source="media/container-registry-enable-conditional-policy/03-Assign-policy-tab.png" alt-text="Screenshot of the assign policy tab.":::

1. Select **Scope** to filter and search for the **Subscription** and **ResourceGroup** and choose **Select**.

   :::image type="content" source="media/container-registry-enable-conditional-policy/04-select-scope.png" alt-text="Screenshot of the Scope tab.":::

1. Select **Policy definition** to filter and search for the built-in policy definitions that configure ACR-scoped authentication.

   :::image type="content" source="media/container-registry-enable-conditional-policy/05-built-in-policy-definitions.png" alt-text="Screenshot of built-in-policy-definitions.":::

1. Use filters to select and confirm **Scope**, **Policy definition**, and **Assignment name**.

1. Use the filters to limit compliance states or to search for policies.

1. Confirm your settings and set policy enforcement as **enabled**. This setting ensures the policy configures registries within the scope to accept only ACR-scoped Microsoft Entra authentication.

1. Select **Review+Create**.

   :::image type="content" source="media/container-registry-enable-conditional-policy/06-enable-policy.png" alt-text="Screenshot showing how to activate a Conditional Access policy.":::

## Next steps

> [!div class="nextstepaction"]
> [Create and configure a Conditional Access policy](container-registry-configure-conditional-access.md)
