---
title: "Workload identity federation in Azure Arc-enabled Kubernetes (preview)"
ms.date: 11/11/2024
ms.topic: concept-article
ms.custom:
  - ignite-2024
description: "Learn how workload identity federation can be used with Azure Arc-enabled Kubernetes clusters."
# Customer intent: As a Kubernetes administrator, I want to implement workload identity federation in Azure Arc-enabled Kubernetes clusters, so that I can securely manage identities without the risk of credential leakage or expiration, thereby enhancing the security and stability of my applications.
---

# Workload identity federation in Azure Arc-enabled Kubernetes (preview)

Software workloads running on Kubernetes clusters need an identity in order to authenticate and access resources or communicate with other services. For a software workload running outside of Azure, you need to use application credentials, such as a secret or certificate, to access resources that are protected by Microsoft Entra (such as Azure Key Vault or Azure Blob storage). These credentials pose a security risk and have to be stored securely and rotated regularly. You also run the risk of service downtime if the credentials expire.

Workload identity federation lets you configure a [user-assigned managed identity](/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities) or [app registration](/entra/identity-platform/app-objects-and-service-principals) in Microsoft Entra ID to trust tokens from an external identity provider (IdP), such as Kubernetes. The user-assigned managed identity or app registration in Microsoft Entra ID becomes an identity for software workloads running on Arc-enabled Kubernetes clusters. Once that trust relationship is created, your workload can exchange trusted tokens from the Arc-enabled Kubernetes clusters for access tokens from Microsoft identity platform. Your software workload uses that access token to access the resources protected by Microsoft Entra. With workload identity federation, you can thus eliminate the maintenance burden of manually managing credentials, and eliminate the risk of leaking secrets or having certificates expire.

> [!IMPORTANT]
> The Azure Arc workload identity federation feature is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How workload identity works with Azure Arc-enabled Kubernetes clusters

Workload identity support for Azure Arc enabled Kubernetes uses [Service Account Token Volume Projection](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#serviceaccount-token-volume-projection) (that is, a service account) so that workload pods can use a Kubernetes identity. A Kubernetes token is issued, and [OpenID Connect (OIDC) federation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens) lets Kubernetes applications access Azure resources securely with Microsoft Entra ID, based on annotated service accounts.

The Arc-enabled Kubernetes cluster acts as the token issuer. Microsoft Entra ID uses OIDC to discover public signing keys and verify the authenticity of the service account token before exchanging it for a Microsoft Entra token. Your workload can exchange a service account token projected to its volume for a Microsoft Entra token using the Azure Identity client library or the Microsoft Authentication Library (MSAL).

:::image type="content" source="media/conceptual-workload-identity/workload-identity-arc-enabled-kubernetes.jpg" alt-text="Diagram showing the process flow for the workload identity feature for Azure Arc-enabled Kubernetes.":::

The following table shows the required OIDC issuer endpoints for Microsoft Entra Workload ID.

|Endpoint  |Description  |
|---------|---------|
|`{IssuerURL}/.well-known/openid-configuration`     |  Also known as the OIDC discovery document. Contains the metadata about the issuer's configurations.        |
|`{IssuerURL}/openid/v1/jwks`    |Contains the public signing key(s) that Microsoft Entra ID uses to verify the authenticity of the service account token.          |

## Service account labels and annotations

[Microsoft Entra Workload ID](/azure/active-directory/develop/workload-identities-overview) supports the following mappings related to a service account:

- **One-to-one**: A service account references a Microsoft Entra object.
- **Many-to-one**: Multiple service accounts reference the same Microsoft Entra object.
- **One-to-many**: A service account references multiple Microsoft Entra objects by changing the client ID annotation. For more information, see [How to federate multiple identities with a Kubernetes service account](https://azure.github.io/azure-workload-identity/docs/faq.html#how-to-federate-multiple-identities-with-a-kubernetes-service-account).

## Requirements

Azure Arc-enabled Kubernetes clusters support Microsoft Entra Workload Identity [starting with agent version 1.21](release-notes.md).

To use the workload identity feature, you must have Azure CLI version 2.64 or higher, and `az connectedk8s` version 1.10.0 or higher. Be sure to update your Azure CLI version before updating your `az connectedk8s` version. If you use Azure Cloud Shell, the latest version of Azure CLI will be installed.

Microsoft Entra Workload ID works especially well with the [Azure Identity client libraries](/azure/aks/workload-identity-overview?tabs=dotnet#azure-identity-client-libraries) or the [Microsoft Authentication Library (MSAL)](/azure/active-directory/develop/msal-overview) collection, together with application registration. Your workload can use any of these libraries to seamlessly authenticate and access Azure cloud resources.

For more information about integration with popular libraries, see [Use Microsoft Entra Workload ID with AKS](/azure/aks/workload-identity-overview?bs=dotnet#azure-identity-client-libraries).

## Current limitations

Keep in mind the following current limitations:

- A maximum of 20 federated identity credentials per managed identity can be configured.
- After the federal identity credential is initially added, it takes a few seconds for it to be propagated.
- Creation of federated identity credentials is not supported on user-assigned managed identities in [certain regions](/entra/workload-id/workload-identity-federation-considerations#unsupported-regions-user-assigned-managed-identities).

## Next steps

- Learn how to [enable workload identity on your Azure Arc enabled Kubernetes cluster and configure your application to use workload identity](workload-identity.md).
- Help to protect your cluster in other ways by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).
