---
author: chasedmicrosoft
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: include
ms.date: 05/20/2025
ms.author: doveychase
# Customer intent: "As a cloud administrator, I want to import container images into my cloud registry using command-line tools, so that I can efficiently manage my container resources and streamline my deployment process."
---
## Import images to your cloud registry

Import the following container images to your cloud registry using the [az acr import](/cli/azure/acr#az-acr-import) command. Skip this step if you already imported these images.

### Connected registry image

Use the [az acr import](/cli/azure/acr#az-acr-import) command to import the connected registry image into your private registry. 

```azurecli
# Use the REGISTRY_NAME variable in the following Azure CLI commands to identify the registry
REGISTRY_NAME=<container-registry-name>

az acr import \
  --name $REGISTRY_NAME \
  --source mcr.microsoft.com/acr/connected-registry:1.0.0
```

### Hello-world image

For testing the connected registry, import the `hello-world` image. This repository will be synchronized to the connected registry and pulled by the connected registry clients.

```azurecli
az acr import \
  --name $REGISTRY_NAME \
  --source mcr.microsoft.com/hello-world:1.1.2
```
