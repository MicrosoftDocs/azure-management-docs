---
title: Azure Container Registry Authentication Options Explained
description: Authentication options for a private Azure container registry, including signing in with a Microsoft Entra identity, using service principals, and using optional admin credentials.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 10/31/2023
---

# Authenticate with an Azure container registry

There are several ways to authenticate with an Azure container registry, each of which is applicable to one or more registry usage scenarios.

Recommended ways include:

* Authenticate to a registry directly via [individual login](#individual-login-with-azure-ad)
* Applications and container orchestrators can perform unattended, or "headless," authentication by using a Microsoft Entra [service principal](#service-principal)

If you use a container registry with Azure Kubernetes Service (AKS) or another Kubernetes cluster, see [Scenarios to authenticate with Azure Container Registry from Kubernetes](authenticate-kubernetes-options.md).

## Authentication options

The following table lists available authentication methods and typical scenarios. See linked content for details.

| Method                               | How to authenticate                                           | Scenarios                                                            | Microsoft Entra role-based access control (RBAC)                             | Limitations                                |
|---------------------------------------|-------------------------------------------------------|---------------------------------------------------------------------|----------------------------------|--------------------------------------------|
| [Individual Microsoft Entra identity](#individual-login-with-azure-ad)                | `az acr login` in Azure CLI<br/><br/> `Connect-AzContainerRegistry` in Azure PowerShell                             | Interactive push/pull by developers, testers                                    | Yes                              | Microsoft Entra token must be renewed every 3 hours     |
| [Microsoft Entra service principal](#service-principal)                  | `docker login`<br/><br/>`az acr login` in Azure CLI<br/><br/> `Connect-AzContainerRegistry` in Azure PowerShell<br/><br/> Registry login settings in APIs or tooling<br/><br/> [Kubernetes pull secret](container-registry-auth-kubernetes.md)                                           | Unattended push from CI/CD pipeline<br/><br/> Unattended pull to Azure or external services  | Yes                              | SP password default expiry is 1 year       |
| [Microsoft Entra managed identity for Azure resources](container-registry-authentication-managed-identity.md)  | `docker login`<br/><br/> `az acr login` in Azure CLI<br/><br/> `Connect-AzContainerRegistry` in Azure PowerShell                                       | Unattended push from Azure CI/CD pipeline<br/><br/> Unattended pull to Azure services<br/><br/>For a list of managed identity role assignment scenarios, consult the [ACR role assignment scenarios](container-registry-rbac-built-in-roles-overview.md).  | Yes<br/><br/>[Microsoft Entra RBAC role assignments with ACR built-in roles](container-registry-rbac-built-in-roles-overview.md)<br/><br/>[Microsoft Entra attribute-based access control (ABAC) for **Microsoft Entra-based repository permissions**](container-registry-rbac-abac-repository-permissions.md)                              | Use only from select Azure services that [support managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities#azure-services-that-support-managed-identities-for-azure-resources)              |
| [AKS cluster node kubelet managed identity](/azure/aks/cluster-container-registry-integration?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json)                    | Attach registry when AKS cluster created or updated  | Unattended pull to AKS cluster node in the same or a different subscription                                                 | No, pull access only             | Only available with AKS cluster<br/><br/>Can't be used for cross-tenant authentication            |
| [AKS cluster service principal](authenticate-aks-cross-tenant.md)                    | Enable when AKS cluster created or updated  | Unattended pull to AKS cluster from registry in another Entra tenant                                                  | No, pull access only             | Only available with AKS cluster            |
| [Admin user](#admin-account)                            | `docker login`                                          | Interactive push/pull by individual developer or tester<br/><br/>Portal deployment of image from registry to Azure App Service or Azure Container Instances                      | No, always pull and push access  | Single account per registry, not recommended for multiple users         |
| [Non-Microsoft Entra token-based repository permissions](container-registry-token-based-repository-permissions.md)               | `docker login`<br/><br/>`az acr login` in Azure CLI<br/><br/> `Connect-AzContainerRegistry` in Azure PowerShell<br/><br/> [Kubernetes pull secret](container-registry-auth-kubernetes.md)    | Interactive push/pull to repository by individual developer or tester<br/><br/> Unattended pull from repository by individual system or external device                  | Token-based repository permissions **does not** support Microsoft Entra RBAC role assignments.<br/><br/>For Microsoft Entra-based repository permissions, see [Microsoft Entra attribute-based access control (ABAC) for **Microsoft Entra-based repository permissions**](container-registry-rbac-abac-repository-permissions.md) instead.                           | Not currently integrated with Microsoft Entra identity  |

<a name='individual-login-with-azure-ad'></a>

## Individual login with Microsoft Entra ID

### [Azure CLI](#tab/azure-cli)

When working with your registry directly, such as pulling images to and pushing images from a development workstation to a registry you created, authenticate by using your individual Azure identity. Sign in to the [Azure CLI](/cli/azure/install-azure-cli) with [az login](/cli/azure/reference-index#az-login), and then run the [az acr login](/cli/azure/acr#az-acr-login) command:

```azurecli
az login
az acr login --name <acrName>
```

When you log in with `az acr login`, the CLI uses the token created when you executed `az login` to seamlessly authenticate your session with your registry. To complete the authentication flow, the Docker CLI and Docker daemon must be installed and running in your environment. `az acr login` uses the Docker client to set a Microsoft Entra token in the `docker.config` file. Once you've logged in this way, your credentials are cached, and subsequent `docker` commands in your session don't require a username or password.

> [!TIP]
> Also use `az acr login` to authenticate an individual identity when you want to push or pull artifacts other than Docker images to your registry, such as [OCI artifacts](container-registry-manage-artifact.md).

For registry access, the token used by `az acr login` is valid for **3 hours**, so we recommend that you always log in to the registry before running a `docker` command. If your token expires, you can refresh it by using the `az acr login` command again to reauthenticate.

Using `az acr login` with Azure identities provides [Azure role-based access control (RBAC)](/azure/role-based-access-control/role-assignments-portal). For some scenarios, you may want to log in to a registry with your own individual identity in Microsoft Entra ID, or configure other Azure users with specific roles. See [Azure Container Registry Entra permissions and roles overview](container-registry-rbac-built-in-roles-overview.md). For cross-service scenarios or to handle the needs of a workgroup or a development workflow where you don't want to manage individual access, you can also log in with a [managed identity for Azure resources](container-registry-authentication-managed-identity.md).

### az acr login with --expose-token

In some cases, you need to authenticate with `az acr login` when the Docker daemon isn't running in your environment. For example, you might need to run `az acr login` in a script in Azure Cloud Shell, which provides the Docker CLI but doesn't run the Docker daemon.

For this scenario, run `az acr login` first with the `--expose-token` parameter. This option exposes an access token instead of logging in through the Docker CLI.

```azurecli
az acr login --name <acrName> --expose-token
```

Output displays the access token, abbreviated here:

```console
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs[...]24V7wA",
  "loginServer": "myregistry.azurecr.io"
}
```
For registry authentication, we recommend that you store the token credential in a safe location and follow recommended practices to manage [docker login](https://docs.docker.com/engine/reference/commandline/login/) credentials. For example, store the token value in an environment variable:

```azurecli
TOKEN=$(az acr login --name <acrName> --expose-token --output tsv --query accessToken)
```

Then, run `docker login`, passing `00000000-0000-0000-0000-000000000000` as the username and using the access token as password:

```console
docker login myregistry.azurecr.io --username 00000000-0000-0000-0000-000000000000 --password-stdin <<< $TOKEN
```
Likewise, you can use the token returned by `az acr login` with the `helm registry login` command to authenticate with the registry:

```console
echo $TOKEN | helm registry login myregistry.azurecr.io \
            --username 00000000-0000-0000-0000-000000000000 \
            --password-stdin
```

### [Azure PowerShell](#tab/azure-powershell)

When working with your registry directly, such as pulling images to and pushing images from a development workstation to a registry you created, authenticate by using your individual Azure identity. Sign in to [Azure PowerShell](/powershell/azure/uninstall-az-ps) with [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount), and then run the [Connect-AzContainerRegistry](/powershell/module/az.containerregistry/connect-azcontainerregistry) cmdlet:

```azurepowershell
Connect-AzAccount
Connect-AzContainerRegistry -Name <acrName>
```

When you log in with `Connect-AzContainerRegistry`, PowerShell uses the token created when you executed `Connect-AzAccount` to seamlessly authenticate your session with your registry. To complete the authentication flow, the Docker CLI and Docker daemon must be installed and running in your environment. `Connect-AzContainerRegistry` uses the Docker client to set a Microsoft Entra token in the `docker.config` file. Once you've logged in this way, your credentials are cached, and subsequent `docker` commands in your session don't require a username or password.

> [!TIP]
> Also use `Connect-AzContainerRegistry` to authenticate an individual identity when you want to push or pull artifacts other than Docker images to your registry, such as [OCI artifacts](container-registry-manage-artifact.md).

For registry access, the token used by `Connect-AzContainerRegistry` is valid for **3 hours**, so we recommend that you always log in to the registry before running a `docker` command. If your token expires, you can refresh it by using the `Connect-AzContainerRegistry` command again to reauthenticate.

Using `Connect-AzContainerRegistry` with Azure identities provides [Azure role-based access control (RBAC)](/azure/role-based-access-control/role-assignments-portal). For some scenarios, you may want to log in to a registry with your own individual identity in Microsoft Entra ID, or configure other Azure users with specific roles. See [Azure Container Registry Entra permissions and roles overview](container-registry-rbac-built-in-roles-overview.md). For cross-service scenarios or to handle the needs of a workgroup or a development workflow where you don't want to manage individual access, you can also log in with a [managed identity for Azure resources](container-registry-authentication-managed-identity.md).

---

## Service principal

If you assign a [service principal](/azure/active-directory/develop/app-objects-and-service-principals) to your registry, your application or service can use it for headless authentication. Service principals allow [Azure role-based access control (RBAC)](/azure/role-based-access-control/role-assignments-portal) to a registry, and you can assign multiple service principals to a registry. Multiple service principals allow you to define different access for different applications.

ACR authentication token gets created upon login to the ACR, and is refreshed upon subsequent operations. The time to live for that token is 3 hours.

For a list of available roles, please see [Azure Container Registry Entra permissions and roles overview](container-registry-rbac-built-in-roles-overview.md).

For CLI scripts to create a service principal for authenticating with an Azure container registry, and more guidance, see [Azure Container Registry authentication with service principals](container-registry-auth-service-principal.md).

## Admin account

Each container registry includes an admin user account, which is disabled by default. You can enable the admin user and manage its credentials in the Azure portal, or by using the Azure CLI, Azure PowerShell, or other Azure tools. The admin account has full permissions to the registry.

The admin account is currently required for some scenarios to deploy an image from a container registry to certain Azure services. For example, the admin account is needed when you use the Azure portal to deploy a container image from a registry directly to [Azure Container Instances](/azure/container-instances/container-instances-using-azure-container-registry#deploy-with-azure-portal) or [Azure Web Apps for Containers](container-registry-tutorial-deploy-app.md).

> [!IMPORTANT]
> The admin account is designed for a single user to access the registry, mainly for testing purposes. We don't recommend sharing the admin account credentials among multiple users. All users authenticating with the admin account appear as a single user with push and pull access to the registry. Changing or disabling this account disables registry access for all users who use its credentials. Individual identity is recommended for users and service principals for headless scenarios.
>

The admin account is provided with two passwords, both of which can be regenerated. New passwords created for admin accounts are available immediately. Regenerating passwords for admin accounts will take 60 seconds to replicate and be available. Two passwords allow you to maintain connection to the registry by using one password while you regenerate the other. If the admin account is enabled, you can pass the username and either password to the `docker login` command when prompted for basic authentication to the registry. For example:

```
docker login myregistry.azurecr.io
```

For recommended practices to manage login credentials, see the [docker login](https://docs.docker.com/engine/reference/commandline/login/) command reference.

### [Azure CLI](#tab/azure-cli)

To enable the admin user for an existing registry, you can use the `--admin-enabled` parameter of the [az acr update](/cli/azure/acr#az-acr-update) command in the Azure CLI:

```azurecli
az acr update -n <acrName> --admin-enabled true
```

### [Azure PowerShell](#tab/azure-powershell)

To enable the admin user for an existing registry, you can use the `EnableAdminUser` parameter of the [Update-AzContainerRegistry](/powershell/module/az.containerregistry/update-azcontainerregistry) command in Azure PowerShell:

```azurepowershell
Update-AzContainerRegistry -Name <acrName> -ResourceGroupName myResourceGroup -EnableAdminUser
```

---

You can enable the admin user in the Azure portal by navigating your registry, selecting **Access keys** under **SETTINGS**, then **Enable** under **Admin user**.

![Enable admin user UI in the Azure portal][auth-portal-01]

## Log in with an alternative container tool instead of Docker
In some scenarios, you need to use alternative container tools like `podman` instead of the common container tool `docker`. For example: [Docker is no longer available in RHEL 8 and 9][docker-deprecated-redhat-8-9], so you have to switch your container tool.  

The default container tool is set to `docker` for `az acr login` commands. If you don't set the default container tool and the `docker` command is missing in your environment, the following error will be popped:
```bash
az acr login --name <acrName>
2024-03-29 07:30:10.014426 An error occurred: DOCKER_COMMAND_ERROR
Please verify if Docker client is installed and running.
```

To change the default container tool that the `az acr login` command uses, you can set the environment variable `DOCKER_COMMAND`. For example: 
```azurecli
DOCKER_COMMAND=podman \
az acr login --name <acrName>
```

> [!NOTE]
> You need the Azure CLI version 2.59.0 or later installed and configured to use this feature. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][install-azure-cli].

## Next steps

* [Push your first image using the Azure CLI](container-registry-get-started-azure-cli.md)

* [Push your first image using Azure PowerShell](container-registry-get-started-powershell.md)

<!-- IMAGES -->
[auth-portal-01]: ./media/container-registry-authentication/auth-portal-01.png

<!-- EXTERNAL LINKS -->
[docker-deprecated-redhat-8-9]: https://access.redhat.com/solutions/3696691

<!-- INTERNAL LINKS -->
[install-azure-cli]: /cli/azure/install-azure-cli
