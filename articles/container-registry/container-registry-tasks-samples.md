---
title: Sample Azure Container Registry (ACR) Tasks
description: Explore sample Azure Container Registry (ACR) tasks to efficiently build, run, and patch container images
author: chasedmicrosoft
ms.topic: reference
ms.author: doveychase
ms.date: 10/31/2023
ms.service: azure-container-registry
# Customer intent: As a DevOps engineer, I want to access sample YAML files and Dockerfiles for Azure Container Registry Tasks, so that I can efficiently build, run, and manage container images in my development workflow.
---

# ACR Tasks samples

This article links to example `task.yaml` files and associated Dockerfiles for several [Azure Container Registry Tasks](container-registry-tasks-overview.md) (ACR Tasks) scenarios. 

For additional examples, see the [Azure samples][task-examples] repo.

## Scenarios

* **Build image** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-hello-world.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Run container** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo.yaml)

* **Build and push image** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Build and run image** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-run-hello-world.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Build and push multiple images** -  [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world-multi.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Build and test images in parallel** -  [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Build and push images to multiple registries** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/multipleRegistries/testtask.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/multipleRegistries/hello-world.dockerfile)


## Next steps

Learn more about ACR Tasks:

* [Multi-step tasks](container-registry-tasks-multi-step.md) - ACR Task-based workflows for building, testing, and patching container images in the cloud.
* [Task reference](container-registry-tasks-reference-yaml.md) - Task step types, their properties, and usage.
* [Cmd repo](https://github.com/AzureCR/cmd) - A collection of containers as commands for ACR Tasks.


<!-- LINKS - External -->
[task-examples]: https://github.com/Azure-Samples/acr-tasks
