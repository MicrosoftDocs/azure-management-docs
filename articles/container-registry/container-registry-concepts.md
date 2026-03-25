---
title: About Registries, Repositories, Images, and Artifacts
description: Introduction to key concepts of Azure container registries, repositories, container images, and other artifacts.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 03/25/2026
# Customer intent: As a DevOps engineer, I want to understand the concepts of container registries, repositories, and artifacts so that I can effectively manage and utilize container images in my deployment workflows.
---

# About registries, repositories, and artifacts

This article introduces the key concepts of container registries, repositories, container images, and related artifacts.

:::image type="content" source="media/container-registry-concepts/registry-elements.png" alt-text="Registry, repositories, and artifacts":::

## Registry

A container *registry* is a service that stores and distributes container images and related artifacts. Docker Hub is an example of a public container registry that serves as a general catalog of Docker container images. Azure Container Registry provides you with direct control of your container content, with integrated authentication, [geo-replication](container-registry-geo-replication.md) supporting global distribution and reliability for network-close deployments, [virtual network configuration with Private Link](container-registry-private-link.md), [tag locking](container-registry-image-lock.md), and many other enhanced features.

In addition to Docker-compatible container images, Azure Container Registry supports a range of [content artifacts](container-registry-image-formats.md) including Helm charts and Open Container Initiative (OCI) image formats.

## Repository

A *repository* is a collection of container images or other artifacts in a registry that have the same name, but different tags. For example, the following three images are in the `acr-helloworld` repository:

- `acr-helloworld:latest`
- `acr-helloworld:v1`
- `acr-helloworld:v2`

Repository names can also include [namespaces](container-registry-best-practices.md#repository-namespaces). By using forward slash-delimited names, you can identify related repositories and artifact ownership in your organization. However, the registry manages all repositories independently, not as a hierarchy. For example:

- `marketing/campaign10-18/web:v2`
- `marketing/campaign10-18/api:v3`
- `marketing/campaign10-18/email-sender:v2`
- `product-returns/web-submission:20230604`
- `product-returns/legacy-integrator:20230715`

Repository names can only include lowercase alphanumeric characters, periods, dashes, underscores, and forward slashes.

## Artifact

A container image or other artifact within a registry is associated with one or more tags, has one or more layers, and is identified by a manifest. Understanding how these components relate to each other can help you manage your registry effectively.

### Tag

The *tag* for an image or other artifact specifies its version. You can assign one or many tags to a single artifact within a repository. You can also "untag" an artifact by deleting all tags from an image while keeping the image's data (its layers) in the registry.

The repository (or repository and namespace) plus a tag defines an image's name. You can push and pull an image by specifying its name in the push or pull operation. The tag `latest` is used by default if you don't provide one in your Docker commands.

How you tag container images depends on your scenarios for developing or deploying them. For example, use stable tags to maintain your base images, and use unique tags for deploying images. For more information, see [Recommendations for tagging and versioning container images](container-registry-image-tag-version.md).

For tag naming rules, see the [Docker documentation](https://docs.docker.com/engine/reference/commandline/tag/).

### Layer

Container images and artifacts consist of one or more *layers*. Different artifact types define layers differently. For example, in a Docker container image, each layer corresponds to a line in the Dockerfile that defines the image:

:::image type="content" source="media/container-registry-concepts/container-image-layers.png" alt-text="Layers of a container image":::

Artifacts in a registry share common layers, which increases storage efficiency. For example, several images in different repositories might have a common ASP.NET Core base layer, but the registry stores only one copy of that layer. Layer sharing also optimizes layer distribution to nodes, with multiple artifacts sharing common layers. If an image already on a node includes the ASP.NET Core layer as its base, the subsequent pull of a different image referencing the same layer doesn't transfer the layer to the node. Instead, it references the layer already existing on the node.

To provide secure isolation and protection from potential layer manipulation, registries don't share layers.

### Manifest

Each container image or artifact you push to a container registry has a *manifest*. The registry generates the manifest when you push the content. The manifest uniquely identifies the artifacts and specifies the layers.

A basic manifest for a Linux `hello-world` image looks similar to the following:

  ```json
  {
    "schemaVersion": 2,
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 1510,
      "digest": "sha256:fbf289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e"
    },
    "layers": [
      {
        "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
        "size": 977,
        "digest": "sha256:2c930d010525941c1d56ec53b97bd057a67ae1865eebf042686d2a2d18271ced"
      }
    ]
  }
  ```

Use the Azure CLI command [`az acr manifest list-metadata`][az-acr-manifest-list-metadata] to list the manifests for a repository:

```azurecli
az acr manifest list-metadata --name <repositoryName> --registry <acrName>
```

For example, list the manifests for the `acr-helloworld` repository:

```azurecli
az acr manifest list-metadata --name acr-helloworld --registry myregistry
```

```output
[
  {
    "digest": "sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108",
    "tags": [
      "latest",
      "v3"
    ],
    "timestamp": "2023-07-12T15:52:00.2075864Z"
  },
  {
    "digest": "sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57",
    "tags": [
      "v2"
    ],
    "timestamp": "2023-07-12T15:50:53.5372468Z"
  },
  {
    "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
    "tags": [
      "v1"
    ],
    "timestamp": "2023-07-11T21:38:35.9170967Z"
  }
]
```

### Manifest digest

Manifests have a unique SHA-256 hash called a *manifest digest*. Each image or artifact - whether tagged or not - has its own digest. The digest value is unique even if the artifact's layer data matches another artifact. This feature lets you push identically tagged images to a registry without any problems. For example, you can push `myimage:latest` to your registry over and over without an error, because each image has its own unique digest.

You can pull an artifact from a registry by adding its digest to the pull operation. Some systems might be set up to pull by digest because it guarantees the image version you're pulling, even if you push an identically tagged image later to the registry.

> [!IMPORTANT]
> If you repeatedly push modified artifacts with identical tags, you might create **orphans**: artifacts that are untagged but still use space in your registry. Untagged images don't show up in the Azure CLI or in the Azure portal when you list or view images by tag. However, their layers still exist and use space in your registry. Deleting an untagged image frees registry space when the manifest is the only one or the last one pointing to a particular layer. To learn how to free space used by untagged images, see [Delete container images in Azure Container Registry](container-registry-delete.md).

## Address an artifact

To address a registry artifact for push and pull operations by using Docker or other client tools, combine the fully qualified registry name, repository name (including namespace path if applicable), and an artifact tag or manifest digest. The general format is:

  **Address by tag**: `[loginServerUrl]/[repository][:tag]`

  **Address by digest**: `[loginServerUrl]/[repository]@[sha256:digest]`  

When you use Docker or other client tools to pull or push artifacts to an Azure container registry, use the registry's fully qualified URL, also called the *login server* name. In the Azure cloud, the fully qualified URL of an Azure container registry is in the format `myregistry.azurecr.io` (all lowercase). The tag `latest` is used by default if you don't provide a tag in your command.  

> [!NOTE]
> You can't specify a port number in the registry login server URL, such as `myregistry.azurecr.io:443`.

Examples of pushing by tag:

`docker push myregistry.azurecr.io/samples/myimage:20230106`

`docker push myregistry.azurecr.io/marketing/email-sender`

Example of pulling by tag:

`docker pull myregistry.azurecr.io/marketing/campaign10-18/email-sender:v2`

Example of pulling by manifest digest:

`docker pull myregistry.azurecr.io/acr-helloworld@sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108`

## Next steps

- Learn more about [registry storage](container-registry-storage.md) and [supported content formats](container-registry-image-formats.md) in Azure Container Registry.

- Learn how to [push and pull images](container-registry-get-started-docker-cli.md) from Azure Container Registry.

<!-- LINKS - Internal -->
[az-acr-manifest-list-metadata]: /cli/azure/acr/manifest#az-acr-manifest-list-metadata
