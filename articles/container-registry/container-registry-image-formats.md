---
title: Supported Content Formats by ACR, Docker, OCI, and Helm
description: Learn about content formats supported by Azure Container Registry, including Docker-compatible container images, Helm charts, OCI images, and OCI artifacts.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 10/31/2023
---

# Content formats supported in Azure Container Registry

Use a private repository in Azure Container Registry to manage one of the following content formats. 

## Docker-compatible container images

The following Docker container image formats are supported:

* [Docker Image Manifest V2, Schema 1](https://docs.docker.com/registry/spec/manifest-v2-1/)

* [Docker Image Manifest V2, Schema 2](https://docs.docker.com/registry/spec/manifest-v2-2/) - includes Manifest Lists which allow registries to store [multi-architecture images](push-multi-architecture-images.md) under a single `image:tag` reference

## OCI images

Azure Container Registry supports images that meet the [Open Container Initiative (OCI) Image Format Specification](https://github.com/opencontainers/image-spec/blob/master/spec.md), including the optional [image index](https://github.com/opencontainers/image-spec/blob/master/image-index.md) specification. Packaging formats include [Singularity Image Format (SIF)](https://github.com/sylabs/sif).

## OCI artifacts

Azure Container Registry supports the [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec), a vendor-neutral, cloud-agnostic spec to store, share, secure, and deploy container images and other content types (artifacts). The specification allows a registry to store a wide range of artifacts in addition to container images. You use tooling appropriate to the artifact to push and pull artifacts. For examples, see:

* [Push and pull an OCI artifact using an Azure container registry](container-registry-manage-artifact.md)
* [Push and pull Helm charts to an Azure container registry](container-registry-helm-repos.md)

To learn more about OCI artifacts, see the [OCI Registry as Storage (ORAS)](https://github.com/deislabs/oras) repo and the [OCI Artifacts](https://github.com/opencontainers/artifacts) repo on GitHub.

## Helm charts

Azure Container Registry can host repositories for [Helm charts](https://helm.sh/), a packaging format used to quickly manage and deploy applications for Kubernetes. [Helm client](https://docs.helm.sh/using_helm/#installing-helm) version 3 is recommended. See [Push and pull Helm charts to an Azure container registry](container-registry-helm-repos.md).

## Next steps

* See how to [push and pull](container-registry-get-started-docker-cli.md) images with Azure Container Registry.

* Use [ACR tasks](container-registry-tasks-overview.md) to build and test container images. 

* Use the [Moby BuildKit](https://github.com/moby/buildkit) to build and package containers in OCI format.

* Set up a [Helm repository](container-registry-helm-repos.md) hosted in Azure Container Registry. 


