---
title: Azure Container Registry roles directory reference
description: Directory reference of Azure Container Registry built-in roles and their permissions.
ms.topic: conceptual
author: rayoef, johnsonshi
ms.author: rayoflores, johsh
ms.date: 04/21/2025
ms.service: azure-container-registry
---

# Azure Container Registry roles directory reference

This directory provides a comprehensive reference of all built-in roles available for Azure Container Registry. Each role is documented with its included control plane and data plane permissions. The directory reference is structured by role category—privileged roles, control plane roles, and data plane roles—and is designed for users who need detailed knowledge of ACR permissions for auditing, security, or custom role design.
For a high-level overview of these built-in roles—including assignment steps, authentication methods, and recommended roles for common scenarios—see [Azure Container Registry RBAC built-in roles](container-registry-rbac-built-in-roles.md).

## Built-in roles reference

Each built-in role is defined by a set of permissions (actions and data actions) that control what operations can be performed on the registry and its resources. These permissions fall into two categories:
- Control plane permissions: Manage and configure Azure Container Registry resources.
- Data plane permissions: Perform operations such as pushing, pulling, modifying, or deleting images and artifacts within the registry.

### Privileged built-in roles

The following built-in roles are privileged roles. Assign these roles only to trusted identities, as they provide access to a wide range of registry resources and permissions.

#### Owner
- **Use case**: Assign to administrators who need complete control over the registry resource, including the ability to assign roles to other identities and perform role assignments for the registry.
- **Permissions**: Full access to all registry control plane operations and data plane operations, including role assignment permissions.
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
- **Use case**: Assign to identities that need to manage registry resources, but do not require role assignment permissions.
- **Permissions**: Full access to all registry control plane operations and all data plane operations, except role assignment permissions.
  - **Control plane permissions**:
    - Same as Owner, except for managing [role assignments for registry resources](container-registry-rbac-built-in-roles.md).
    - Read and list (but not manage) [role assignments for registry resources](container-registry-rbac-built-in-roles.md).
  - **Data plane permissions**:
    - Same as Owner

#### Reader
- **Use case**: Assign to identities who only need to view and list registry resources and registry configuration.
- **Permissions**: Grants the same visibility as Owner and Contributor, but restricted to read-only operations. Does not permit create, update, or delete actions on registry resources.
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
- **Use case**: Ideal for registry administrators, CI/CD pipelines, or automated processes that need to create and configure registries, set up registry authentication mechanisms, manage registry network access, and manage registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to create, configure, and manage registry resources, including authentication settings, tokens, private endpoints, network access, and registry policies. Does not include data plane operations (e.g., image push/pull) or role assignment capabilities.
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
- **Use case**: Ideal for auditors, monitoring systems, and vulnerability scanners that only need to view registries, audit registry authentication mechanisms, audit registry network access configurations, and view registry policies—without needing permissions to push/pull images or assign roles.
- **Permissions**: Grants control plane access to view and list registry resources, including authentication settings, tokens, private endpoints, network access, and registry policies. Does not include data plane operations (e.g., image push/pull) or role assignment capabilities.
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
- **Use case**: Assign to identities—such as CI/CD pipelines or automation tools—that need to manage ACR tasks and task-related resources without access to other registry operations or image data.
- **Permissions**: Grants control plane access to manage [ACR tasks](container-registry-tasks-overview.md), including task definitions, runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr.md#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr.md#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md). Does not include data plane permissions or access to registry configuration outside of tasks.
  <!-- Need to validate if this role has permissions to manage task identities. -->
  - **Control plane permissions**:
    - Manage [ACR tasks](container-registry-tasks-overview.md), task runs, [task agent pools](tasks-agent-pools.md), quick tasks ([quick builds with `az acr build`](/cli/azure/acr.md#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr.md#az-acr-run)), [task logs](container-registry-tasks-logs.md), and [task identities](container-registry-tasks-authentication-managed-identity.md)
    <!-- Need to validate if this role has permissions to manage task identities. -->
    - Manage [auto-purge on ACR Tasks](container-registry-auto-purge.md)
  - **Data plane permissions**:
    - None

#### Container Registry Transfer Pipeline Contributor
- **Use case**: Assign to CI/CD pipelines or automation processes that need to manage [ACR transfer pipelines](container-registry-transfer-cli.md) for moving artifacts across network, tenant, or air gap boundaries. This role is ideal when transfers must flow through an intermediary Azure Storage account to bridge isolated environments.
- **Permissions**: Grants control plane access to configure and operate [ACR import/export transfer pipelines](container-registry-transfer-cli.md) using intermediary storage accounts, enabling secure artifact transfer between disconnected or segmented environments. Does not include data plane permissions, broader registry access, or permissions to manage other Azure resource types such as storage accounts or key vaults.
  - **Control plane permissions**:
    - Manage [ACR transfer pipelines for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries](container-registry-transfer-cli.md) (import pipelines, export pipelines, and import/export pipeline runs)
  - **Data plane permissions**:
    - None

#### Container Registry Data Importer and Data Reader
- **Use case**: Assign to identities—such as CI/CD pipelines—that need to [import images from other registries with `az acr import`](container-registry-import-images.md). The role also enables reading images and artifacts in a registry to validate the success of the import operation.
- **Permissions**: Grants control plane access to trigger [image imports using `az acr import`](container-registry-import-images.md), and data plane access to pull images and artifacts, view repository contents, referrers, tags, and artifact streaming configurations. Does not allow pushing or modifying any content in the registry.
  - **Control plane permissions**:
    - Trigger [ACR image imports with `az acr import`](container-registry-import-images.md)
  - **Data plane permissions**:
    - Pull images and artifacts within repositories in the registry
    - View and list OCI referrer artifacts
    - View and list image and artifact metadata such as tags
    - View and list repositories (image names) in the registry
    - View [artifact streaming](container-registry-artifact-streaming.md) configuration for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)

### Data plane built-in roles

The following built-in roles are data plane roles. Assign these roles to identities that need to perform data plane operations to interact with images and artifacts stored within a registry, but do not require control plane permissions to manage registry resources.

#### AcrPush
- **Use case**: Assign to CI/CD pipelines, automation tools, or developers that need to push and pull container images, manage tags, and work with artifacts—without needing control over registry configuration or settings.
- **Permissions**: Grants data plane access to push and pull images and artifacts, manage tags, work with OCI referrers, and configure artifact streaming for repositories and images. Does not include any control plane permissions.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Push and pull images and artifacts within repositories in the registry
    - Create, view, list, and delete OCI referrer artifacts
    - Manage image and artifact metadata such as tags (creating, reading, listing, retagging, and untagging tags)
    - View and list repositories (image names) in the registry
    - Configure [artifact streaming](container-registry-artifact-streaming.md) for repositories and images (such as setting repository policies for automatic artifact streaming conversion, and enabling/disabling artifact streaming conversion for specific images)

#### AcrPull
- **Use case**: Assign to container host nodes, orchestrators, vulnerability scanners, or developers that only need to pull images and read repository metadata—without permissions to push or modify content.
- **Permissions**: Grants data plane read-only access to pull images and artifacts, view tags, repositories, OCI referrers, and artifact streaming configurations. Does not include any control plane or write permissions.
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
- **Permissions**: Delete artifacts and tags in the registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Delete images, artifacts, digests, and tags

#### AcrImageSigner
- **Use case**: Assign to automated processes or services that sign images as part of a trusted supply chain, such as CI/CD pipelines.
- **Permissions**: Sign images in the registry with [Docker Content Trust (DCT)](container-registry-content-trust.md). Take note that Docker Content Trust is being [deprecated](container-registry-content-trust-deprecation.md).
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Sign container images with [Docker Content Trust (DCT)](container-registry-content-trust.md)

#### AcrQuarantineWriter
- **Use case**: Assign to automated processes or services that manage quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: Manage quarantined images in a registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - Manage [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md) (list and read quarantined artifacts, modify artifact quarantine status)

#### AcrQuarantineReader
- **Use case**: Assign to automated processes or services that list, read, and pull quarantined images, such as CI/CD pipelines and vulnerability scanners.
- **Permissions**: List, read, and pull quarantined images in a registry.
  - **Control plane permissions**:
    - None
  - **Data plane permissions**:
    - View and list (but not manage) [quarantined artifacts](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)

## Next steps

* For an overview of ACR built-in roles, see [Azure Container Registry built-in roles](container-registry-rbac-built-in-roles.md).
* For more information on creating custom roles that meet your specific needs and requirements, see [Azure Container Registry custom roles](container-registry-rbac-custom-roles.md).
