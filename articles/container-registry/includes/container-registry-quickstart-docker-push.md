---
title: include file
description: include file
services: container-registry
author: chasedmicrosoft

ms.service: azure-container-registry
ms.topic: include
ms.date: 08/04/2020
ms.author: doveychase
ms.custom: include file
# Customer intent: As a developer, I want to push container images to a container registry, so that I can manage and deploy my applications efficiently.
---
## Push image to registry

To push an image to an Azure Container registry, you must first have an image. If you don't yet have any local container images, run the following [docker pull][docker-pull] command to pull an existing public image. For this example, pull the `hello-world` image from Microsoft Container Registry.

```
docker pull mcr.microsoft.com/hello-world
```

Before you can push an image to your registry, you must tag it using the [docker tag][docker-tag] with the fully qualified name of your registry login server.

* The login server name format for Domain Name Label (DNL) protected registries with a unique DNS name hash included is `mycontainerregistry-abc123.azurecr.io`.
* The login server name format for registries created with the `Unsecure` DNL option is `mycontainerregistry.azurecr.io`.

For example, if your registry was created with the `Tenant Reuse` DNL scope, the login server might look like `mycontainerregistry-abc123.azurecr.io` with a hash in the DNS name. If your registry was created with the `Unsecure` DNL option, the login server would look like `mycontainerregistry.azurecr.io` without the hash.

For more details on DNL options during registry creation and DNS name implications, see [Quickstart - Create Registry in Portal](../container-registry-get-started-portal.md#configure-domain-name-label-dnl-option).

### Example: Tagging an image before push

Tag the image using the [docker tag][docker-tag] command using your registry's login server.

#### Tagging image for a non-DNL registry:

```
docker tag mcr.microsoft.com/hello-world mycontainerregistry.azurecr.io/hello-world:v1
```

#### Tagging image for a DNL-enabled registry:

```
docker tag mcr.microsoft.com/hello-world mycontainerregistry-abc123.azurecr.io/hello-world:v1
```

Finally, use [docker push][docker-push] to push the image to the registry instance. Replace `<login-server>` with the login server name of your registry instance. This example creates the **hello-world** repository, containing the `hello-world:v1` image.

```
docker push <login-server>/hello-world:v1
```

After pushing the image to your container registry, remove the `hello-world:v1` image from your local Docker environment. (Note that this [docker rmi][docker-rmi] command doesn't remove the image from the **hello-world** repository in your Azure container registry.)

```
docker rmi <login-server>/hello-world:v1
```

<!-- LINKS - External -->
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/

<!-- LINKS - Internal -->

