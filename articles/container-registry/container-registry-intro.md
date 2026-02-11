---
title: Introduction to Azure Container Registry
description: Get basic information about the Azure service that provides cloud-based, managed container registries.
author: rayoef
ms.topic: overview
ms.date: 02/11/2026
ms.author: rayoflores
ms.service: azure-container-registry
ms.custom: mvc
# Customer intent: "As a developer managing containerized applications, I want to leverage a managed container registry service, so that I can efficiently store, build, and deploy container images within my development and deployment workflows."
---

# Introduction to Azure Container Registry

Azure Container Registry is a managed registry service based on the open-source [Docker platform](https://docs.docker.com/get-started/docker-overview/). Create and maintain Azure container registries to store and manage your [container images and related artifacts](container-registry-concepts.md).

Use container registries with your existing container development and deployment pipelines, or use Azure Container Registry tasks to build container images in Azure. Build on demand, or fully automate builds with triggers such as source code commits and base image updates.

Azure provides tooling like the Azure CLI, the Azure portal, and API support to manage your container registries. Optionally, install the [Container Tools extension](https://code.visualstudio.com/docs/containers/overview) for Visual Studio Code. You can use this extension to perform tasks such as pulling and pushing images to your registry, all within Visual Studio Code.

Container registries are available in three tiers (SKUs): Basic, Standard, and Premium. Each tier supports common features such as webhook integration, registry authentication with Microsoft Entra ID, and delete functionality. Premium registries support additional capabilities and have greater storage and limits. For more details about the different tiers, see [Azure Container Registry SKUs](container-registry-skus.md).

Take advantage of local, network-close storage of your container images by creating a registry in the same Azure location as your deployments. The [geo-replication](container-registry-geo-replication.md) feature of Premium registries supports advanced replication and container image distribution across geographies.

## Container registry deployment targets and workflows

Pull images from an Azure container registry to various deployment targets:

* *Scalable orchestration systems* that manage containerized applications across clusters of hosts, including [Kubernetes](https://kubernetes.io/docs/), [DC/OS](https://dcos.io/), and [Docker Swarm](https://docs.docker.com/guides/swarm-deploy/).
* *Azure services* that support building and running applications at scale, such as [Azure Kubernetes Service (AKS)](/azure/aks/what-is-aks), [App Service](/azure/app-service/overview), [Batch](/azure/batch/batch-technical-overview), and [Service Fabric](/azure/service-fabric/service-fabric-overview).

Developers can also push to a container registry as part of a container development workflow. For example, you can target a container registry from a continuous integration and continuous delivery (CI/CD) tool such as [Azure Pipelines](/azure/devops/pipelines/ecosystems/containers/acr-template) or [Jenkins](https://jenkins.io/).

Configure [Azure Container Registry tasks](container-registry-tasks-overview.md)  to automatically rebuild application images when their base images are updated, or automate image builds when your team commits code to a Git repository. Use tasks to extend your development inner loop to the cloud by offloading `docker build` operations to Azure, or configure build tasks to automate your container OS and framework patching pipeline. Create [multi-step tasks](container-registry-tasks-overview.md#multi-step-tasks) to automate building, testing, and patching container images in parallel in the cloud.

## Supported images and artifacts

When images are grouped in a repository, each image is a read-only snapshot of a Docker-compatible container. Azure container registries can include both Windows and Linux images. You control image names for all your container deployments.

Use standard [Docker commands](https://docs.docker.com/reference/cli/docker/) to push images into a repository or pull an image from a repository. Azure Container Registry stores Docker container images and [related content formats](container-registry-image-formats.md) such as [Helm charts](container-registry-helm-repos.md) and images built to the [Open Container Initiative (OCI) Image Format Specification](https://github.com/opencontainers/image-spec/blob/master/spec.md).

## Container registry security and access

You authenticate to a registry by using the Azure CLI or the standard `docker login` command. Azure Container Registry transfers container images over HTTPS, and it supports TLS to help secure client connections.

[Control access](container-registry-authentication.md) to a container registry by using an Azure identity, a Microsoft Entra [service principal](/azure/active-directory/develop/app-objects-and-service-principals), or a provided admin account. Use [Azure role-based access control (RBAC)](container-registry-rbac-built-in-roles-directory-reference.md) to assign specific registry permissions to users or systems.

Security features of the Premium service tier include [content trust](container-registry-content-trust.md) for image tag signing, and [private endpoints (preview)](container-registry-private-link.md) to restrict access to the registry. Microsoft Defender for Cloud optionally integrates with Azure Container Registry to [scan images](/azure/container-registry/scan-images-defender) whenever you push an image to a registry.

## Related content

* [Create a container registry by using the Azure portal](container-registry-get-started-portal.md).
* [Create a container registry by using the Azure CLI](container-registry-get-started-azure-cli.md).
* [Automate container builds and maintenance by using Azure Container Registry tasks](container-registry-tasks-overview.md).
