---
title: Azure Container Registry Entra permissions and role assignments overview
description: Use Microsoft Entra role-based access control to manage permissions for an Azure Container Registry.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.date: 04/24/2025
ms.service: azure-container-registry
# Customer intent: "As an IT administrator, I want to manage role assignments and permissions for an Azure Container Registry, so that I can ensure appropriate access control and security for containers and related resources."
---

# Azure Container Registry Entra permissions and role assignments overview

Azure Container Registry (ACR) offers a set of [built-in roles](/azure/role-based-access-control/built-in-roles) that provide Microsoft Entra-based permissions management to an ACR registry.
Using [Azure role-based access control (RBAC)](/azure/role-based-access-control/), you can assign a built-in role to users, managed identities, or service principals to grant Microsoft Entra-based permissions defined within the role.
You can also define and assign [custom roles](container-registry-rbac-custom-roles.md) with fine-grained permissions tailored to your specific needs if the built-in roles don't meet your requirements.

## Supported role assignment identity types

ACR roles can be assigned to the following identity types to grant permissions to a registry:
- [Individual user identity](container-registry-authentication.md#individual-login-with-azure-ad)
- [Managed identity for Azure resources](container-registry-authentication-managed-identity.md)
  - [Azure DevOps - Azure Pipelines identity](/azure/devops/pipelines/ecosystems/containers/publish-to-acr)
  - [Azure Kubernetes Service (AKS) node's kubelet identity](/azure/aks/cluster-container-registry-integration) to enable the AKS node to pull images from ACR. ACR supports role assignments for both [AKS-managed kubelet identity and AKS pre-created kubelet identity](/azure/aks/use-managed-identity) for AKS nodes to pull images from ACR.
  - [Azure Container Apps (ACA) identity](/azure/container-apps/managed-identity-image-pull)
  - [Azure Container Instances (ACI) identity](/azure/container-instances/using-azure-container-registry-mi)
  - [Azure Machine Learning (AML) workspace identity](/azure/machine-learning/how-to-identity-based-service-authentication)
  - [AML-attached Kubernetes cluster node kubelet identity](/azure/machine-learning/how-to-attach-kubernetes-to-workspace) to allow the Kubernetes cluster's nodes to pull images from ACR.
  - [AML online endpoint identity](/azure/machine-learning/how-to-access-resources-from-endpoints-managed-identities)
  - [Azure App Service identity](/azure/app-service/tutorial-custom-container)
  - [Azure Web Apps identity](/troubleshoot/azure/azure-container-registry/pull-image-to-web-app-fail)
  - [Azure Batch identity](/azure/batch/batch-docker-container-workloads)
  - [Azure Functions identity](/azure/azure-functions/functions-app-settings#acrusermanagedidentityid)
- [Service principal](container-registry-authentication.md#service-principal)
  - [AKS cluster service principal](authenticate-aks-cross-tenant.md) to enable AKS nodes to pull images from ACR. ACR also supports cross-tenant AKS node to ACR authentication through [cross-tenant service principal role assignments and authentication](authenticate-aks-cross-tenant.md).
  - [ACI service principal](container-registry-auth-aci.md)
  - [Hybrid or on-premises AKS clusters on Azure Stack Hub using service principal](/azure-stack/user/container-registry-deploy-image-aks-cluster)

Take note that [ACR connected registry](intro-connected-registry.md), ACR's on-premises registry offering that differs from cloud-based ACR, doesn't support Azure role assignments and Entra-based permissions management.

## Performing role assignments to grant permissions

See [Steps to add a role assignment](/azure/role-based-access-control/role-assignments-steps) for information on how to assign a role to an identity.
Role assignments can be made using:
- [Azure portal](/azure/role-based-access-control/role-assignments-portal)
- [Azure CLI](/azure/role-based-access-control/role-assignments-cli)
- [Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell)

To perform role assignments, you must either have the `Owner` role or `Role Based Access Control Administrator` role on the registry.

## Scoping role assignments to specific repositories

You can use Microsoft Entra attribute-based access control (ABAC) for managing Microsoft Entra-based repository permissions.
This feature allows you to scope role assignments to specific repositories in a registry.

For an overview of Microsoft Entra ABAC repository permissions, including the ACR built-in roles that support Microsoft Entra ABAC conditions, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
Alternatively, you can consult the [Azure Container Registry roles directory reference](container-registry-rbac-built-in-roles-directory-reference.md) for a list of built-in roles that support Microsoft Entra ABAC conditions.

## Recommended built-in roles by scenario

Apply the principle of least privilege by assigning only the permissions necessary for an identity to perform its intended function.
These common scenarios each have a recommended built-in role.

> [!NOTE]
> The applicable built-in roles and role behavior depends on the registry's "Role assignment permissions mode". This is visible in the "Properties" blade in the Azure portal:
> - **RBAC Registry + ABAC Repository Permissions**: Supports standard RBAC role assignments with optional Microsoft Entra ABAC conditions to scope assignments to specific repositories.
> - **RBAC Registry Permissions**: Supports only standard RBAC assignments without ABAC conditions.
>
> For details on Microsoft Entra ABAC and ABAC-enabled roles, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).

### [Registries configured with "RBAC Registry + ABAC Repository Permissions"](#tab/registries-configured-with-rbac-registry-abac-repository-permissions)

- **Scenario: Identities that need to pull images and validate supply chain artifacts such as developers, pipelines, and container orchestrators (for example, Azure Kubernetes Service node kubelet identity, Azure Container Apps, Azure Container Instances, Azure Machine Learning workspaces)**
  - Role: `Container Registry Repository Reader`
  - Purpose: Grants data plane read-only access to pull images and artifacts, view tags, repositories, Open Container Initiative (OCI) referrers, and artifact streaming configurations. Doesn't include any control plane or write permissions. **Doesn't grant repository catalog list permissions** to list any repositories in the registry.
  - ABAC support: This role supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories in the registry.

- **Scenario: Identities such as CI/CD build pipelines and developers that build and push images, as well as manage image tags**
  - Role: `Container Registry Repository Writer`
  - Permissions: Grants data plane access to push, pull, and update (but not delete) images and artifacts, read/manage tags, read/manage OCI referrers, and enable (but not disable) artifact streaming for repositories and images. Doesn't include any control plane permissions. **Doesn't grant repository catalog list permissions** to list any repositories in the registry.
  - ABAC support: This role supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories in the registry.

- **Scenario: Identities that need to delete images, artifacts, tags, and OCI referrers**
  - Role: `Container Registry Repository Contributor`
  - Permissions: Grants permissions to **read, write, update, and delete** images and artifacts, read/manage/delete tags, read/manage/delete OCI referrers, and enable/disable artifact streaming for repositories and images. Doesn't include any control plane permissions. **Doesn't grant repository catalog list permissions** to list any repositories in the registry.
  - ABAC support: This role supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories in the registry.

- **Scenario: Identities that need to list all repositories in the registry**
  - Role: `Container Registry Repository Catalog Lister`
  - Permissions: Grants data plane access to list all repositories in the registry, including through the `{loginServerURL}/acr/v1/_catalog` or `{loginServerURL}/v2/_catalog` registry API endpoints. Doesn't include any control plane permissions or permissions to push/pull images.
  - ABAC support: This role **doesn't support Microsoft Entra ABAC conditions**. As such, this role assignment will **grant permissions to list all repositories** in the registry.

- **Scenario: Pipelines, identities, and developers that sign images**
  - For signing images with OCI referrers such as [Notary Project](container-registry-tutorial-sign-build-push.md):
    - Role: `Container Registry Repository Writer`
    - Permissions: Grants data plane access to push signatures in the form of OCI referrers attached to images and artifacts. Doesn't include any control plane permissions.
    - ABAC support: This role supports optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories in the registry.
  - For signing images with [Docker Content Trust (DCT)](container-registry-content-trust.md):
    - Signing images with DCT for ABAC-enabled registries is not supported.

- **Scenario: Pipelines, identities, and developers that need to create, update, or _delete_ ACR registries**
  - Role: `Container Registry Contributor and Data Access Configuration Administrator`
  - Permissions:
    - Grants control plane access to **create, configure, manage, and _delete_** registries, including:
      - configure [registry SKUs](container-registry-skus.md)
      - authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md)),
      - high availability features ([geo-replications](container-registry-geo-replication.md), [availability zones, and zone redundancy](zone-redundancy.md)),
      - on-premises features ([connected registries](intro-connected-registry.md)),
      - registry endpoints ([dedicated data endpoints](container-registry-dedicated-data-endpoints.md))
      - network access ([private link and private endpoint settings](container-registry-private-link.md), [public network access](container-registry-private-link.md#disable-public-access-to-a-registry), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - registry policies ([retention policy](container-registry-retention-policy.md), [registry-wide quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
      - manage a [registry's system-assigned managed identity](/cli/azure/acr/identity)
    - Note: this role grants permissions to delete the registry itself.
    - Note: this role doesn't include data plane operations (for example, image push/pull), role assignment capabilities, or ACR task.
    - Note: to manage a [registry's user-assigned managed identity](/cli/azure/acr/identity), the assignee must also have the [`Managed Identity Operator`](/azure/role-based-access-control/built-in-roles/identity#managed-identity-operator) role.
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to manage control plane settings and configurations for the entire registry.

- **Scenario: Pipelines, infrastructure engineers, or control plane observability/monitoring tools that need to list registries and view registry configurations, but not access to registry images**
  - Role: `Container Registry Configuration Reader and Data Access Configuration Reader`
  - Permissions: **Read-only counterpart of the `Container Registry Contributor and Data Access Configuration Administrator` role**. Grants control plane access to view and list registries and inspect registry configurations, but not modify them. Doesn't include data plane operations (for example, image push/pull) or role assignment capabilities.
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to read control plane settings and configurations for the entire registry.

- **Scenario: Vulnerability scanners and tools that need to audit registries and registry configurations, as well as access to registry images to scan them for vulnerabilities**
  - Roles: `Container Registry Repository Reader`, `Container Registry Repository Catalog Lister`, and `Container Registry Configuration Reader and Data Access Configuration Reader`
  - Permissions: Grants control plane access to view and list ACR registries, as well as to audit registry configurations for audit and compliance. Also grants permissions to pull images, artifacts, and view tags to scan and analyze images for vulnerabilities.
  - ABAC support: ACR recommends that vulnerability scanners and monitors have full data plane access to all repositories in the registry. As such, these roles should be assigned without Microsoft Entra ABAC conditions to grant role permissions without scoping them to specific repositories.

- **Scenario: Pipelines and identities that orchestrate [ACR tasks](container-registry-tasks-overview.md)**
  - Role: `Container Registry Tasks Contributor`
  - Permissions: Manage [ACR tasks](container-registry-tasks-overview.md), including task definitions and task runs, [task agent pools](tasks-agent-pools.md), [quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run), and [task logs](container-registry-tasks-logs.md). Doesn't include data plane permissions or broader registry configuration
  - Note: to fully manage [task identities](container-registry-tasks-authentication-managed-identity.md), the assignee must have the [`Managed Identity Operator`](/azure/role-based-access-control/built-in-roles/identity#managed-identity-operator) role.
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to manage all ACR Tasks in the registry.

- **Scenario: Identities such as pipelines and developers that [import images with `az acr import`](container-registry-import-images.md)**
  - Role: `Container Registry Data Importer and Data Reader`
  - Permissions: Grants control plane access to trigger [image imports using `az acr import`](container-registry-import-images.md), and data plane access to validate import success (pull imported images and artifacts, view repository contents, list OCI referrers, and inspect imported tags). Doesn't allow pushing or modifying any content in the registry.
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to import images into any repository in the registry. It also grants permissions to read images in all repositories in the registry.

- **Scenario: Identities such as pipelines and developers that manage [ACR transfer pipelines](container-registry-transfer-cli.md) for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries**
  - Role: `Container Registry Transfer Pipeline Contributor`
  - Permissions: Grants control plane access to manage [ACR import/export transfer pipelines and pipeline runs](container-registry-transfer-cli.md) using intermediary storage accounts. Doesn't include data plane permissions, broader registry access, or permissions to manage other Azure resource types such as storage accounts or key vaults.
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to manage all ACR transfer pipelines in the registry.

- **Scenario: Management of [quarantined images](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)**
  - Roles: `AcrQuarantineReader` and `AcrQuarantineWriter`
  - Permissions: Manage quarantined images in the registry, including listing and pulling quarantined images for further inspection, and modifying the quarantine status of images. Quarantined images are pushed images that can't be pulled or used until they're unquarantined.
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to manage all quarantined images in the registry.

- **Scenario: Developers or processes that configure registry [auto-purge on ACR Tasks](container-registry-auto-purge.md)**
  - Role: `Container Registry Tasks Contributor`
  - Permissions: Grants control plane permissions to manage [auto-purge, which runs on ACR Tasks](container-registry-auto-purge.md).
  - ABAC support: This role doesn't support Microsoft Entra ABAC conditions as the role is scoped to the registry level, granting permissions to manage all ACR Tasks in the registry.

- **Scenario: Visual Studio Code Docker extension users**
  - Roles: `Container Registry Repository Writer`, `Container Registry Tasks Contributor`, and `Container Registry Contributor and Data Access Configuration Administrator`
  - Permissions: Grants capabilities to browse registries, pull and push images, and build images using `az acr build`, supporting common developer workflows in Visual Studio Code.
  - ABAC support: ACR recommends that Visual Studio Code users have full data plane access to all repositories in the registry. As such, these roles should be assigned without Microsoft Entra ABAC conditions to grant role permissions without scoping them to specific repositories.

### [Registries configured with "RBAC Registry Permissions"](#tab/registries-configured-with-rbac-registry-permissions)

- **Scenario: Identities that need to pull images and validate supply chain artifacts such as developers, pipelines, and container orchestrators (for example, Azure Kubernetes Service node kubelet identity, Azure Container Apps, Azure Container Instances, Azure Machine Learning workspaces)**
  - Role: `AcrPull`
  - Purpose: Grants data plane read-only access to pull images and artifacts, view tags, repositories, Open Container Initiative (OCI) referrers, and artifact streaming configurations. Doesn't include any control plane or write permissions.

- **Scenario: Identities such as CI/CD build pipelines and developers that build and push images, as well as manage image tags**
  - Role: `AcrPush`
  - Permissions: Grants data plane access to push and pull images and artifacts, manage tags, work with OCI referrers, and configure artifact streaming for repositories and images. Doesn't include any control plane permissions.

- **Scenario: Pipelines, identities, and developers that sign images**
  - For signing images with OCI referrers such as [Notary Project](container-registry-tutorial-sign-build-push.md):
    - Role: `AcrPush`
    - Permissions: Grants data plane access to push signatures in the form of OCI referrers attached to images and artifacts. Doesn't include any control plane permissions.
  - For signing images with [Docker Content Trust (DCT)](container-registry-content-trust.md):
    - Role: `AcrImageSigner`
    - Permissions: Sign images in the registry with [Docker Content Trust (DCT)](container-registry-content-trust.md).
    - Note: Docker Content Trust is being [deprecated](container-registry-content-trust-deprecation.md).

- **Scenario: Pipelines, identities, and developers that need to create, update, or _delete_ ACR registries**
  - Role: `Container Registry Contributor and Data Access Configuration Administrator`
  - Permissions:
    - Grants control plane access to **create, configure, manage, and _delete_** registries, including:
      - configure [registry SKUs](container-registry-skus.md)
      - authentication access settings ([admin user login credentials](container-registry-authentication.md#admin-account), [anonymous pull](anonymous-pull-access.md), [non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md), and [Microsoft Entra authentication-as-arm token audience](container-registry-disable-authentication-as-arm.md)),
      - high availability features ([geo-replications](container-registry-geo-replication.md), [availability zones, and zone redundancy](zone-redundancy.md)),
      - on-premises features ([connected registries](intro-connected-registry.md)),
      - registry endpoints ([dedicated data endpoints](container-registry-dedicated-data-endpoints.md))
      - network access ([private link and private endpoint settings](container-registry-private-link.md), [public network access](container-registry-private-link.md#disable-public-access-to-a-registry), [trusted services bypass](allow-access-trusted-services.md), [network firewall rules](container-registry-access-selected-networks.md), and [Virtual Network (VNET) service endpoints](container-registry-vnet.md))
      - registry policies ([retention policy](container-registry-retention-policy.md), [registry-wide quarantine enablement](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md), [soft-delete enablement](container-registry-soft-delete-policy.md), and [data exfiltration export policy](data-loss-prevention.md))
      - [diagnostics and monitoring settings](monitor-container-registry.md) (diagnostic settings, logs, metrics, [webhooks for registries and geo-replications](container-registry-webhook.md), and [Event Grid](container-registry-event-grid-quickstart.md))
      - manage a [registry's system-assigned managed identity](/cli/azure/acr/identity)
    - Note: this role grants permissions to delete the registry itself.
    - Note: this role doesn't include data plane operations (for example, image push/pull), role assignment capabilities, or ACR task.
    - Note: to manage a [registry's user-assigned managed identity](/cli/azure/acr/identity), the assignee must also have the [`Managed Identity Operator`](/azure/role-based-access-control/built-in-roles/identity#managed-identity-operator) role.

- **Scenario: Pipelines, infrastructure engineers, or control plane observability/monitoring tools that need to list registries and view registry configurations, but not access to registry images**
  - Role: `Container Registry Configuration Reader and Data Access Configuration Reader`
  - Permissions: **Read-only counterpart of the `Container Registry Contributor and Data Access Configuration Administrator` role**. Grants control plane access to view and list registries and inspect registry configurations, but not modify them. Doesn't include data plane operations (for example, image push/pull) or role assignment capabilities.

- **Scenario: Vulnerability scanners and tools that need to audit registries and registry configurations, as well as access to registry images to scan them for vulnerabilities**
  - Roles: `AcrPull` and `Container Registry Configuration Reader and Data Access Configuration Reader`
  - Permissions: Grants control plane access to view and list ACR registries, as well as to audit registry configurations for audit and compliance. Also grants permissions to pull images, artifacts, and view tags to scan and analyze images for vulnerabilities.

- **Scenario: Pipelines and identities that orchestrate [ACR tasks](container-registry-tasks-overview.md)**
  - Role: `Container Registry Tasks Contributor`
  - Permissions: Manage [ACR tasks](container-registry-tasks-overview.md), including task definitions and task runs, [task agent pools](tasks-agent-pools.md), [quick builds with `az acr build`](/cli/azure/acr#az-acr-build) and [quick runs with `az acr run`](/cli/azure/acr#az-acr-run), and [task logs](container-registry-tasks-logs.md). Doesn't include data plane permissions or broader registry configuration
  - Note: to fully manage [task identities](container-registry-tasks-authentication-managed-identity.md), the assignee must have the [`Managed Identity Operator`](/azure/role-based-access-control/built-in-roles/identity#managed-identity-operator) role.

- **Scenario: Identities such as pipelines and developers that [import images with `az acr import`](container-registry-import-images.md)**
  - Role: `Container Registry Data Importer and Data Reader`
  - Permissions: Grants control plane access to trigger [image imports using `az acr import`](container-registry-import-images.md), and data plane access to validate import success (pull imported images and artifacts, view repository contents, list OCI referrers, and inspect imported tags). Doesn't allow pushing or modifying any content in the registry.

- **Scenario: Identities such as pipelines and developers that manage [ACR transfer pipelines](container-registry-transfer-cli.md) for transferring artifacts between registries using intermediary storage accounts across network, tenant, or air gap boundaries**
  - Role: `Container Registry Transfer Pipeline Contributor`
  - Permissions: Grants control plane access to manage [ACR import/export transfer pipelines and pipeline runs](container-registry-transfer-cli.md) using intermediary storage accounts. Doesn't include data plane permissions, broader registry access, or permissions to manage other Azure resource types such as storage accounts or key vaults.

- **Scenario: Management of [quarantined images](https://github.com/Azure/acr/blob/main/docs/preview/quarantine/readme.md)**
  - Roles: `AcrQuarantineReader` and `AcrQuarantineWriter`
  - Permissions: Manage quarantined images in the registry, including listing and pulling quarantined images for further inspection, and modifying the quarantine status of images. Quarantined images are pushed images that can't be pulled or used until they're unquarantined.

- **Scenario: Deleting images, artifacts, tags, and OCI referrers**
  - Role: `AcrDelete`
  - Permissions: Provides permissions to delete artifacts and tags across all repositories in the registry. This role doesn't grant permissions to delete the registry itself.

- **Scenario: Developers or processes that configure registry [auto-purge on ACR Tasks](container-registry-auto-purge.md)**
  - Role: `Container Registry Tasks Contributor`
  - Permissions: Grants control plane permissions to manage [auto-purge, which runs on ACR Tasks](container-registry-auto-purge.md).

- **Scenario: Visual Studio Code Docker extension users**
  - Roles: `AcrPush`, `Container Registry Tasks Contributor`, and `Container Registry Contributor and Data Access Configuration Administrator`
  - Permissions: Grants capabilities to browse registries, pull and push images, and build images using `az acr build`, supporting common developer workflows in Visual Studio Code.

---

## Next steps

* To perform role assignments with optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
* For a detailed reference of every ACR built-in role, including the permissions granted by each role, see the [Azure Container Registry roles directory reference](container-registry-rbac-built-in-roles-directory-reference.md).
* For more information on creating custom roles that meet your specific needs and requirements, see [Azure Container Registry custom roles](container-registry-rbac-custom-roles.md).
