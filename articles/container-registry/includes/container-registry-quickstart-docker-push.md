---
title: include file
description: include file
services: container-registry
author: dlepow

ms.service: azure-container-registry
ms.topic: include
ms.date: 08/04/2020
ms.author: danlep
ms.custom: include file
---
## Push image to registry

To push an image to an Azure Container registry, you must first have an image. If you don't yet have any local container images, run the following [docker pull][docker-pull] command to pull an existing public image. For this example, pull the `hello-world` image from Microsoft Container Registry.

```
docker pull mcr.microsoft.com/hello-world
```

Before you can push an image to your registry, you must tag it with the fully qualified name of your registry login server. The login server name is in the format *\<registry-name\>.azurecr.io* (must be all lowercase), for example, *mycontainerregistry.azurecr.io*.

Tag the image using the [docker tag][docker-tag] command. Replace `<login-server>` with the login server name of your ACR instance.

```
docker tag mcr.microsoft.com/hello-world <login-server>/hello-world:v1
```

Example:

```
docker tag mcr.microsoft.com/hello-world mycontainerregistry.azurecr.io/hello-world:v1
```


Finally, use [docker push][docker-push] to push the image to the registry instance. Replace `<login-server>` with the login server name of your registry instance. This example creates the **hello-world** repository, containing the `hello-world:v1` image.

```
docker push <login-server>/hello-world:v1
```

After pushing the image to your container registry, remove the `hello-world:v1` image from your local Docker environment. (Note that this [docker rmi][docker-rmi] command does not remove the image from the **hello-world** repository in your Azure container registry.)

```
docker rmi <login-server>/hello-world:v1
```
If you don't have a local environment, you can consider creating an Azure VM (https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-cli) and install Docker on it (https://docs.docker.com/engine/install/).

---

## Push image to registry using the Azure Cloud Shell

Alternatively, if you don’t have access to an environment suitable for the push operation or simply prefer using Azure Cloud Shell, you can install ORAS. Keep in mind that this installation of ORAS does not include the Docker daemon, so running containers will not be possible. However, you can still perform pull and copy operations (a functional alternative to "push") seamlessly.

### Step 1: Set the version of `oras`

Set the version of `oras` you want to install. Replace `<version>` with the version number you intend to use. You can check the latest one here: https://github.com/oras-project/oras/releases. For example, version `1.2.0`:

```
VERSION="<version>"
```

For example:

```
VERSION="1.2.0"
```

### Step 2: Download the `oras` binary and extract it

Download the specified version of `oras` from the GitHub releases page. Replace `<version>` with the version you set in the previous step.

```
curl -LO "https://github.com/oras-project/oras/releases/download/v${VERSION}/oras_${VERSION}_linux_amd64.tar.gz"
```

```
mkdir -p oras-install/
```

```
tar -zxf oras_${VERSION}_*.tar.gz -C oras-install/
```

### Step 3: Install `oras` in a rootless environment

Since Azure Cloud Shell does not allow root access, install `oras` in a user-specific directory (`~/bin/`) and add it to the `PATH` so you can use it without `sudo` permissions.

```
mkdir -p ~/bin
```

```
mv oras-install/oras ~/bin/
```

```
echo 'export PATH=$PATH:~/bin' >> ~/.bashrc
```

```
source ~/.bashrc
```

### Step 4: Clean up

Remove the tarball and temporary directory to clean up after the installation:

```
rm -rf oras_${VERSION}_*.tar.gz oras-install/
```

### Step 5: Pull an image from a registry

To pull an image from a registry, use the following command, replacing `<registry>`, `<repository>`, and `<tag>` with the appropriate values. For example, to pull `hello-world` from `mcr.microsoft.com`:

```
oras pull <registry>/<repository>:<tag>
```

For example:

```
oras pull mcr.microsoft.com/hello-world:latest
```

### Step 6: Log in to Azure Container Registry (ACR)

Before copying the image to Azure, authenticate with your Azure Container Registry using the `oras login` command. Replace `<acrname>` with your ACR name. You will be prompted to specify the username and password.

```
oras login <acrname>.azurecr.io
```

### Step 7: Copy the image to your Azure Container Registry

Now, you can copy the pulled image to your Azure Container Registry (ACR). Replace `<repository>`, `<tag>`, and `<acrname>` with the appropriate values. For example:

```
oras copy <registry>/<repository>:<tag> <acrname>.azurecr.io/<repository>:<tag>
```

For example:

```
oras copy mcr.microsoft.com/hello-world:latest acr123.azurecr.io/hello-oras:v1
```

> [!NOTE]
> In case you are wondering why we are using `oras copy` instead of `oras tag` plus `oras push`, it's because `oras tag` does not support cross-registry tagging like Docker. When dealing with different registries, `oras copy` allows transferring images between them directly, while `oras tag` only modifies the local reference without transferring the image.


<!-- LINKS - External -->
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/

<!-- LINKS - Internal -->

