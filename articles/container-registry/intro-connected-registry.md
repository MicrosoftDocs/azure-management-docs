---
title: Connected Registry in Azure Container Registry
description: Discover the connected registry feature in Azure Container Registry. Learn about its benefits and practical use cases for container management.
ms.author: jeburke
ms.service: azure-container-registry
ms.topic: overview
ms.date: 10/31/2023
ms.custom: references_regions
#customer intent: As a reader, I want to understand the overview and scenarios of the connected registry feature of Azure Container Registry so that I can utilize it effectively.
---

# What is a connected registry? 

In this article, you learn about the *connected registry* feature of [Azure Container Registry](container-registry-intro.md). A connected registry is an on-premises or remote replica that synchronizes container images with your cloud-based Azure container registry. Use a connected registry to help speed-up access to registry artifacts on-premises or remote.

## Billing and Support

The connected registry is a preview feature of the **Premium** container registry service tier, and subject to [limitations](#limitations). For information about registry service tiers and limits, see [Azure Container Registry service tiers](container-registry-skus.md).

>[!IMPORTANT]
> Please note that there are **Important upcoming changes** to the connected registry Deployment Model Support and Billing starting from January 1st, 2025. For any inquiries or assistance with the transition, please reach out to the customer support team.

### Billing

- The connected registry incurs no charges until it reaches general availability (GA).
- Post-GA, a monthly price of $10 will apply for each connected registry deployed.
- This price represents Microsoft's commitment to deliver high-quality services and product support.
- The price is applied to the Azure subscription associated with the parent registry.

### Support

- Microsoft will end support for the connected registry deployment on IoT Edge devices on January 1st, 2025.
- After January 1st, 2025 connected registry will solely support Arc-enabled Kubernetes clusters as the deployment model.
- Microsoft advises users to begin planning their transition to Arc-enabled Kubernetes clusters as the deployment model.

## Available regions

A connected registry can be deployed in any region where Azure Container Registry is available.

## Scenarios

A cloud-based Azure container registry provides [features](container-registry-intro.md#key-features) including geo-replication, integrated security, Azure-managed storage, and integration with Azure development and deployment pipelines. At the same time, customers are extending their cloud investments to their on-premises and field solutions.

To run with the required performance and reliability in on-premises or remote environments, container workloads need container images and related artifacts to be available nearby. The connected registry provides a performant, on-premises registry solution that regularly synchronizes content with a cloud-based Azure container registry.

Scenarios for a connected registry include:

* Connected factories
* Point-of-sale retail locations
* Shipping, oil-drilling, mining, and other occasionally connected environments

## How does the connected registry work?

The connected registry is deployed on a server or device on-premises, or an environment that supports container workloads on-premises such as Azure IoT Edge and Azure Arc-enabled Kubernetes. The connected registry synchronizes container images and other OCI artifacts with a cloud-based Azure container registry.

The following image shows a typical deployment model for the connected registry using IoT Edge. 

:::image type="content" source="media/intro-connected-registry/connected-registry-edge.png" alt-text="Diagram of connected registry overview using IoT Edge.":::

The following image shows a typical deployment model for the connected registry using Azure Arc-enabled Kubernetes. 

:::image type="content" source="media/intro-connected-registry/connected-registry-azure-arc.png" alt-text="Diagram of connected registry overview using Arc-enabled Kubernetes.":::

### Deployment

Each connected registry is a resource you manage within a cloud-based Azure container registry. The top parent in the connected registry hierarchy is an Azure container registry in the Azure cloud. The connected registry can be deployed either on Azure IoT Edge or Arc-enabled Kubernetes clusters.

To install the connected registry, use Azure tools on a server or device on your premises, or in an environment that supports on-premises container workloads, such as [Azure IoT Edge](/azure/iot-edge/tutorial-nested-iot-edge). 

Deploy the connected registry Arc extension to the Arc-enabled Kubernetes cluster. Secure the connection with TLS using default configurations for read-only access and a continuous sync window. This setup allows the connected registry to synchronize images from the Azure container registry (ACR) to the connected registry on-premises, enabling image pulls from the connected registry.

The connected registry's *activation status* indicates whether it's deployed on-premises.

* **Active** - The connected registry is currently deployed on-premises. It can't be deployed again until it's deactivated. 
* **Inactive** - The connected registry is not deployed on-premises. It can be deployed at this time.  
 
### Content synchronization

The connected registry regularly accesses the cloud registry to synchronize container images and OCI artifacts. 

It can also be configured to synchronize a subset of the repositories from the cloud registry or to synchronize only during certain intervals to reduce traffic between the cloud and the premises.

### Modes

A connected registry can work in one of two modes: *ReadWrite* or *ReadOnly*

**ReadOnly mode** - The default mode, when the connected registry is in ReadOnly mode, clients can only pull (read) artifacts. This configuration is used in scenarios where clients need to pull a container image to operate. This default mode aligns with our secure-by-default approach and is effective starting with CLI version 2.60.0.

**ReadWrite mode** - This mode allows clients to pull and push artifacts (read and write) to the connected registry. Artifacts that are pushed to the connected registry will be synchronized with the cloud registry. The ReadWrite mode is useful when a local development environment is in place. The images are pushed to the local connected registry and from there synchronized to the cloud.

### Registry hierarchy

Each connected registry must be connected to a parent. The top parent is the cloud registry. For hierarchical scenarios such as [nested IoT Edge][overview-connected-registry-and-iot-edge], you can nest connected registries in either mode. The parent connected to the cloud registry can operate in either mode. 

Child registries must be compatible with their parent capabilities. Thus, both ReadOnly and ReadWrite modes of the connected registries can be children of a connected registry operating in ReadWrite mode, but only a ReadOnly mode registry can be a child of a connected registry operating in ReadOnly mode.  

## Client access

On-premises clients use standard tools such as the Docker CLI to push or pull content from a Connected registry. To manage client access, you create Azure container registry [non-Microsoft Entra tokens][non-Microsoft Entra token-based repository permissions] for access to each connected registry. You can scope the client tokens for pull or push access to one or more repositories in the registry.

Each connected registry also needs to regularly communicate with its parent registry. For this purpose, the registry is issued a synchronization token (*sync token*) by the cloud registry. This token is used to authenticate with its parent registry for synchronization and management operations.

For more information, see [Manage access to a connected registry][overview-connected-registry-access].

## Limitations

- Number of tokens and scope maps is [limited](container-registry-skus.md) to 20,000 each for a single container registry. This indirectly limits the number of connected registries for a cloud registry, because every Connected registry needs a sync and client token.
- Number of repository permissions in a scope map is limited to 500.
- Number of clients for the connected registry is currently limited to 20.
- [Image locking](container-registry-image-lock.md) through repository/manifest/tag metadata isn't currently supported for connected registries.
- [Repository delete](container-registry-delete.md) isn't supported on the connected registry using ReadOnly mode.
- [Resource logs](monitor-service-reference.md#resource-logs) for connected registries are currently not supported.
- Connected registry is coupled with the registry's home region data endpoint. Automatic migration for [geo-replication](container-registry-geo-replication.md) isn't supported.
- Deletion of a connected registry needs manual removal of the containers on-premises and removal of the respective scope map or tokens in the cloud.
- Connected registry sync limitations are as follows:
  - For continuous sync:
    - `minMessageTtl` is one day
    - `maxMessageTtl` is 90 days
  - For occasionally connected scenarios, where you want to specify sync window:
    - `minSyncWindow` is 1 hr
    - `maxSyncWindow` is seven days

## Conclusion

In this overview, you learned about the connected registry and some basic concepts. Continue to the one of the following articles to learn about specific scenarios where connected registry can be utilized.

> [!div class="nextstepaction"]
<!-- LINKS - internal -->
[overview-connected-registry-access]:overview-connected-registry-access.md
[overview-connected-registry-and-iot-edge]:overview-connected-registry-and-iot-edge.md
[non-Microsoft Entra token-based repository permissions]: container-registry-token-based-repository-permissions.md
