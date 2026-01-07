---
title: Cross-Tenant Authentication from AKS to ACR
description: Configure an AKS cluster's service principal with permissions to access your Azure container registry in a different Microsoft Entra tenant.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 01/07/2026
ms.custom: sfi-image-nochange
# Customer intent: As a developer, I want to configure cross-tenant authentication for an AKS cluster to access an Azure container registry, so that I can enable seamless pulling of container images across different Microsoft Entra tenants.
---

# Pull images from a container registry to an AKS cluster in a different Microsoft Entra tenant

If you have an Azure Kubernetes Service (AKS) cluster in one Microsoft Entra tenant, and an Azure container registry in a different tenant, you can configure cross-tenant authentication to enable the AKS cluster to pull images from the container registry. This article walks through the steps to enable cross-tenant authentication by using the AKS service principal credential to pull from the container registry.

In this article, we refer to the two tenants as **Tenant A** and **Tenant B**.* The AKS cluster is in **Tenant A** and the Azure container registry is in **Tenant B**.

The high-level steps to enable cross-tenant authentication are:

1. Create a new multitenant app (service principal) in **Tenant A**.
1. Provision the app in **Tenant B**.
1. Configure the service principal to pull from the registry in **Tenant B**.
1. Update the AKS cluster in **Tenant A** to authenticate by using the new service principal.

> [!NOTE]
> When the cluster and the container registry are in different tenants, you can't use an AKS managed identity to attach the registry and authenticate.

## Prerequisites



The AKS cluster must be configured with [service principal authentication](/azure/aks/kubernetes-service-principal) in **Tenant A**.

You need at least the **Contributor** role for the AKS cluster's subscription. You also need the **Role Based Access Control Administrator** and **Container Registry Contributor and Data Access Configuration Administrator** roles in the container registry's subscription (or roles with an equivalent or greater level of access).

<a name='step-1-create-multitenant-azure-ad-application'></a>

## Create a multitenant Microsoft Entra application

1. Sign in to the [Azure portal](https://portal.azure.com/) in **Tenant A**.
1. Search for and select **Microsoft Entra ID**.
1. In the service menu, under **Manage**, select **App registrations**.
1. Select **+ New registration**.
1. Enter a name for the application.
1. In **Supported account types**, select **Accounts in any organizational directory**.
1. In **Redirect URI**, select **Web** for **Platform** and enter *https://www.microsoft.com*.
1. Select **Register**.
1. On the **Overview** page, take note of the **Application (client) ID**. You'll need this ID later.

    :::image type="content" source="media/authenticate-aks-cross-tenant/service-principal-overview.png" alt-text="Service principal application ID":::

1. In the service menu, under **Manage**, select **Certificates & secrets**.
1. In the **Client secrets** section, select **+ New client secret**.
1. Enter a **Description** such as *Password*, and then select **Add**.
1. In **Client secrets**, take note of the value of the client secret. You'll use this value to update the AKS cluster's service principal.

## Provision the service principal in the ACR tenant

1. Edit the following link with the tenant ID for **Tenant B** and the application (client) ID of the multitenant app.

    ```console
    https://login.microsoftonline.com/<Tenant B ID>/oauth2/authorize?client_id=<Multitenant application ID>&response_type=code&redirect_uri=<redirect url>
    ```

1. Open the edited link with an admin account in **Tenant B**.

1. Select **Consent on behalf of your organization** and then **Accept**.

## Configure the service principal to pull from registry

In **Tenant B**, assign the correct role to the service principal, scoped to the target container registry. For [ABAC-enabled registries](container-registry-rbac-abac-repository-permissions.md), assign `Container Registry Repository Reader`. For non-ABAC registries, assign `AcrPull`.

You can use the [Azure portal](/azure/role-based-access-control/role-assignments-portal), [the Azure CLI](container-registry-auth-service-principal.md#use-an-existing-service-principal), or other tools to assign this role.

<a name='step-4-update-aks-with-the-azure-ad-application-secret'></a>

## Update the AKS cluster with the Microsoft Entra application secret

Use the multitenant app's application (client) ID and client secret to [update the AKS service principal credential](/azure/aks/update-credentials#update-aks-cluster-with-service-principal-credentials).

Updating the service principal can take several minutes.

## Related content

* Learn more about [Azure Container Registry authentication with service principals](container-registry-auth-service-principal.md).
* Learn more about image pull secrets in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod).
* Learn about [Application and service principal objects in Microsoft Entra ID](/azure/active-directory/develop/app-objects-and-service-principals).
* Learn more about [scenarios to authenticate with Azure Container Registry](authenticate-kubernetes-options.md) from a Kubernetes cluster.
