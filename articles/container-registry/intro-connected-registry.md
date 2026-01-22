---
title: Connected Registry in Azure Container Registry
description: Use the connected registry of Azure Container Registry to help speed up access to registry artifacts. on-premises or remote.
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: overview
ms.date: 05/20/2025
ms.custom: references_regions
# Customer intent: "As a container management administrator, I want to understand the connected registry feature of Azure Container Registry so that I can effectively synchronize container images between on-premises and cloud environments to enhance accessibility and performance for my workloads."
---

# What is a connected registry? 

The connected registry feature of [Azure Container Registry](container-registry-intro.md) ebables an on-premises or remote replica that synchronizes container images with your cloud-based Azure container registry. Use a connected registry to help speed up access to registry artifacts hosted on-premises or remotely.

A cloud-based Azure container registry provides [features](container-registry-intro.md#key-features) including geo-replication, integrated security, Azure-managed storage, and integration with Azure development and deployment pipelines. However, customers might extend their cloud investments to their on-premises and field solutions, requiring options outside of the standard cloud-based Azure container registry, or they may have intermittent or limited connectivity to the cloud.

In on-premises or remote environments, container workloads need container images and related artifacts to be available nearby in order to maintain performance and reliability. Connected registries provide a performant, on-premises registry solution that regularly synchronizes content with a cloud-based Azure container registry.

## Pricing and availability

The connected registry feature is currently available only for the **Premium** [service tier](container-registry-skus.md) (SKU).

You can deploy a connected registry in any region where Azure Container Registry is available.

Changes to connected registry billing begin on August 1, 2025. These changes include a monthly charge applied to the Azure subscription associated with the parent registry. For more information, see [Azure Container Registry pricing](https://azure.microsoft.com/pricing/details/container-registry/).

## How does the connected registry work?

You can deploy a connected registry on a server or device on-premises, or in an environment that supports container workloads on-premises, such as [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/overview). The connected registry synchronizes container images and other OCI (Open Container Initiative) artifacts with a cloud-based Azure container registry.

The following diagram shows a typical deployment model for a connected registry using an Azure Arc-enabled Kubernetes cluster.

:::image type="content" source="media/intro-connected-registry/connected-registry-azure-arc.png" alt-text="Diagram of connected registry overview using Arc-enabled Kubernetes.":::

### Deployment and synchronization

You manage each connected registry as a resource within a cloud-based Azure container registry. This Azure container registry is considered to be the parent of the connected registry.

You can [deploy the connected registry Arc extension to the Arc-enabled Kubernetes cluster](quickstart-connected-registry-arc-cli.md). Secure the connection by using TLS (Transport Layer Security) with default configurations for read-only access and a continuous sync window. This setup allows the connected registry to synchronize images from the Azure container registry (ACR) to the connected registry on-premises, enabling image pulls from the connected registry.

The connected registry's *activation status* indicates whether it's been deployed on-premises (**Active**) or not currently deployed (**Inactive**).

Once the connected registry is **Active**, it regularly accesses the cloud registry to synchronize container images and OCI artifacts.

You can also [configure synchronization](tutorial-connected-registry-sync.md) to synchronize a subset of the repositories from the cloud registry, or to synchronize only during certain intervals, to reduce traffic between the cloud and the premises.

### Modes

A connected registry works in one of two modes: `ReadWrite` or `ReadOnly`.

**`ReadOnly mode`** is the default. When the connected registry is in `ReadOnly` mode, clients can only pull (read) artifacts. Use this configuration in scenarios where clients need to pull a container image to operate. This default mode aligns with the secure-by-default approach.

**`ReadWrite mode`** - This mode allows clients to [pull and push artifacts (read and write) to the connected registry](pull-images-from-connected-registry.md). Artifacts that you push to the connected registry synchronize with the cloud registry. The ReadWrite mode is useful when a local development environment is in place. You push the images to the local connected registry, and from there they're synchronized to the cloud-based parent registry.

Connected registries must be compatible with their parent registry capabilities. If a parent registry uses `ReadWrite` mode, child registries can be in either mode. However, if the parent uses `ReadOnly` mode, any child registries must also use `ReadOnly` mode.  

## Client access

On-premises clients use standard tools such as the Docker CLI to push or pull content from a connected registry. To manage client access, create Azure container registry [non-Microsoft Entra tokens](container-registry-token-based-repository-permissions.md) for access to each connected registry. You can scope the client tokens for pull or push access to one or more repositories in the registry.

Each connected registry also needs to regularly communicate with its parent registry. For this purpose, the registry is issued a synchronization token (*sync token*) by the cloud registry. This token is used to authenticate with its parent registry for synchronization and management operations.

For more information, see [Manage access to a connected registry][overview-connected-registry-access].

## Current limitations

When you use connected registries, be aware of the following limitations:

- The number of tokens and scope maps for a single container registry is [limited](container-registry-skus.md) to 20,000 each. This limit indirectly limits the number of connected registries for a cloud registry, because every connected registry needs a sync token and a client token.
- The number of repository permissions in a scope map is limited to 500.
- The number of clients for the connected registry is limited to 50.
- The number of connected registries per container registry is limited to 50.
- [Image locking](container-registry-image-lock.md) through repository, manifest, or tag metadata isn't supported for connected registries.
- [Repository delete](container-registry-delete.md) isn't supported on the connected registry when using ReadOnly mode.
- [Resource logs](monitor-service-reference.md#resource-logs) for connected registries aren't supported.
- Connected registry is coupled with the registry's home region data endpoint. Automatic migration for [geo-replication](container-registry-geo-replication.md) isn't supported.
- Deletion of a connected registry requires manual removal of the containers on-premises and removal of the respective scope map or tokens in the cloud.
- Connected registry sync limitations are as follows:
  - For continuous sync:
    - `minMessageTtl` is one day.
    - `maxMessageTtl` is 90 days.
  - For occasionally connected scenarios, where you want to specify sync window:
    - `minSyncWindow` is 1 hr.
    - `maxSyncWindow` is seven days.

## Next steps

- Learn how to [create a connected registry resource](quickstart-create-connected-registry.md) and [deploy the connected registry to Azure Arc-enabled Kubernetes](quickstart-connected-registry-arc-cli.md).