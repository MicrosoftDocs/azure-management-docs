---
title: Secure your Azure Container Registry deployment
description: Learn how to secure Azure Container Registry, with best practices for protecting your registry, images, and access.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-container-registry
ms.topic: best-practice
ms.custom: horz-security
ms.date: 09/28/2026
ai-usage: ai-generated
---

# Secure your Azure Container Registry deployment

Azure Container Registry is a managed registry service for storing and distributing container images and related artifacts, such as Helm charts and OCI artifacts. Because a registry is a trusted source for the images that run across your compute platforms, a compromised or misconfigured registry can propagate vulnerable or malicious images throughout your environment.

This article provides security recommendations to help protect your Azure Container Registry deployment.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

## Service-specific security

The registry sits at the boundary between your build pipelines and your runtime platforms, so its authentication surface and its public defaults deserve early attention.

- **Keep anonymous pull access disabled unless the registry is intentionally public**: Anonymous pull applies to every repository in the registry and lets unauthenticated clients pull any image, so enable it only for registries that host deliberately public content. For more information, see [Make your container registry content publicly available](anonymous-pull-access.md).
- **Keep the admin account disabled**: The admin account is a single set of registry-wide credentials with full permissions and is disabled by default. Authenticate with Microsoft Entra identities instead so that access is attributable and governed by role assignments. For more information, see [Admin account](container-registry-authentication.md#admin-account).
- **Configure the registry to require ACR-scoped Microsoft Entra tokens**: Disable the authentication-as-ARM policy so the registry rejects broadly scoped Azure Resource Manager tokens during `az acr login` and accepts only ACR-scoped tokens. Some Azure integrations, such as Azure App Service image pulls, rely on ARM-scoped credentials, so test compatibility in a nonproduction environment before you enforce this setting. For more information, see [Configure a registry for ACR-scoped tokens](container-registry-disable-authentication-as-arm.md).

## Network security

By default a registry is reachable over the public internet. Restricting network access is one of the most effective ways to reduce the attack surface, and the strongest controls require the Premium service tier.

- **Connect to the registry through a private endpoint**: Use Azure Private Link to assign the registry a private IP address in your virtual network and keep pull and push traffic off the public internet. Private endpoints require the Premium service tier. For more information, see [Connect privately to a container registry using Azure Private Link](container-registry-private-endpoints.md).
- **Disable public network access**: After private connectivity is in place, set public network access to disabled so the registry is reachable only through its private endpoints. For more information, see [Disable public access to a registry](container-registry-private-endpoints.md#disable-public-access-to-a-registry).
- **Restrict access to selected public IP addresses when public access is required**: Configure IP network rules to allow only known public IP ranges, such as your build agents or egress addresses, and deny all others. Selected-network rules require the Premium service tier. For more information, see [Configure public IP network rules](container-registry-access-selected-networks.md).
- **Allow only the trusted Azure services that need registry access**: The trusted-services bypass lets select Microsoft services reach a network-restricted registry. Enable it only for services you actually use, such as Azure Container Instances, Microsoft Defender for Cloud, or Machine Learning. For more information, see [Allow trusted services to access a network-restricted registry](allow-access-trusted-services.md).
- **Disable the network-rule bypass for ACR Tasks unless a task requires it**: Prevent ACR Tasks from bypassing the registry's network rules so that automated builds are subject to the same restrictions as other clients. For more information, see [Manage the network bypass policy for tasks](manage-network-bypass-policy-for-tasks.md).
- **Account for regional endpoints when configuring firewall rules**: When you restrict outbound access from clients or use geo-replication, allow the global registry endpoint and each required regional data endpoint so that pulls aren't silently blocked. For more information, see [Configure client firewall rules for Azure Container Registry](container-registry-firewall-rules.md).

## Identity and access management

Azure Container Registry integrates with Microsoft Entra ID for both control-plane and data-plane access. Use managed identities and least-privilege roles instead of shared credentials, and scope access to individual repositories where isolation matters.

- **Authenticate with Microsoft Entra ID instead of shared credentials**: Use Microsoft Entra identities for interactive and programmatic access so that user access is attributable and you can govern it with Conditional Access and role assignments. For more information, see [Authentication options for Azure Container Registry](container-registry-authentication.md).
- **Use managed identities for Azure-hosted workloads**: Assign a managed identity to compute such as Azure Kubernetes Service, Azure Container Instances, or ACR Tasks so that you don't store or rotate registry credentials by hand. For more information, see [Authenticate with a managed identity](container-registry-authentication-managed-identity.md).
- **Assign the least-privilege built-in role for each identity**: Separate control-plane administration from data-plane image access by assigning data-plane roles such as `AcrPull`, `AcrPush`, and `AcrDelete` rather than broad roles like Contributor. For more information, see [Azure Container Registry roles and permissions](container-registry-rbac-built-in-roles-overview.md).
- **Restrict identities to specific repositories with attribute-based access control**: Enable Microsoft Entra attribute-based access control (ABAC) and add repository conditions to roles such as `Container Registry Repository Reader` and `Container Registry Repository Writer` so an identity can reach only named repositories. For more information, see [Microsoft Entra ABAC repository permissions](container-registry-rbac-abac-repository-permissions.md).
- **Create a custom role only when built-in roles are too broad**: Define a custom role that limits the permitted `Microsoft.ContainerRegistry` actions when no built-in role expresses the boundary you need. For more information, see [Create a custom role with Azure Container Registry permissions](container-registry-rbac-custom-roles.md).
- **Use repository-scoped tokens for clients that can't use Microsoft Entra ID**: Issue tokens backed by scope maps to grant a client access to only specific repositories and actions instead of registry-wide credentials. For more information, see [Create a token with repository-scoped permissions](container-registry-token-based-repository-permissions.md).
- **Grant service principals only the roles their automation requires**: For external automation that can't use a managed identity, use a Microsoft Entra service principal scoped to `AcrPull` or `AcrPush` rather than the admin account. For more information, see [Authenticate with a service principal](container-registry-auth-service-principal.md).

## Data protection

Registry content is encrypted at rest by default. Add customer key control where your compliance requirements demand it, and protect image integrity from build through deployment.

- **Encrypt the registry with a customer-managed key when you need control over encryption keys**: Configure a customer-managed key stored in Azure Key Vault and accessed through a managed identity to meet regulatory or organizational key-control requirements. Customer-managed keys require the Premium service tier and must be configured at registry creation. For more information, see [Encrypt registry using a customer-managed key](tutorial-enable-customer-managed-keys.md).
- **Sign and verify images with the Notary Project instead of Docker Content Trust**: Docker Content Trust is deprecated and will be removed from Azure Container Registry on March 31, 2028. Sign images with Notation and verify signatures in your pipelines and deployment platforms to enforce image integrity. For more information, see [Sign container images with Notation and Azure Key Vault](container-registry-tutorial-sign-build-push.md). For the Docker Content Trust deprecation timeline, see [Content trust deprecation](container-registry-content-trust-deprecation.md).
- **Verify signatures against a trusted certificate authority**: Establish trust policies that accept only images signed by an approved certificate authority so that unsigned or untrusted images are rejected before deployment. For more information, see [Sign images with a certificate issued by a trusted CA](container-registry-tutorial-sign-trusted-ca.md).
- **Lock production images against deletion and mutation**: Set the `deleteEnabled` and `writeEnabled` attributes to false on production tags, manifests, or repositories so that critical images can't be accidentally overwritten or removed. For more information, see [Lock a container image in an Azure container registry](container-registry-image-lock.md).
- **Disable artifact export for network-restricted registries**: Set the registry export policy to disabled to prevent users from exporting images in a private, network-restricted Premium registry to another registry, reducing the risk of artifact exfiltration. For more information, see [Disable export of artifacts from a container registry](data-loss-prevention.md).

## Logging and monitoring

Azure Monitor resource logs record registry authentication and repository operations. Capture them centrally, and add vulnerability scanning so that pushed images are continuously assessed.

- **Send registry resource logs to a Log Analytics workspace**: Create a diagnostic setting that routes registry resource logs and metrics to Azure Monitor so that authentication and repository activity is retained for detection and investigation. For more information, see [Monitor Azure Container Registry](monitor-container-registry.md).
- **Audit authentication activity with the ContainerRegistryLoginEvents table**: Review the `ContainerRegistryLoginEvents` table to investigate authentication status, the identity used, and the source IP address of sign-in attempts. For more information, see [Azure Container Registry monitoring data reference](monitor-container-registry-reference.md).
- **Audit image operations with the ContainerRegistryRepositoryEvents table**: Review the `ContainerRegistryRepositoryEvents` table to audit push, pull, untag, and delete operations against your repositories. For more information, see [Azure Container Registry monitoring data reference](monitor-container-registry-reference.md).
- **Scan pushed images for vulnerabilities with Microsoft Defender for Cloud**: Enable Microsoft Defender for Containers to scan images in the registry and surface vulnerability findings for remediation before the images are deployed. For more information, see [Scan registry images with Microsoft Defender for Cloud](scan-images-defender.md).

## Compliance and governance

Enforce and audit secure registry configuration at scale with Azure Policy, and protect the registry resource from accidental change.

- **Enforce secure configuration with Azure Policy definitions for Container Registry**: Assign built-in policy definitions to audit or enforce controls such as disabled anonymous pull, disabled public network access, and customer-managed key encryption across your registries. For more information, see [Audit compliance of Azure Container Registry using Azure Policy](container-registry-azure-policy.md).
- **Map registry controls to regulatory compliance requirements**: Use the Azure Policy Regulatory Compliance controls for Container Registry to map built-in definitions to the compliance standards your organization must meet. For more information, see [Azure Policy Regulatory Compliance controls for Azure Container Registry](security-controls-policy.md).
- **Apply resource locks to prevent accidental deletion**: Add a CanNotDelete or ReadOnly lock to the registry resource so that a misconfiguration or accidental command can't delete the registry and its images. For more information, see [Lock your resources to protect your infrastructure](/azure/azure-resource-manager/management/lock-resources).
- **Tag registries to support governance and incident response**: Apply tags that record owner, environment, and data classification so that you can inventory, attribute, and prioritize registries during audits and investigations. For more information, see [Use tags to organize your Azure resources](/azure/azure-resource-manager/management/tag-resources).

## Backup and recovery

Geo-replication and soft delete provide registry availability and recoverability. Plan them together, and be aware of how they interact.

- **Geo-replicate the registry for regional resilience**: Replicate a single logical registry to multiple regions so that a regional outage doesn't cut off image pulls and pushes. Geo-replication requires the Premium service tier. For more information, see [Geo-replication in Azure Container Registry](container-registry-geo-replication.md).
- **Enable soft delete to recover deleted artifacts**: Turn on the soft delete policy to retain deleted tags and manifests for a configurable recovery window of 1 to 90 days. Soft delete is currently in preview and doesn't support geo-replicated registries, so don't rely on it as your only recovery control for a replicated registry. For more information, see [Enable soft delete policy in Azure Container Registry](container-registry-soft-delete-policy.md).
- **Protect intermediary storage and identities during image transfer**: Treat import, export, and transfer operations as security-sensitive recovery paths by restricting the identities that run them and protecting the storage that holds transferred artifacts, especially across tenants or air-gapped networks. For more information, see [Transfer artifacts with ACR Transfer](container-registry-transfer-cli.md).

## Related content

- [Security overview for Azure Container Registry](container-registry-security-overview.md)
- [Best practices for Azure Container Registry](container-registry-best-practices.md)
- [Zero Trust guidance center](/security/zero-trust/zero-trust-overview)
