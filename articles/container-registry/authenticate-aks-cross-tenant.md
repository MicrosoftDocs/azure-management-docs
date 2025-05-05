---
title: Cross-Tenant Authentication from AKS to ACR
description: Configure an AKS cluster's service principal with permissions to access your Azure container registry in a different Microsoft Entra tenant.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 02/28/2025
#customer intent: As a developer, I want to configure an AKS cluster's service principal with permissions to access my Azure container registry in a different Microsoft Entra tenant so that I can pull images from the registry.
---

# Pull images from a container registry to an AKS cluster in a different Microsoft Entra tenant

In some cases, you might have your Azure AKS cluster in one Microsoft Entra tenant and your Azure container registry in a different tenant. This article walks through the steps to enable cross-tenant authentication using the AKS service principal credential to pull from the container registry.

> [!NOTE]
> You can't attach the registry and authenticate using an AKS managed identity when the cluster and the container registry are in different tenants.

## Scenario overview

Assumptions for this example:

* The AKS cluster is in **Tenant A** and the Azure container registry is in **Tenant B**. 
* The AKS cluster is configured with service principal authentication in **Tenant A**. For more information, see [Create and use a service principal for your AKS cluster](/azure/aks/kubernetes-service-principal).

You need at least the **Contributor** role in the AKS cluster's subscription. You will also need at least the **Role Based Access Control Administrator** and **Container Registry Contributor and Data Access Configuration Administrator** roles in the container registry's subscription.

Use the following steps to:

1. Create a new multitenant app (service principal) in **Tenant A**. 
1. Provision the app in **Tenant B**.
1. Configure the service principal to pull from the registry in **Tenant B**.
1. Update the AKS cluster in **Tenant A** to authenticate using the new service principal.

## Step-by-step instructions

<a name='step-1-create-multitenant-azure-ad-application'></a>

### Step 1: Create multitenant Microsoft Entra application

1. Sign in to the [Azure portal](https://portal.azure.com/) in **Tenant A**.
1. Search for and select **Microsoft Entra ID**.
1. Under **Manage**, select **App registrations > + New registration**.
1. In **Supported account types**, select **Accounts in any organizational directory**.
1. In **Redirect URI**, enter *https://www.microsoft.com*.
1. Select **Register**.
1. On the **Overview** page, take note of the **Application (client) ID**. You use this ID in Step 2 and Step 4.

    :::image type="content" source="media/authenticate-kubernetes-cross-tenant/service-principal-overview.png" alt-text="Service principal application ID":::
1. In **Certificates & secrets**, under **Client secrets**, select **+ New client secret**.
1. Enter a **Description** such as *Password* and select **Add**.
1. In **Client secrets**, take note of the value of the client secret. You use it to update the AKS cluster's service principal in Step 4.

    :::image type="content" source="media/authenticate-kubernetes-cross-tenant/configure-client-secret.png" alt-text="Configure client secret":::

### Step 2: Provision the service principal in the ACR tenant

1. Open the following link using an admin account in **Tenant B**. Where indicated, insert the **ID of Tenant B** and the **application ID** (client ID) of the multitenant app.

    ```console
    https://login.microsoftonline.com/<Tenant B ID>/oauth2/authorize?client_id=<Multitenant application ID>&response_type=code&redirect_uri=<redirect url>
    ```

1. Select **Consent on behalf of your organization** and then **Accept**. 
    
    :::image type="content" source="media/authenticate-kubernetes-cross-tenant/multitenant-app-consent.png" alt-text="Grant tenant access to application":::

### Step 3: Grant service principal permission to pull from registry

In **Tenant B**, assign the correct role to the service principal, scoped to the target container registry. You must assign either `Container Registry Repository Reader` (for [ABAC-enabled registries](container-registry-rbac-abac-repository-permissions.md)) or `AcrPull` (for non-ABAC registries).

You can use the [Azure portal](/azure/role-based-access-control/role-assignments-portal) or other tools to assign the role. For example steps using the Azure CLI, see [Azure Container Registry authentication with service principals](container-registry-auth-service-principal.md#use-an-existing-service-principal).

:::image type="content" source="media/authenticate-kubernetes-cross-tenant/multitenant-app-acr-pull.png" alt-text="Screenshot of assigning role to multitenant app.":::

<a name='step-4-update-aks-with-the-azure-ad-application-secret'></a>

### Step 4: Update AKS with the Microsoft Entra application secret

Use the multitenant application (client) ID and client secret you collected in Step 1 to [update the AKS service principal credential](/azure/aks/update-credentials#update-aks-cluster-with-service-principal-credentials).

Updating the service principal can take several minutes.

## Next steps

* Learn more about [Azure Container Registry authentication with service principals](container-registry-auth-service-principal.md).
* Learn more about image pull secrets in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod).
* Learn about [Application and service principal objects in Microsoft Entra ID](/azure/active-directory/develop/app-objects-and-service-principals).
* Learn more about [scenarios to authenticate with Azure Container Registry](authenticate-kubernetes-options.md) from a Kubernetes cluster.
