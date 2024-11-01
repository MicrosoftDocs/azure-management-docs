---
title: "Workload identity in Azure Arc-enabled Kubernetes (preview)"
ms.date: 11/01/2024
ms.topic: conceptual
description: "Learn how workload identity federation can be used with Azure Arc-enabled Kubernetes clusters.."
---

# Workload identity in Azure Arc-enabled Kubernetes (preview)

Software workloads running on Kubernetes clusters needs an identity in order to authenticate and access resources or communicate with other services. For a software workload running outside of Azure, you need to use application credentials (a secret or certificate) to access Microsoft Entra protected resources (such as Azure Key Vault, Azure Blob storage). These credentials pose a security risk and have to be stored securely and rotated regularly. You also run the risk of service downtime if the credentials expire. 

Workload identity federation can be used to configure a user-assigned managed identity or app registration in Microsoft Entra ID to trust tokens from an external identity provider (IdP), such as Kubernetes. The user-assigned managed identity or app registration in Microsoft Entra ID becomes an identity for software workloads running on Arc enabled Kubernetes clusters. Once that trust relationship is created, your workload can exchange trusted tokens from the Arc enabled kubernetes clusters for access tokens from Microsoft identity platform. Your software workload uses that access token to access the Microsoft Entra protected resources to which the workload has been granted access. With Workload Identity Federation, you can thus eliminate the maintenance burden of manually managing credentials and eliminate the risk of leaking secrets or having certificates expire.

> [!IMPORTANT]
> The Azure Arc workload identity federation feature is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How workload identity works with Azure Arc-enabled Kubernetes clusters

Microsoft Entra Workload ID uses Service Account Token Volume Projection (that is, a service account), to enable workload pods to use a Kubernetes identity. A Kubernetes token is issued and OIDC federation enables Kubernetes applications to access Azure resources securely with Microsoft Entra ID, based on annotated service accounts. 

Arc enabled Kubernetes cluster acts as the token issuer. Microsoft Entra ID uses OpenID Connect to discover public signing keys and verify the authenticity of the service account token before exchanging it for a Microsoft Entra token. Your workload can exchange a service account token projected to its volume for a Microsoft Entra token using the Azure Identity client library or the Microsoft Authentication Library (MSAL). 

### Diagram TBD

he following table describes the required OIDC issuer endpoints for Microsoft Entra Workload ID: 

Expand table 

Endpoint 

Description 

{IssuerURL}/.well-known/openid-configuration 

Also known as the OIDC discovery document. This contains the metadata about the issuer's configurations. 

{IssuerURL}/openid/v1/jwks 

This contains the public signing key(s) that Microsoft Entra ID uses to verify the authenticity of the service account token. 

The following diagram summarizes the authentication sequence using OpenID Connect. 

### Another diagram TBD

## Service account labels and annotations

Microsoft Entra Workload ID supports the following mappings related to a service account: 

One-to-one, where a service account references a Microsoft Entra object. 

Many-to-one, where multiple service accounts reference the same Microsoft Entra object. 

One-to-many, where a service account references multiple Microsoft Entra objects by changing the client ID annotation. For more information, see How to federate multiple identities with a Kubernetes service account. 

## Requirements 

Azure Arc enabled Kubernetes clusters supports Microsoft Entra Workload Identity on version 1.21 and higher. 

The Azure CLI version 2.47.0 or later. Run az --version to find the version, and run az upgrade to upgrade the version. If you need to install or upgrade, see Install Azure CLI. 

 Microsoft Entra Workload ID works especially well with the Azure Identity client libraries or the Microsoft Authentication Library (MSAL) collection, together with application registration. Your workload can use any of these libraries to seamlessly authenticate and access Azure cloud resources. 

This article helps you to understand integration with popular libraries - Use a Microsoft Entra Workload ID on AKS - Azure Kubernetes Service | Microsoft Learn  

## Current limitations 

A maximum of 20 federated identity credentials per managed identity can be configured. 

It takes a few seconds for the federated identity credential to be propagated after being initially added. 

Creation of federated identity credentials is not supported on user-assigned managed identities in these regions. 

## Next steps

To learn how to enable workload identity on your Azure Arc enabled Kubernetes cluster and configure your application to use workload identity, see Deploy and configure workload identity on an Arc enabled Kubernetes cluster. 