---
title: Disable Authentication as ARM
description: Disabling azureADAuthenticationAsArmPolicy forces the registry to only recognize ACR audience Microsoft Entra tokens, enhancing the security of your container registries.
ms.author: doveychase
ms.service: azure-container-registry
ms.custom: devx-track-arm-template, devx-track-azurecli
ms.topic: tutorial  #Don't change.
ms.date: 10/31/2023
# Customer intent: As a DevOps engineer, I want to disable ARM audience token authentication in Azure Container Registry, so that I can restrict access to only Microsoft Entra ACR audience tokens and enhance the security of my container registries.
---

# Disable authentication as ARM for Microsoft Entra tokens

Microsoft Entra tokens are used when registry users authenticate with Azure Container Registry (ACR). By default, Azure Container Registry (ACR) accepts Microsoft Entra tokens with an audience scope set for Azure Resource Manager (ARM), a control plane management layer for managing Azure resources.

By configuring your registry to not recognize Microsoft Entra ARM Audience Tokens and only recognize Microsoft Entra ACR Audience tokens, you can enhance the security of your container registries during the authentication process by narrowing the scope of accepted tokens.

With ACR Audience Token enforcement, only Microsoft Entra Tokens with an audience scope set for ACR will be accepted during the registry authentication and sign-in process. This means that the previously accepted ARM Audience Tokens will no longer be valid for registry authentication, thereby enhancing the security of your container registries.

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Disable authentication-as-arm in ACR - Azure CLI.
> * Disable authentication-as-arm in the ACR - Azure portal.

## Prerequisites

* [Install or upgrade Azure CLI](/cli/azure/install-azure-cli) version 2.40.0 or later. To find the version, run `az --version`.
* Sign in to the [Azure portal](https://portal.azure.com).

## Disable authentication-as-arm in ACR - Azure CLI

Disabling `azureADAuthenticationAsArmPolicy` forces the registry to use ACR audience token. You can use Azure CLI version 2.40.0 or later, run `az --version` to find the version. 

1. Run the command to show the current configuration of the registry's policy for authentication using ARM tokens with the registry. If the status is `enabled`, then both ACRs and ARM audience tokens can be used for authentication. If the status is `disabled` it means only ACR's audience tokens can be used for authentication.

   ```azurecli-interactive
   az acr config authentication-as-arm show -r <registry>
   ```

1. Run the command to update the status of the registry's policy.

   ```azurecli-interactive
   az acr config authentication-as-arm update -r <registry> --status [enabled/disabled]
   ```

## Authenticating with ACR using Microsoft Entra ACR Audience Token

You can authenticate with ACR using Microsoft Entra ACR Audience Token.
To obtain a Microsoft Entra ACR Audience Token, specify `--scope https://containerregistry.azure.net/.default` when you run the `az login` command.

> [!NOTE]
> You must specify `https://containerregistry.azure.net/.default` to obtain a Microsoft Entra ACR Audience token scoped for the ACR service.
> You cannot specify `https://registryname.azurecr.io/` as the scope, as neither Microsoft Entra nor ACR supports registry-specific token audiences.

```azurecli
az login --scope https://containerregistry.azure.net/.default
```

After logging in, a Microsoft Entra ACR Audience token (scoped for the ACR service) will be stored in the local cache.
You can use this token to authenticate with all ACR registries that you have permissions to.

```azurecli
az acr login -n <registry>
``` 

## Disable authentication-as-arm in the ACR - Azure portal

Disabling `authentication-as-arm` property by assigning a built-in policy will automatically disable the registry property for the current and the future registries. This automatic behavior is for registries created within the policy scope. The possible policy scopes include either Resource Group level scope or Subscription ID level scope within the tenant.

You can disable authentication-as-arm in the ACR, by following below steps:

   1. Sign in to the [Azure portal](https://portal.azure.com).
   
   1. Refer to the ACR's built-in policy definitions in the [azure-container-registry-built-in-policy definition's](policy-reference.md).
   
   1. Assign a built-in policy to disable authentication-as-arm definition - Azure portal.

### Assign a built-in policy definition to disable ARM audience token authentication - Azure portal.
  
You can enable registry's Conditional Access policy in the [Azure portal](https://portal.azure.com). 

Azure Container Registry has two built-in policy definitions to disable authentication-as-arm, as below:

* `Container registries should have ARM audience token authentication disabled.` - This policy will report, block any non-compliant resources, and also sends a request to update non-compliant to compliant.
* `Configure container registries to disable ARM audience token authentication.` - This policy offers remediation and updates non-compliant to compliant resources.


   1. Sign in to the [Azure portal](https://portal.azure.com).

   1. Navigate to your **Azure Container Registry** > **Resource Group** > **Settings** > **Policies** .
   
      :::image type="content" source="media/container-registry-enable-conditional-policy/01-azure-policies.png" alt-text="Screenshot showing how to navigate Azure policies.":::

   1. Navigate to  **Azure Policy**, On the **Assignments**, select **Assign policy**.
      
      :::image type="content" source="media/container-registry-enable-conditional-policy/02-Assign-policy.png" alt-text="Screenshot showing how to assign a policy.":::

   1. Under the **Assign policy** , use filters to search and find the **Scope**, **Policy definition**, **Assignment name**.

      :::image type="content" source="media/container-registry-enable-conditional-policy/03-Assign-policy-tab.png" alt-text="Screenshot of the assign policy tab.":::

   1. Select **Scope** to filter and search for the **Subscription** and **ResourceGroup** and choose **Select**.
   
   
      :::image type="content" source="media/container-registry-enable-conditional-policy/04-select-scope.png" alt-text="Screenshot of the Scope tab.":::


   1. Select **Policy definition** to filter and search the built-in policy definitions for the Conditional Access policy.
      
      :::image type="content" source="media/container-registry-enable-conditional-policy/05-built-in-policy-definitions.png" alt-text="Screenshot of built-in-policy-definitions.":::


   1. Use filters to select and confirm  **Scope**, **Policy definition**, and **Assignment name**.

   1. Use the filters to limit compliance states or to search for policies.

   1. Confirm your settings and set policy enforcement as **enabled**.

   1. Select **Review+Create**.

      :::image type="content" source="media/container-registry-enable-conditional-policy/06-enable-policy.png" alt-text="Screenshot to activate a Conditional Access policy.":::


## Next steps

> [!div class="nextstepaction"]
> [Create and configure a Conditional Access policy](container-registry-configure-conditional-access.md)
