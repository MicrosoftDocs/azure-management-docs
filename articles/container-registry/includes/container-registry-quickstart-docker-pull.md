---
title: include file
description: include file
services: container-registry
author: rayoef

ms.service: azure-container-registry
ms.topic: include
ms.date: 08/04/2020
ms.author: rayoflores
ms.custom: include file
# Customer intent: As a developer, I want to pull and run container images from my container registry, so that I can quickly verify the functionality of my applications during the development process.
---

## Run image from registry

Now, you can pull and run the `hello-world:v1` container image from your container registry by using [docker run][docker-run]:

```
docker run <login-server>/hello-world:v1  
```

Example output: 

```
Unable to find image 'mycontainerregistry.azurecr.io/hello-world:v1' locally
v1: Pulling from hello-world
Digest: sha256:662dd8e65ef7ccf13f417962c2f77567d3b132f12c95909de6c85ac3c326a345
Status: Downloaded newer image for mycontainerregistry.azurecr.io/hello-world:v1

Hello from Docker!
This message shows that your installation appears to be working correctly.

[...]
```

<!-- LINKS - External -->
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
