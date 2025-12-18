---
title: View Repositories intthe Azure Portal
description: Use the Azure portal to view Azure Container Registry repositories, which host Docker container images and other supported artifacts.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 12/18/2025
ms.service: azure-container-registry
# Customer intent: As a developer managing container images, I want to view repositories in the Azure portal, so that I can organize and track the versions of my Docker container images effectively.
---

# View container registry repositories in the Azure portal

Azure Container Registry allows you to store Docker container images and other artifacts in  [repositories](container-registry-concepts.md#repository). When you push images to repositories, you can see a list of the repositories hosting your images, as well as any image tags, in the Azure portal.

To view a repository and its image tags in the Azure portal, follow these steps:

1. In the [Azure portal](https://portal.azure.com), go to the container registry where you pushed the Nginx image.
1. Under **Services**, select **Repositories** to see a list of the repositories that contain the images in the registry.
1. Select a repository to see the image tags within that repository.

For example, if you followed the steps in [Push and pull an image](container-registry-get-started-docker-cli.md) and didn't subsequently delete the image, you should have an Nginx image in your container registry. The instructions in that article specify that you tag the `nginx` image with a `samples` namespace tag, resulting  in `/samples/nginx`. In this case, you see something similar to the following image:

:::image type="content" source="media/container-registry-repositories/container-registry-repositories.png" alt-text="Screenshot showing a repository and tags for a container registry in the Azure portal." lightbox="media/container-registry-repositories/container-registry-repositories.png":::

## Related content

- Learn more about [registries, repositories, and artifacts](container-registry-concepts.md).
- Understand [best practices for repository namespaces](container-registry-best-practices.md#repository-namespaces).