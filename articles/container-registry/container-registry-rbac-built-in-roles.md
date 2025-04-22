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

Azure Container Registry offers a set of [built-in Azure roles](/azure/role-based-access-control/built-in-roles) that provide various permissions to an Azure Container Registry resource. Using [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/), you can assign a built-in role to users, managed identities, or service principals to grant permissions defined within the role. You can also define and assign [custom roles](container-registry-rbac-custom-roles.md) with fine-grained permissions tailored to your specific needs.

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

## Built-in roles

The following built-in roles are available for Azure Container Registry. Each role has a specific set of permissions (actions and data actions) that determine what operations can be performed on the registry and its resources. The permissions are divided into two categories:
- **Control plane permissions**: operations that manage and configure of Azure Container Registry resources
- **Data plane permissions**: operations that push, pull, modify, or delete images and artifacts stored within an Azure Container Registry resource

### Privileged built-in roles

The following built-in roles are privileged roles. Assign these roles only to trusted identities, as they provide access to a wide range of registry resources and permissions.

#### Owner
- **Use case**: Assign to administrators who need complete control over the registry resource, including the ability to assign roles to other identities and perform role assignments for the registry.
- **Permissions**:
  - **Description**: Full access to all registry control plane operations and all data plane operations, including role assignment permissions.
  - **Control plane permissions**:
    - Create, update, view, list and delete registry resources (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - Manage [role assignments for registry resources](container-registry-rbac-built-in-roles.md)
    - Manage [geo-replications](container-registry-geo-replication.md)
    - Manage [connected registries](intro-connected-registry.md)
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr.md#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr.md#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
    - Manage [auto-purge on ACR Tasks](container-registry-auto-purge.md)
    - Configure [artifact cache rules and credential sets](artifact-cache-overview.md)
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
    - Update registry resource configuration
      - Configure the [registry's managed identity](/cli/azure/acr/identity.md) (registry's managed system-assigned managed identity or user-assigned managed identity)
      - Configure network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [VNET service endpoints](container-registry-vnet.md))
      - Configure [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - Configure authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [tokens and scope maps](container-registry-repository-scoped-permissions.md), and [Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - Configure registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
    - View registry usage (storage usage)
  - **Data plane permissions**:
    - Push and pull images and artifacts within repositories in the registry
    - Create, view, list, and delete OCI referrer artifacts
    - Manage image and artifact metadata such as tags (creating, reading, listing, retagging, and untagging tags)
    - View and list repositories (image names) in the registry
    - Configure [artifact streaming](container-registry-artifact-streaming.md) for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)
    - Manage [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md) (list and read quarantined artifacts, modify artifact quarantine status)
    - Manage [soft deleted artifacts](container-registry-soft-delete-policy.md) (list and restore soft deleted artifacts)
    - Configure policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - Sign container images with [Docker Content Trust (DCT)](container-registry-content-trust.md)

#### Contributor
- **Use case**: Assign to identities that need to manage registry resources but do not require role assignment permissions.
- **Permissions**:
  - **Description**: Full access to all registry control plane operations and all data plane operations, except role assignment permissions.
  - **Control plane permissions**:
    - Same as Owner, except for managing [role assignments for registry resources](container-registry-rbac-built-in-roles.md).
    - Read and list (but not manage) [role assignments for registry resources](container-registry-rbac-built-in-roles.md).
  - **Data plane permissions**:
    - Same as Owner

#### Reader
- **Use case**: Assign to identities who only need to view registry resources, view registry configuration, list repositories, pull images, or read image metadata.
- **Permissions**:
  - **Description**: Grants the same visibility as Owner and Contributor, but restricted to read-only operations. Does not permit create, update, or delete actions on control or data plane resources.
  - **Control plane permissions**:
    - View and list registry resources (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - Read and list (but not manage) [role assignments for registry resources](container-registry-rbac-built-in-roles.md)
    - View and list [geo-replications](container-registry-geo-replication.md)
    - View and list [connected registries](intro-connected-registry.md)
    - View and list [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
    - View and list [artifact cache rules and credential sets](artifact-cache-overview.md)
    - View and list [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
    - View registry resource configuration
      - View and list the [registry's managed identity](/cli/azure/acr/identity.md) (registry's managed system-assigned managed identity or user-assigned managed identity)
      - View and list network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [VNET service endpoints](container-registry-vnet.md))
      - View and list [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - View and list authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [tokens and scope maps](container-registry-repository-scoped-permissions.md), and [Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - View and list registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
    - View and list registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
    - View registry usage (storage usage)
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts
    - View and list image and artifact metadata such as tags
    - View and list repositories (image names) in the registry
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)
    - View and list (but not manage) [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - View and list (but not manage) [soft deleted artifacts](container-registry-soft-delete-policy.md)
    - View policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))

### Control plane built-in roles

The following built-in roles are control plane roles. Assign these roles to identities that need to manage registry resources, but do not require data plane permissions.

#### Container Registry Contributor and Data Access Configuration Administrator
- **Use case**: Assign to identities that need to manage registry resources and data access settings, but do not require any other control plane or data plane permissions.
- **Permissions**:
  - **Description**: Provides permissions to manage registry resources and data access settings. This role is useful for managing registry resources and auditing, compliance, and monitoring purposes.
  - **Control plane permissions**:
    - Create, update, view, list and delete registry resources (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - Manage [geo-replications](container-registry-geo-replication.md)
    - Manage [connected registries](intro-connected-registry.md)
    - Update registry resource configuration
      - Configure the [registry's managed identity](/cli/azure/acr/identity.md) (registry's managed system-assigned managed identity or user-assigned managed identity)
      <!-- Need to validate if this role has permissions to manage a registry's managed identity, both system-assigned and user-assigned. -->
      - Configure network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [VNET service endpoints](container-registry-vnet.md))
      - Configure [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - Configure authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [tokens and scope maps](container-registry-repository-scoped-permissions.md), and [Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - Configure registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
  <!-- This role is missing Event Grid management permissions, and permissions to list registry usage (storage usage). -->

#### Container Registry Configuration Reader and Data Access Configuration Reader
- **Use case**: Assign to identities that need to view registry configuration and data access settings, but do not require any other control plane or data plane permissions.
- **Permissions**:
  - **Description**: Read-only access to registry configuration and data access settings. This role is useful for viewing registry resources and auditing, compliance, and monitoring purposes.
  - **Control plane permissions**:
    - View and list registry resources (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list [geo-replications](container-registry-geo-replication.md)
    - View and list [connected registries](intro-connected-registry.md)
    - View registry resource configuration
      - View and list the [registry's managed identity](/cli/azure/acr/identity.md) (registry's managed system-assigned managed identity or user-assigned managed identity)
      <!-- Need to validate if this role has permissions to view a registry's managed identity, both system-assigned and user-assigned. -->
      - View and list network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [VNET service endpoints](container-registry-vnet.md))
      - View and list [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - View and list authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [tokens and scope maps](container-registry-repository-scoped-permissions.md), and [Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - View and list registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
    - View and list registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, and [webhooks for registries and geo-replications](container-registry-webhook.md))
    <!-- This role is missing Event Grid read permissions, and permissions to list registry usage (storage usage). -->
  - **Data plane permissions**:
    - None

#### Container Registry Tasks Contributor
- **Use case**: Assign to identities that need to manage ACR tasks and task resources, but do not require any other control plane or data plane permissions.
- **Permissions**:
  - **Description**: Manage ACR tasks and task resources.
  - **Control plane permissions**:
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr.md#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr.md#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
    <!-- Need to validate if this role has permissions to manage task identities. -->
    - Manage [auto-purge on ACR Tasks](container-registry-auto-purge.md)
  - **Data plane permissions**:
    - None

#### Container Registry Transfer Pipeline Contributor
- **Use case**: Assign to identities that need to manage ACR transfer pipelines and task resources, but do not require any other control plane or data plane permissions.
- **Permissions**:
  - **Description**: Manage ACR transfer pipelines, import pipelines, export pipelines, and import/export pipeline runs.
  - **Control plane permissions**:
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
  - **Data plane permissions**:
    - None

### Data plane built-in roles

The following built-in roles are data plane roles. Assign these roles to identities that need to perform data plane operations to interact with images and artifacts stored within a registry, but do not require control plane permissions to manage registry resources.

#### Container Registry Data Importer and Data Reader
- **Use case**: Assign to identities that need to import images from other registries and read images and artifacts in the registry.
- **Permissions**:
  - **Description**: Import images from other registries and read images and artifacts in the registry.
  - **Control plane permissions**:
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts
    - View and list image and artifact metadata such as tags
    - View and list repositories (image names) in the registry
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)

#### AcrPush
- **Use case**: Assign to identities such as CI/CD pipelines, automated processes, or developers that need to push and pull images and tags.
- **Permissions**:
  - **Description**: Perform all data plane operations.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Push and pull images and artifacts within repositories in the registry
    - Create, view, list, and delete OCI referrer artifacts
    - Manage image and artifact metadata such as tags (creating, reading, listing, retagging, and untagging tags)
    - View and list repositories (image names) in the registry
    - Configure [artifact streaming](container-registry-artifact-streaming.md) for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)

#### AcrPull
- **Use case**: Assign to identities such as container host nodes, container orchestrators, vulnerability scanners, or developers that need to pull images and read tags, without needing to push images.
- **Permissions**:
  - **Description**: Read artifacts in the registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts
    - View and list image and artifact metadata such as tags
    - View and list repositories (image names) in the registry
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)

#### AcrDelete
- **Use case**: Assign to identities or services responsible for managing image lifecycle and cleanup.
- **Permissions**:
  - **Description**: Delete artifacts in the registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Delete images, artifacts, digests, and tags

#### AcrImageSigner
- **Use case**: Assign to automated processes or services that sign images as part of a trusted supply chain, such as CI/CD pipelines.
- **Permissions**:
  - **Description**: Sign images in the registry with [Docker Content Trust (DCT)](container-registry-content-trust.md). Take note that Docker Content Trust is being [deprecated](container-registry-content-trust-deprecation.md).
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Sign container images with [Docker Content Trust (DCT)](container-registry-content-trust.md)

#### AcrQuarantineWriter
- **Use case**: Assign to automated processes or services that manage quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**:
  - **Description**: Manage quarantined images in the registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Manage [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md) (list and read quarantined artifacts, modify artifact quarantine status)

#### AcrQuarantineReader
- **Use case**: Assign to automated processes or services that read quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**:
  - **Description**: Read quarantined images in the registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - View and list (but not manage) [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
