---
title: Azure Container Registry roles and permissions
description: Use Azure RBAC, ABAC, and IAM to provide fine-grained permissions to resources in an Azure Container Registry.
ms.topic: conceptual
author: rayoef, johnsonshi
ms.author: rayoflores, johsh
ms.date: 04/16/2025
ms.service: azure-container-registry
---

# Azure Container Registry roles and permissions

Azure Container Registry (ACR) offers a set of [built-in roles](/azure/role-based-access-control/built-in-roles) that provide various permissions to an ACR registry resource. Using [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/), you can assign a built-in role to users, managed identities, or service principals to grant permissions defined within the role. You can also define and assign [custom roles](container-registry-rbac-custom-roles.md) with fine-grained permissions tailored to your specific needs.

## Assigning a built-in role

See [Steps to add a role assignment](/azure/role-based-access-control/role-assignments-steps) for high-level steps to assign a role assignment to an existing [individual user identity](container-registry-authentication.md#individual-login-with-azure-ad), group, [managed identity for Azure resources](container-registry-authentication-managed-identity.md) (including [AKS cluster managed identity](/azure/aks/cluster-container-registry-integration?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json)), and [service principal](container-registry-authentication.md#service-principal) (including [AKS cluster service principal](authenticate-aks-cross-tenant.md)).

Role assignments can be made using the [Azure portal](/azure/role-based-access-control/role-assignments-portal), [Azure CLI](/azure/role-based-access-control/role-assignments-cli), or or [Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell).

## Authenticating after role assignment

After assigning a built-in role to an identity, refer to the following resources to authenticate and access the registry based on the identity type:
- [individual user identity](container-registry-authentication.md#individual-login-with-azure-ad)
- [managed identity for Azure resources](container-registry-authentication-managed-identity.md)
  - [AKS cluster managed identity](/azure/aks/cluster-container-registry-integration?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json)
- [service principal](container-registry-authentication.md#service-principal)
  - [AKS cluster service principal](authenticate-aks-cross-tenant.md)

## Recommended built-in role by scenario

Apply the principle of least privilege by assigning only the permissions necessary for a user or service to perform its intended function. Below are common usage scenarios and the corresponding recommended built-in role.

### CI/CD solutions
- **Recommended role**: `AcrPush`
- **Reason**: Enables image push (`docker push`) to a specific registry, without granting broader registry or Azure Resource Manager permissions.

### Container host nodes
- **Recommended role**: `AcrPull`
- **Reason**: Enables image pull (`docker pull`) from a specific registry, without granting broader registry or Azure Resource Manager permissions.

### Vulnerability scanners
- **Recommended role**: `Container Registry Configuration Reader and Data Access Configuration Reader` and `AcrPull`
- **Reason**: Allows viewing and listing registries (from the `Container Registry Configuration Reader and Data Access Configuration Reader` role) and pulling images (from the `AcrPull` role) to scan and analyze images for vulnerabilities.

### Visual Studio Code Docker extension
- **Recommended roles**: `Contributor Registry Contributor and Data Access Configuration Administrator`, `Container Registry Tasks Contributor`, and `AcrPush`
- **Reason**: Grants capabilities to browse registries, pull and push images, and build images using `az acr build`, supporting common developer workflows in Visual Studio Code.

## Overview of built-in roles

This overview walks through the key Azure Container Registry built-in roles and their recommended use across common scenarios.
For a complete list of ACR built-in roles with detailed permission breakdowns, see the [Azure Container Registry roles directory reference](container-registry-rbac-built-in-roles-directory-reference.md).
If the built-in roles do not meet your needs, you can create custom roles with the permissions you require. For more information on creating custom roles, see [Azure Container Registry custom roles](container-registry-rbac-custom-roles.md).

### Owner
- **Use case**: Assign to administrators who need complete control over the registry resource, including the ability to assign roles to other identities and perform role assignments for the registry.
- **Permissions**: Full access to all registry control plane operations and data plane operations, including role assignment permissions.

### Contributor
- **Use case**: Assign to identities that need to manage registry resources, but do not require role assignment permissions.
- **Permissions**: Full access to all registry control plane operations and all data plane operations, except role assignment permissions.

### Reader
- **Use case**: Assign to identities who only need to view and list registry resources and registry configuration.
- **Permissions**: Grants the same visibility as Owner and Contributor, but restricted to read-only operations. Does not permit create, update, or delete actions on registry resources.

### Container Registry Contributor and Data Access Configuration Administrator
- **Use case**: Ideal for registry administrators, CI/CD pipelines, or automated processes that need to create and configure registries, set up registry authentication mechanisms, manage registry network access, and manage registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to create, configure, and manage registry resources, including authentication settings, tokens, private endpoints, network access, and registry policies. Does not include data plane operations (e.g., image push/pull) or role assignment capabilities.

### Container Registry Configuration Reader and Data Access Configuration Reader
- **Use case**: Ideal for auditors, monitoring systems, and vulnerability scanners that only need to view registries, audit registry authentication mechanisms, audit registry network access configurations, and view registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to view and list registry resources, including authentication settings, tokens, private endpoints, network access, and registry policies. Does not include data plane operations (e.g., image push/pull) or role assignment capabilities.

### Container Registry Tasks Contributor
- **Use case**: Assign to identities—such as CI/CD pipelines or automation tools—that need to manage ACR tasks and task-related resources without access to other registry operations or image data.
- **Permissions**: Grants control plane access to manage [ACR tasks](container-registry-tasks-overview.md), including task definitions, runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr.md#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr.md#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md). Does not include data plane permissions or access to registry configuration outside of tasks.

### Container Registry Transfer Pipeline Contributor
- **Use case**: Assign to CI/CD pipelines or automation processes that need to manage [ACR transfer pipelines](container-registry-transfer-cli.md) for moving artifacts across network, tenant, or air gap boundaries. This role is ideal when transfers must flow through an intermediary Azure Storage account to bridge isolated environments.
- **Permissions**: Grants control plane access to configure and operate [ACR import/export transfer pipelines](container-registry-transfer-cli.md) using intermediary storage accounts, enabling secure artifact transfer between disconnected or segmented environments. Does not include data plane permissions, broader registry access, or permissions to manage other Azure resource types such as storage accounts or key vaults.

### Container Registry Data Importer and Data Reader
- **Use case**: Assign to identities—such as CI/CD pipelines—that need to [import images from other registries with `az acr import`](container-registry-import-images.md). The role also enables reading images and artifacts in a registry to validate the success of the import operation.
- **Permissions**: Grants control plane access to trigger [image imports using `az acr import`](container-registry-import-images.md), and data plane access to pull images and artifacts, view repository contents, referrers, tags, and artifact streaming configurations. Does not allow pushing or modifying any content in the registry.

### AcrPush
- **Use case**: Assign to CI/CD pipelines, automation tools, or developers that need to push and pull container images, manage tags, and work with artifacts—without needing control over registry configuration or settings.
- **Permissions**: Grants data plane access to push and pull images and artifacts, manage tags, work with OCI referrers, and configure artifact streaming for repositories and images. Does not include any control plane permissions.

### AcrPull
- **Use case**: Assign to container host nodes, orchestrators, vulnerability scanners, or developers that only need to pull images and read repository metadata—without permissions to push or modify content.
- **Permissions**: Grants data plane read-only access to pull images and artifacts, view tags, repositories, OCI referrers, and artifact streaming configurations. Does not include any control plane or write permissions.

### AcrDelete
- **Use case**: Assign to identities or services responsible for managing image lifecycle and cleanup.
- **Permissions**: Delete artifacts and tags in the registry.

### AcrImageSigner
- **Use case**: Assign to automated processes or services that sign images as part of a trusted supply chain, such as CI/CD pipelines.
- **Permissions**:
  - **Description**: Sign images in the registry with [Docker Content Trust (DCT)](container-registry-content-trust.md). Take note that Docker Content Trust is being [deprecated](container-registry-content-trust-deprecation.md).

### AcrQuarantineWriter
- **Use case**: Assign to automated processes or services that manage quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: Manage quarantined images in a registry.

### AcrQuarantineReader
- **Use case**: Assign to automated processes or services that list, read, and pull quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: List, read, and pull quarantined images in a registry.

## Next steps

* For a detailed reference of every ACR built-in role, including the permissions granted by each role, see the [Azure Container Registry roles directory reference](container-registry-rbac-built-in-roles-directory-reference.md).
* For more information on creating custom roles that meet your specific needs and requirements, see [Azure Container Registry custom roles](container-registry-rbac-custom-roles.md).
