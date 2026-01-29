---
title: Azure Container Registry roles directory reference
description: Directory reference of Azure Container Registry built-in roles and their permissions.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.date: 04/24/2025
ms.service: azure-container-registry
# Customer intent: As a cloud administrator, I want to reference the built-in roles and permissions for Azure Container Registry, so that I can effectively manage identity access and security within the registry for different user scenarios.
---

# Azure Container Registry roles directory reference

This directory provides a comprehensive reference of all built-in roles available for Azure Container Registry (ACR).
This document is designed for expert users who need detailed knowledge of ACR permissions in built-in roles for identity management, auditing, security, or custom role design.
Each ACR built-in role is documented here with its included control plane and data plane permissions.

The following built-in role types are available:
- [Control plane roles](#control-plane-roles)
- [Data plane roles](#data-plane-roles)
- [Privileged roles](#privileged-roles)

For a high-level overview of these built-in roles—including supported role assignment identity types, steps to perform a role assignment, and recommended roles for common scenarios—see [Azure Container Registry built-in roles](container-registry-rbac-built-in-roles-overview.md).

> [!NOTE]
> The applicable built-in roles and role behavior depends on the registry's "Role assignment permissions mode." This is visible in the "Properties" blade in the Azure portal:
> - **RBAC Registry + ABAC Repository Permissions**: Supports standard RBAC role assignments with optional Microsoft Entra ABAC conditions to scope assignments to specific repositories.
> - **RBAC Registry Permissions**: Supports only standard RBAC assignments without ABAC conditions.
>
> For details on Microsoft Entra ABAC and ABAC-enabled roles, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).

## Built-in roles reference

Each built-in role includes a set of permissions (actions and data actions) that control what operations can be performed on the registry. These permissions fall into two categories:
- Control plane permissions: Create, manage, delete, and configure ACR registries, registry-wide configurations, and registry-wide policies.
- Data plane permissions: Perform operations that read, modify, or delete data within a registry, such as pushing, pulling, modifying, or deleting images, artifacts, and tags within the registry. Also includes operations that modify repository-specific configurations and repository-specific policies.

### Control plane roles

The following built-in roles are control plane roles. Assign these roles to identities that need to manage registries, but don't require data plane permissions.

The applicable roles and role behavior depends on the registry's "Role assignment permissions mode." This is visible in the "Properties" blade in the Azure portal. For more information on Entra ABAC, see [Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).

#### [Registries configured with "RBAC Registry + ABAC Repository Permissions"](#tab/registries-configured-with-rbac-registry-abac-repository-permissions)

##### Container Registry Contributor and Data Access Configuration Administrator
- **Use case**: Ideal for registry administrators, CI/CD pipelines, or automated processes that need to create and configure registries, set up registry authentication mechanisms, manage registry network access, and manage registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to create, configure, and manage registries and registry configurations, including authentication settings, tokens, private endpoints, network access, and registry policies. Doesn't include data plane operations (for example, image push/pull) or role assignment capabilities.
  - **Control plane permissions**:
    - Create, update, view, list, and delete registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list (but not manage) [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - Manage [geo-replications](container-registry-geo-replication.md)
    - Manage [connected registries](intro-connected-registry.md)
    - Update registry configuration
      - Configure the [registry's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage a registry's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Configure network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - Configure [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - Configure authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - Configure registry policies (configure [retention policy](container-registry-retention-policy.md), [registry-wide quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md)).
      - Note: This role does not grant permissions to manage, view, and list quarantined or soft-deleted artifacts, only granting permissions to configure quarantine and soft-delete settings for a registry.
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
  - **Data plane permissions**:
    - None

##### Container Registry Configuration Reader and Data Access Configuration Reader
- **Use case**: Ideal for auditors, monitoring systems, and vulnerability scanners that only need to view registries, audit registry authentication mechanisms, audit registry network access configurations, and view registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to view and list registries and registry configurations, including authentication settings, tokens, private endpoints, network access, and registry policies. Doesn't include data plane operations (for example, image push/pull) or role assignment capabilities.
  - **Control plane permissions**:
    - View and list registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list (but not manage) [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - View and list [geo-replications](container-registry-geo-replication.md)
    - View and list [connected registries](intro-connected-registry.md)
    - View registry configuration
      - View and list both the [registry's system-assigned managed identity and user-assigned managed identity](/cli/azure/acr/identity)
      - View and list network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - View and list [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - View and list authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - View and list registry policies (configure [retention policy](container-registry-retention-policy.md), [registry-wide quarantine enablement status](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - Note: This role does not grant permissions to manage, view, and list quarantined or soft-deleted artifacts, only granting permissions to view quarantine and soft-delete settings for a registry.
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
  - **Data plane permissions**:
    - None

##### Container Registry Tasks Contributor
- **Use case**: Assign to identities—such as CI/CD pipelines or automation tools—that need to manage ACR tasks and task-related resources without access to other registry operations or image data.
- **Permissions**: Grants control plane access to manage [ACR tasks](container-registry-tasks-overview.md), including task definitions, runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md). Doesn't include data plane permissions or access to registry configuration outside of tasks.
  - **Control plane permissions**:
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
      - Grants permissions to configure an [ACR task's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage an ACR task's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Grants permissions to manage [auto-purge on ACR tasks](container-registry-auto-purge.md)
      - **For ABAC-enabled registries, ACR tasks, quick builds, and quick runs don't have default data plane permissions to push, pull, or delete images and tags within repositories. See [effects of enabling ABAC on ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs](container-registry-rbac-abac-repository-permissions.md#effects-of-enabling-abac-on-acr-tasks-quick-tasks-quick-builds-and-quick-runs).**
        - ACR tasks belonging to ABAC-enabled registries must have the `Container Registry Repository Reader/Writer/Contributor` and `Container Registry Repository Catalog Lister` roles assigned to the task identity in order to perform data plane operations.
        - For quick builds and quick runs, the identity (caller) invoking the quick task must have the `Container Registry Repository Reader/Writer/Contributor` and `Container Registry Repository Catalog Lister` roles assigned to it in order to perform data plane operations.
  - **Data plane permissions**:
    - None

##### Container Registry Transfer Pipeline Contributor
- **Use case**: Assign to CI/CD pipelines or automation processes that need to manage [ACR transfer pipelines](container-registry-transfer-cli.md) for moving artifacts across network, tenant, or air gap boundaries. This role is ideal when transfers must flow through an intermediary Azure Storage account to bridge isolated environments.
- **Permissions**: Grants control plane access to configure and operate [ACR import/export transfer pipelines](container-registry-transfer-cli.md) using intermediary storage accounts, enabling secure artifact transfer between disconnected or segmented environments. Doesn't include data plane permissions, broader registry access, or permissions to manage other Azure resource types such as storage accounts or key vaults.
  - **Control plane permissions**:
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
  - **Data plane permissions**:
    - None

##### Container Registry Data Importer and Data Reader
- **Use case**: Assign to identities—such as CI/CD pipelines—that need to [import images from other registries with `az acr import`](container-registry-import-images.md). The role also enables reading images and artifacts in a registry to validate the success of the import operation.
- **Permissions**: Grants control plane access to trigger [image imports using `az acr import`](container-registry-import-images.md), and data plane access to pull images and artifacts, view repository contents, Open Container Initiative (OCI) referrers, tags, and artifact streaming configurations. Doesn't allow pushing or modifying any content in the registry.
  - **Control plane permissions**:
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts within repositories in the registry
    - View and list image and artifact metadata such as tags (listing and reading tags) within repositories in the registry
    - View and list all repositories (image names) in the registry
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories and images (such as viewing repository policies for automatic artifact streaming conversion, and viewing artifact streaming configuration for an image)

#### [Registries configured with "RBAC Registry Permissions"](#tab/registries-configured-with-rbac-registry-permissions)

##### Container Registry Contributor and Data Access Configuration Administrator
- **Use case**: Ideal for registry administrators, CI/CD pipelines, or automated processes that need to create and configure registries, set up registry authentication mechanisms, manage registry network access, and manage registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to create, configure, and manage registries and registry configurations, including authentication settings, tokens, private endpoints, network access, and registry policies. Doesn't include data plane operations (for example, image push/pull) or role assignment capabilities.
  - **Control plane permissions**:
    - Create, update, view, list, and delete registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list (but not manage) [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - Manage [geo-replications](container-registry-geo-replication.md)
    - Manage [connected registries](intro-connected-registry.md)
    - Update registry configuration
      - Configure the [registry's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage a registry's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Configure network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - Configure [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - Configure authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - Configure registry policies (configure [retention policy](container-registry-retention-policy.md), [registry-wide quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - Note: This role does not grant permissions to manage, view, and list quarantined or soft-deleted artifacts, only granting permissions to configure quarantine and soft-delete settings for a registry.
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
  - **Data plane permissions**:
    - None

##### Container Registry Configuration Reader and Data Access Configuration Reader
- **Use case**: Ideal for auditors, monitoring systems, and vulnerability scanners that only need to view registries, audit registry authentication mechanisms, audit registry network access configurations, and view registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to view and list registries and registry configurations, including authentication settings, tokens, private endpoints, network access, and registry policies. Doesn't include data plane operations (for example, image push/pull) or role assignment capabilities.
  - **Control plane permissions**:
    - View and list registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list (but not manage) [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - View and list [geo-replications](container-registry-geo-replication.md)
    - View and list [connected registries](intro-connected-registry.md)
    - View registry configuration
      - View and list both the [registry's system-assigned managed identity and user-assigned managed identity](/cli/azure/acr/identity)
      - View and list network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - View and list [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - View and list authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - View and list registry policies (configure [retention policy](container-registry-retention-policy.md), [registry-wide quarantine enablement status](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - Note: This role does not grant permissions to manage, view, and list quarantined or soft-deleted artifacts, only granting permissions to view quarantine and soft-delete settings for a registry.
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
  - **Data plane permissions**:
    - None

##### Container Registry Tasks Contributor
- **Use case**: Assign to identities—such as CI/CD pipelines or automation tools—that need to manage ACR tasks and task-related resources without access to other registry operations or image data.
- **Permissions**: Grants control plane access to manage [ACR tasks](container-registry-tasks-overview.md), including task definitions, runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md). Doesn't include data plane permissions or access to registry configuration outside of tasks.
  - **Control plane permissions**:
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
      - Configure an [ACR task's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage an ACR task's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Manage [auto-purge on ACR Tasks](container-registry-auto-purge.md)
  - **Data plane permissions**:
    - None

##### Container Registry Transfer Pipeline Contributor
- **Use case**: Assign to CI/CD pipelines or automation processes that need to manage [ACR transfer pipelines](container-registry-transfer-cli.md) for moving artifacts across network, tenant, or air gap boundaries. This role is ideal when transfers must flow through an intermediary Azure Storage account to bridge isolated environments.
- **Permissions**: Grants control plane access to configure and operate [ACR import/export transfer pipelines](container-registry-transfer-cli.md) using intermediary storage accounts, enabling secure artifact transfer between disconnected or segmented environments. Doesn't include data plane permissions, broader registry access, or permissions to manage other Azure resource types such as storage accounts or key vaults.
  - **Control plane permissions**:
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
  - **Data plane permissions**:
    - None

##### Container Registry Data Importer and Data Reader
- **Use case**: Assign to identities—such as CI/CD pipelines—that need to [import images from other registries with `az acr import`](container-registry-import-images.md). The role also enables reading images and artifacts in a registry to validate the success of the import operation.
- **Permissions**: Grants control plane access to trigger [image imports using `az acr import`](container-registry-import-images.md), and data plane access to pull images and artifacts, view repository contents, Open Container Initiative (OCI) referrers, tags, and artifact streaming configurations. Doesn't allow pushing or modifying any content in the registry.
  - **Control plane permissions**:
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts within repositories in the registry
    - View and list image and artifact metadata such as tags (listing and reading tags) within repositories in the registry
    - View and list all repositories (image names) in the registry
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories and images (such as viewing repository policies for automatic artifact streaming conversion, and viewing artifact streaming configuration for an image)

---

### Data plane roles

The following built-in roles are data plane roles. Assign these roles to identities that need to perform data plane operations to interact with images and artifacts stored within a registry, but don't require control plane permissions to manage registries.

The applicable roles and role behavior depends on the registry's "Role assignment permissions mode." This is visible in the "Properties" blade in the Azure portal. For more information on Microsoft Entra ABAC, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).

#### [Registries configured with "RBAC Registry + ABAC Repository Permissions"](#tab/registries-configured-with-rbac-registry-abac-repository-permissions)

##### Container Registry Repository Reader
- **Use case**: Assign to container host nodes, orchestrators, vulnerability scanners, or developers that only need to pull images and read repository metadata—without permissions to push or modify content.
- **Permissions**: Grants data plane read-only access to pull images and artifacts, view tags, repositories, Open Container Initiative (OCI) referrers, and artifact streaming configurations. Doesn't include any control plane or write permissions.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts within repositories in the registry
    - View and list images and artifact metadata such as tags (listing and reading tags) within repositories in the registry
    - **Doesn't grant repository catalog list permissions (permissions to list repositories in the registry)**
    - View policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for a specific image
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as configuring or viewing repository policies for automatic artifact streaming conversion)
    - Does not grant permissions to manage, view, and list [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - Does not grant permissions to manage, view, and list [soft-deleted artifacts](container-registry-soft-delete-policy.md)
  - **ABAC support**: Supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories.

##### Container Registry Repository Writer
- **Use case**: Assign to CI/CD pipelines, automation tools, or developers that need to push and pull container images, manage tags, and work with artifacts—without needing control over registry configuration or settings. Also assign to automated processes or services that sign images as part of a trusted supply chain.
- **Permissions**: Grants data plane access to push and pull images and artifacts, read/manage tags, read/manage OCI referrers, and enable (but not disable) artifact streaming for repositories and images. Doesn't include any control plane permissions.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Push and pull images and artifacts within repositories in the registry
    - Create, view, and list OCI referrer artifacts within repositories in the registry
    - Manage image and artifact metadata such as tags (creating, reading, listing, retagging, and untagging tags) within repositories in the registry
    - **Doesn't grant repository catalog list permissions (permissions to list repositories in the registry)**
    - Manage policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - Enabling (**but not disabling**) [artifact streaming](container-registry-artifact-streaming.md) configuration for a specific image
    - Manage [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as configuring or viewing repository policies for automatic artifact streaming conversion)
    - Does not grant permissions to manage, view, and list [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - Does not grant permissions to manage, view, and list [soft-deleted artifacts](container-registry-soft-delete-policy.md)
  - **ABAC support**: Supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories.

##### Container Registry Repository Contributor
- **Use case**: Assign to identities or services responsible for managing image lifecycle and cleanup.
- **Permissions**: Grants permissions to **read, write, update, and delete** images and artifacts, read/manage/delete tags, read/manage/delete OCI referrers, and enable/disable artifact streaming for repositories and images. Doesn't include any control plane permissions.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Push, pull, and **delete** images and artifacts within repositories in the registry
    - Create, view, list, and **delete** OCI referrer artifacts within repositories in the registry
    - Manage and **delete** image and artifact metadata such as tags (creating, reading, listing, retagging, untagging, and **deleting** tags) within repositories in the registry
    - **Doesn't grant repository catalog list permissions (permissions to list repositories in the registry)**
    - Manage policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - Enabling **and disabling** [artifact streaming](container-registry-artifact-streaming.md) configuration for a specific image
      - Disable artifact streaming support for a specific image by deleting an image's streaming OCI referrer artifact
    - Manage [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as configuring or viewing repository policies for automatic artifact streaming conversion)
    - Does not grant permissions to manage, view, and list [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - Does not grant permissions to manage, view, and list [soft-deleted artifacts](container-registry-soft-delete-policy.md)
  - **ABAC support**: Supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories.

##### Container Registry Repository Catalog Lister
  - **Use case**: Assign to identities or services that need to **list all repositories** in a registry, such as CI/CD pipelines, developers, vulnerability scanners, or registry monitoring and auditing tools.
  - **Permissions**: Grants data plane access to **list all repositories** in the registry. Doesn't include any control plane permissions or permissions to push/pull images.
    - **Control plane permissions**:
      - None
    - **Data plane permissions**:
      - List all repositories (image names) in the registry
      - Grants permissions to invoke the `{loginServerURL}/acr/v1/_catalog` or `{loginServerURL}/v2/_catalog` registry API endpoints to list all repositories in the registry
      - **Doesn't grant permissions to view or list images, artifacts, tags, or OCI referrers within repositories in the registry**
    - **ABAC support**: This role **doesn't support Entra ABAC conditions**. As such, this role assignment will **grant permissions to list all repositories** in the registry.

##### AcrQuarantineWriter
- **Use case**: Assign to automated processes or services that manage quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: Manage quarantined images in a registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Manage [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md) (list and read quarantined artifacts, modify artifact quarantine status)
  - **ABAC support**: Doesn't support Microsoft Entra ABAC conditions.

##### AcrQuarantineReader
- **Use case**: Assign to automated processes or services that list, read, and pull quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: List, read, and pull quarantined images in a registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - View and list (but not manage) [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
  - **ABAC support**: Doesn't support Microsoft Entra ABAC conditions.

#### [Registries configured with "RBAC Registry Permissions"](#tab/registries-configured-with-rbac-registry-permissions)

##### AcrPull
- **Use case**: Assign to container host nodes, orchestrators, vulnerability scanners, or developers that only need to pull images and read repository metadata—without permissions to push or modify content.
- **Permissions**: Grants data plane read-only access to pull images and artifacts, view tags, repositories, OCI referrers, and artifact streaming configurations. Doesn't include any control plane or write permissions.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts within repositories in the registry
    - View and list image and artifact metadata such as tags (listing and reading tags) within repositories in the registry
    - View and list all repositories (image names) in the registry
    - Does not grant permissions to manage or view policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for a specific image
    - Does not grant permissions to manage or view the [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as configuring or viewing repository policies for automatic artifact streaming conversion)
    - Does not grant permissions to manage, view, and list [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - Does not grant permissions to manage, view, and list [soft-deleted artifacts](container-registry-soft-delete-policy.md)

##### AcrPush
- **Use case**: Assign to CI/CD pipelines, automation tools, or developers that need to push and pull container images, manage tags, and work with artifacts—without needing control over registry configuration or settings.
- **Permissions**: Grants data plane access to push and pull images and artifacts, manage tags, work with OCI referrers, and configure artifact streaming for repositories and images. Doesn't include any control plane permissions.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Push and pull images and artifacts within repositories in the registry
    - Create, view, list, and delete OCI referrer artifacts within repositories in the registry
    - Manage image and artifact metadata such as tags (creating, reading, listing, retagging, and untagging tags) within repositories in the registry
    - View and list all repositories (image names) in the registry
    - Does not grant permissions to manage or view policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - Enable (**but not disable**) [artifact streaming](container-registry-artifact-streaming.md) for a specific image
    - Does not grant permissions to manage or view the [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as configuring or viewing repository policies for automatic artifact streaming conversion)
    - Does not grant permissions to manage, view, and list [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - Does not grant permissions to manage, view, and list [soft-deleted artifacts](container-registry-soft-delete-policy.md)

##### AcrDelete
- **Use case**: Assign to identities or services responsible for managing image lifecycle and cleanup.
- **Permissions**: Delete artifacts and tags in the registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Delete images, artifacts, digests, and tags within repositories in the registry
    - Disable artifact streaming support for a specific image by deleting an image's streaming OCI referrer artifact

##### AcrImageSigner
- **Use case**: Assign to automated processes or services that sign images as part of a trusted supply chain, such as CI/CD pipelines.
- **Permissions**: Sign images in the registry with [Docker Content Trust (DCT)](container-registry-content-trust.md). Take note that Docker Content Trust is being [deprecated](container-registry-content-trust-deprecation.md).
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Sign container images with [Docker Content Trust (DCT)](container-registry-content-trust.md)

##### AcrQuarantineWriter
- **Use case**: Assign to automated processes or services that manage quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: Manage quarantined images in a registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Manage [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md) (list and read quarantined artifacts, modify artifact quarantine status)

##### AcrQuarantineReader
- **Use case**: Assign to automated processes or services that list, read, and pull quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: List, read, and pull quarantined images in a registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - View and list (but not manage) [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)

---

### Privileged roles

The following built-in roles are privileged roles.
Assign these roles only to trusted identities, as they provide access to a wide range of resources and permissions over other resource types, not just Azure Container Registry.

Azure recommends using less privileged [control plane roles](#control-plane-roles) or [data plane roles](#data-plane-roles) whenever possible instead of these privileged roles.

The applicable roles and role behavior depends on the registry's "Role assignment permissions mode." This is visible in the "Properties" blade in the Azure portal. For more information on Microsoft Entra ABAC, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).

#### [Registries configured with "RBAC Registry + ABAC Repository Permissions"](#tab/registries-configured-with-rbac-registry-abac-repository-permissions)

##### Owner
- **Use case**: Assign to administrators who need complete control over the registry, including the ability to assign roles to other identities and perform role assignments for the registry.
- **Permissions**: Full access to all registry control plane operations, including [role assignment permissions](container-registry-rbac-built-in-roles-overview.md) and managing [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
  - **Control plane permissions**:
    - Create, update, view, list, and delete registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - Manage [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - Manage [geo-replications](container-registry-geo-replication.md)
    - Manage [connected registries](intro-connected-registry.md)
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
      - Grants permissions to configure an [ACR task's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage an ACR task's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Grants permissions to manage [auto-purge on ACR tasks](container-registry-auto-purge.md)
      - **For ABAC-enabled registries, ACR tasks, quick builds, and quick runs don't have default data plane permissions to push, pull, or delete images and tags within repositories. See [effects of enabling ABAC on ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs](container-registry-rbac-abac-repository-permissions.md#effects-of-enabling-abac-on-acr-tasks-quick-tasks-quick-builds-and-quick-runs).**
        - ACR tasks belonging to ABAC-enabled registries must have the `Container Registry Repository Reader/Writer/Contributor` and `Container Registry Repository Catalog Lister` roles assigned to the task identity in order to perform data plane operations.
        - For quick builds and quick runs, the identity (caller) invoking the quick task must have the `Container Registry Repository Reader/Writer/Contributor` and `Container Registry Repository Catalog Lister` roles assigned to it in order to perform data plane operations.
    - Configure [artifact cache rules and credential sets](artifact-cache-overview.md)
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
    - Update registry configuration
      - Configure the [registry's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage a registry's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Configure network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - Configure [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - Configure authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - Configure registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - Note: This role does not grant permissions to manage, view, and list quarantined or soft-deleted artifacts, only granting permissions to configure quarantine and soft-delete settings for a registry.
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
    - View registry usage (storage usage)
  - **Data plane permissions**:
    - **None - ABAC-enabled registries don't have data plane permissions for the built-in Owner role.**

##### Contributor
- **Use case**: Assign to identities that need to manage registries, but don't require role assignment permissions.
- **Permissions**: Full access to all registry control plane operations, except role assignment permissions.
  - **Control plane permissions**:
    - Same as Owner, except for managing or performing [role assignments for registries](container-registry-rbac-built-in-roles-overview.md). Only permissions for viewing and listing role assignments for a registry are granted.
    - Note: to manage or perform role assignments for registries, the `Role Based Access Control Administrator` role is required. This less privileged role is recommended in lieu of the `Owner` role for managing role assignments.
  - **Data plane permissions**:
    - **None - ABAC-enabled registries don't have data plane permissions for the built-in Contributor role.**

##### Reader
- **Use case**: Assign to identities who only need to view and list registries and registry configuration.
- **Permissions**: Grants the same visibility as Owner and Contributor, but restricted to read-only operations. Doesn't permit create, update, or delete actions on registries.
  - **Control plane permissions**:
    - View and list registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list (but not manage) [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - View and list [geo-replications](container-registry-geo-replication.md)
    - View and list [connected registries](intro-connected-registry.md)
    - View and list [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
    - View and list [artifact cache rules and credential sets](artifact-cache-overview.md)
    - View and list [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
    - View registry configuration
      - View and list both the [registry's system-assigned managed identity and user-assigned managed identity](/cli/azure/acr/identity)
      - View and list network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - View and list [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - View and list authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - View and list registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - Note: This role does not grant permissions to manage, view, and list quarantined or soft-deleted artifacts, only granting permissions to view quarantine and soft-delete settings for a registry.
    - View and list registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
    - View registry usage (storage usage)
  - **Data plane permissions**:
    - **None - ABAC-enabled registries don't have data plane permissions for the built-in Reader role.**

#### [Registries configured with "RBAC Registry Permissions"](#tab/registries-configured-with-rbac-registry-permissions)

##### Owner
- **Use case**: Assign to administrators who need complete control over the registry, including the ability to assign roles to other identities and perform role assignments for the registry.
- **Permissions**: Full access to all registry control plane operations and data plane operations, including [role assignment permissions](container-registry-rbac-built-in-roles-overview.md) and managing [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
  - **Control plane permissions**:
    - Create, update, view, list, and delete registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - Manage [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - Manage [geo-replications](container-registry-geo-replication.md)
    - Manage [connected registries](intro-connected-registry.md)
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
      - Grants permissions to configure an [ACR task's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage an ACR task's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Grants permissions to manage [auto-purge on ACR tasks](container-registry-auto-purge.md)
    - Configure [artifact cache rules and credential sets](artifact-cache-overview.md)
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
    - Update registry configuration
      - Configure the [registry's system-assigned managed identity](/cli/azure/acr/identity). Note: to manage a registry's user-assigned managed identity, the separate `Managed Identity Operator` role is required.
      - Configure network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - Configure [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - Configure authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - Configure registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
    - Configure registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
    - View registry usage (storage usage)
  - **Data plane permissions**:
    - Push and pull images and artifacts within repositories in the registry
    - Create, view, list, and delete OCI referrer artifacts within repositories in the registry
    - Manage image and artifact metadata such as tags (creating, reading, listing, retagging, and untagging tags) within repositories in the registry
    - View and list all repositories (image names) in the registry
    - Manage policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - Manage [artifact streaming](container-registry-artifact-streaming.md) configuration for a specific image
    - Manage [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as configuring or viewing repository policies for automatic artifact streaming conversion)
    - Manage [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md) (list and read quarantined artifacts, modify artifact quarantine status)
    - Manage [soft-deleted artifacts](container-registry-soft-delete-policy.md) (list and restore soft-deleted artifacts)
    - Sign container images with [Docker Content Trust (DCT)](container-registry-content-trust.md)

##### Contributor
- **Use case**: Assign to identities that need to manage registries, but don't require role assignment permissions.
- **Permissions**: Full access to all registry control plane operations and all data plane operations, except role assignment permissions.
  - **Control plane permissions**:
    - Same as Owner, except for managing or performing [role assignments for registries](container-registry-rbac-built-in-roles-overview.md). Only permissions for viewing and listing role assignments for a registry are granted.
    - Note: to manage or perform role assignments for registries, the `Role Based Access Control Administrator` role is required. This less privileged role is recommended in lieu of the `Owner` role for managing role assignments.
  - **Data plane permissions**:
    - Same as Owner - full data plane access.

##### Reader
- **Use case**: Assign to identities who only need to view and list registries and registry configuration.
- **Permissions**: Grants the same visibility as Owner and Contributor, but restricted to read-only operations. Doesn't permit create, update, or delete actions on registries.
  - **Control plane permissions**:
    - View and list registries (including [registry SKUs](container-registry-skus.md) and [availability zones and zone redundancy](zone-redundancy.md))
    - View and list (but not manage) [role assignments for registries](container-registry-rbac-built-in-roles-overview.md)
    - View and list [geo-replications](container-registry-geo-replication.md)
    - View and list [connected registries](intro-connected-registry.md)
    - View and list [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
    - View and list [artifact cache rules and credential sets](artifact-cache-overview.md)
    - View and list [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
    - View registry configuration
      - View and list both the [registry's system-assigned managed identity and user-assigned managed identity](/cli/azure/acr/identity)
      - View and list network access settings ([public network access](container-registry-private-link.md#disable-public-access), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - View and list [private endpoint settings](container-registry-private-link.md) (set up, approve, reject, and list private endpoint connections and private link resources)
      - View and list authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md))
      - View and list registry policies (configure [retention policy](container-registry-retention-policy.md), [quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
    - View and list registry [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
    - View registry usage (storage usage)
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts within repositories in the registry
    - View and list image and artifact metadata such as tags (reading and listing tags) within repositories in the registry
    - View and list all repositories (image names) in the registry
    - View policies for repositories and images (including [repository, image, digest, and tag locking](container-registry-image-lock.md))
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for a specific image
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories (such as viewing repository policies for automatic artifact streaming conversion)
    - View and list (but not manage) [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)
    - View and list (but not manage) [soft-deleted artifacts](container-registry-soft-delete-policy.md)

---

## Next steps

* For a high-level overview of these built-in roles—including supported role assignment identity types, steps to perform a role assignment, and recommended roles for common scenarios—see [Azure Container Registry RBAC built-in roles](container-registry-rbac-built-in-roles-overview.md).
* To perform role assignments with optional Entra ABAC conditions to scope role assignments to specific repositories, see [Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
* For more information on creating custom roles that meet your specific needs and requirements, see [Azure Container Registry custom roles](container-registry-rbac-custom-roles.md).
