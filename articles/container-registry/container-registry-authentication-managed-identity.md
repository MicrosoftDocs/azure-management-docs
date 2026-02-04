---
title: Managed Identity Authentication for ACR
description: Provide access to images in your Azure container registry by using a user-assigned or system-assigned managed Azure identity.
ms.topic: how-to
ms.custom: devx-track-azurecli, devx-track-azurepowershell, linux-related-content
author: rayoef
ms.service: azure-container-registry
ms.author: rayoflores
ms.date: 02/03/2026

# Customer intent: As a developer, I want to configure managed identities for authenticating to Azure Container Registry, so that I can securely access images without having to manage sensitive credentials.
---

# Use a managed identity to authenticate to an Azure container registry

You can use a [managed identity for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) to authenticate to an Azure container registry from another Azure resource, without needing to provide or manage registry credentials.

For example, set up a [user-assigned or system-assigned managed identity](/entra/identity/managed-identities-azure-resources/overview#managed-identity-types) on a Linux VM to access container images from your container registry, as easily as you use a public registry. Or, set up an Azure Kubernetes Service cluster to use its [managed identity](/azure/aks/cluster-container-registry-integration) to pull container images from Azure Container Registry for pod deployments.

For an overview of managed identities in Azure, see [What is managed identities for Azure resources?](/azure/active-directory/managed-identities-azure-resources/overview)

You can assign a managed identity a role with pull, push and pull, or other permissions to one or more private registries in Azure. For a complete list of registry roles, see [Azure Container Registry permissions and roles overview](container-registry-rbac-built-in-roles-overview.md).

Then, use the identity to authenticate to any [service that supports managed identities](/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities), without requiring any credentials in your code.

In this article, you learn how to:

> [!div class="checklist"]
> * Enable a user-assigned or system-assigned identity on an Azure virtual machine (VM)
> * Grant the identity access to an Azure container registry
> * Use the managed identity to access the registry and pull a container image

## Prerequisites

You can use either Azure CLI or Azure PowerShell to complete the steps in this article. Use the most recent version of either tool. If you don't have either tool installed, see [Install the Azure CLI][azure-cli-install] or [Install Azure PowerShell][azure-powershell-install].

To set up a container registry and push a container image to it, you must also have Docker installed locally. Docker provides packages that easily configure Docker on any [macOS][docker-mac], [Windows][docker-windows], or [Linux][docker-linux] system.

If you don't already have an Azure container registry, create a registry and push a sample container image to it. Follow the steps to create a registry by using [the Azure CLI](container-registry-get-started-azure-cli.md), [Azure PowerShell](container-registry-get-started-powershell.md), or the [Azure portal](container-registry-get-started-portal.md).

This article assumes you have the `aci-helloworld:v1` container image stored in your registry. The examples use a registry name of *myContainerRegistry*. You can replace these values with your own registry and image names.

## Create and configure a Docker-enabled VM

Create a Docker-enabled Ubuntu VM to use with your managed identity. You also need to install either the Azure CLI or Azure PowerShell on the virtual machine, depending on which tool you want to use.

If you already have an Azure VM with Azure CLI or Azure PowerShell installed, you can move to the next section.

### Create the VM

### [Azure CLI](#tab/azure-cli)

Deploy a default Ubuntu Azure virtual machine by using [`az vm create`][az-vm-create]. The following example creates a VM named *myDockerVM* in an existing resource group named *myResourceGroup*:

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myDockerVM \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys
```

It takes a few minutes to create the VM. When the command finishes, note the `publicIpAddress` displayed by the Azure CLI. Use this address to make SSH connections to the VM in the next step.

### [Azure PowerShell](#tab/azure-powershell)

Deploy a default Ubuntu Azure virtual machine by using [`New-AzVM`][new-azvm]. The following example creates a VM named *myDockerVM* in an existing resource group named *myResourceGroup*. You're prompted for a user name that you use when you connect to the VM. Specify *azureuser* as the user name. You're also asked for a password, which you can leave blank. Password authentication for the VM is disabled when using an SSH key.

```azurepowershell-interactive
$vmParams = @{
    ResourceGroupName   = 'MyResourceGroup'
    Name                = 'myDockerVM'
    Image               = 'UbuntuLTS'
    PublicIpAddressName = 'myPublicIP'
    GenerateSshKey      = $true
    SshKeyName          = 'mySSHKey'
}
New-AzVM @vmParams
```

It takes a few minutes to create the VM. When the command finishes, run the following command to get the public IP address. Use this address to make SSH connections to the VM in the next step.

```azurepowershell-interactive
Get-AzPublicIpAddress -Name myPublicIP -ResourceGroupName myResourceGroup | Select-Object -ExpandProperty IpAddress
```

---

### Install Docker on the VM

Next, install Docker on your VM so that it can pull and run container images from your Azure Container Registry.

Once the VM is running, make an SSH connection to the VM. Replace *publicIpAddress* with the public IP address of your VM.

```bash
ssh azureuser@publicIpAddress
```

Run the following command to install Docker on the VM:

```bash
sudo apt update
sudo apt install docker.io -y
```

After installation, run the following command to verify that Docker is running properly on the VM:

```bash
sudo docker run -it mcr.microsoft.com/hello-world
```

```output
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

### Install Azure CLI or Azure PowerShell on the VM

### [Azure CLI](#tab/azure-cli)

Follow the steps in [Install Azure CLI with apt](/cli/azure/install-azure-cli-linux?pivots=apt) to install the Azure CLI on your Ubuntu virtual machine. For this article, be sure to install the most recent version.

After installing the Azure CLI, exit the SSH session.

### [Azure PowerShell](#tab/azure-powershell)

Follow the steps in [Installing PowerShell on Ubuntu][powershell-install] and [Install the Azure Az PowerShell module][azure-powershell-install] to install PowerShell and Azure PowerShell on your Ubuntu virtual machine. For this article, be sure to install Azure PowerShell version 7.5.0 or later.

Depending on your VM's setup, you might need to use `sudo` to run the Azure PowerShell commands and docker commands in this article.

After installing Azure PowerShell, exit the SSH session.

---

## Configure the VM with a user-assigned managed identity

A [user-assigned managed identity](/entra/identity/managed-identities-azure-resources/overview#managed-identity-types) is a standalone Azure resource that you manage separately from the resources that use it. You can associate a user-assigned managed identity with multiple Azure resources.

This section explains how to configure your VM with a user-assigned identity to securely access your Azure Container Registry.

### Create a user-assigned managed identity

### [Azure CLI](#tab/azure-cli)

Create an identity in your subscription by using the [az identity create][az-identity-create] command. You can use the same resource group you used previously to create the container registry or virtual machine, or choose a different one.

```azurecli-interactive
az identity create --resource-group myResourceGroup --name myACRId
```

To configure the identity in the following steps, use the [az identity show][az-identity-show] command to store the identity's resource ID and service principal ID in variables.

```azurecli-interactive
# Get resource ID of the user-assigned identity
userID=$(az identity show --resource-group myResourceGroup --name myACRId --query id --output tsv)

# Get service principal ID of the user-assigned identity
spID=$(az identity show --resource-group myResourceGroup --name myACRId --query principalId --output tsv)
```

Because you need the identity's ID in a later step when you sign in to the CLI from your virtual machine, show the value:

```bash
echo $userID
```

The ID is of the form:

```output
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACRId
```

### [Azure PowerShell](#tab/azure-powershell)

Create an identity in your subscription by using the [New-AzUserAssignedIdentity][new-azuserassignedidentity] cmdlet. You can use the same resource group you used previously to create the container registry or virtual machine, or choose a different one.

```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName myResourceGroup -Location eastus -Name myACRId
```

To configure the identity in the following steps, use the [Get-AzUserAssignedIdentity][get-azuserassignedidentity] cmdlet to store the identity's resource ID and service principal ID in variables.

```azurepowershell-interactive
# Get resource ID of the user-assigned identity
$userID = (Get-AzUserAssignedIdentity -ResourceGroupName myResourceGroup -Name myACRId).Id

# Get service principal ID of the user-assigned identity
$spID = (Get-AzUserAssignedIdentity -ResourceGroupName myResourceGroup -Name myACRId).PrincipalId
```

Because you need the identity's ID in a later step when you sign in to the Azure PowerShell from your virtual machine, show the value:

```azurepowershell-interactive
$userID
```

The ID is of the form:

```output
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACRId
```

---

### Configure the VM with the user-assigned managed identity

Next, configure your Docker VM to use the user-assigned identity you created in the previous step.

### [Azure CLI](#tab/azure-cli)

Use [`az vm identity assign`][az-vm-identity-assign] to configure the VM, using the ID you retrieved in the previous step:

```azurecli-interactive
az vm identity assign --resource-group myResourceGroup --name myDockerVM --identities $userID
```

### [Azure PowerShell](#tab/azure-powershell)

Use [`Update-AzVM`][update-azvm] to configure the VM, using the ID you retrieved in the previous step:

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myDockerVM
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -IdentityType UserAssigned -IdentityID $userID
```

---

### Grant the user-assigned managed identity access to the container registry

Now configure the identity so that it has access to your container registry.

You must assign either `Container Registry Repository Reader` (for [ABAC-enabled registries](container-registry-rbac-abac-repository-permissions.md)) or `AcrPull` (for non-ABAC registries). This role assignment provides [pull permissions](container-registry-rbac-built-in-roles-overview.md) to the registry.

To provide both pull and push permissions, assign either the `Container Registry Repository Writer` role for ABAC-enabled registries, or the `AcrPush` role for non-ABAC-enabled registries.

### [Azure CLI](#tab/azure-cli)

Use [`az acr show`][az-acr-show] to get the resource ID of the registry:

```azurecli-interactive
resourceID=$(az acr show --resource-group myResourceGroup --name myContainerRegistry --query id --output tsv)
```

Use [`az role assignment create`][az-role-assignment-create] to assign the desired role to the identity. This example assigns the `Container Registry Repository Reader` role, which grants pull permissions only, on an ABAC-enabled registry:

```azurecli-interactive
az role assignment create --assignee $spID --scope $resourceID \
    --role "Container Registry Repository Reader" # For ABAC-enabled registries. Otherwise, use AcrPull for non-ABAC registries.
```

### [Azure PowerShell](#tab/azure-powershell)

Use [`Get-AzContainerRegistry`][get-azcontainerregistry] to get the resource ID of the registry:

```azurepowershell-interactive
$resourceID = (Get-AzContainerRegistry -ResourceGroupName myResourceGroup -Name myContainerRegistry).Id
```

Use [`New-AzRoleAssignment`][new-azroleassignment] to assign the desired role to the identity. This example assigns the `Container Registry Repository Reader` role, which grants pull permissions only, on an ABAC-enabled registry:

```azurepowershell-interactive
New-AzRoleAssignment -ObjectId $spID -Scope $resourceID -RoleDefinitionName "Container Registry Repository Reader"
```

---

### Use the user-assigned managed identity to access the registry

SSH into the Docker virtual machine that the identity configures, and then run the following commands to enable the identity to access your container registry.

### [Azure CLI](#tab/azure-cli)

On the VM, authenticate to the Azure CLI by using [az login][az-login], using the identity you retrieved earlier.

```azurecli-interactive
az login --identity --username <userID>
```

Next, authenticate to the registry by using [az acr login][az-acr-login]. When you use this command, the CLI uses the Active Directory token created when you ran `az login` to seamlessly authenticate your session with the container registry. 

```azurecli-interactive
az acr login --name myContainerRegistry
```

You should see a `Login succeeded` message. You can then run `docker` commands without providing credentials. For example, run [`docker pull`][docker-pull] to pull the `aci-helloworld:v1` image, specifying the login server name of your registry:

```bash
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

### [Azure PowerShell](#tab/azure-powershell)

On the VM, authenticate to Azure PowerShell by using [`Connect-AzAccount`][connect-azaccount], using the ID you retrieved earlier. For `-AccountId`, specify a client ID of the identity.

```azurepowershell-interactive
$clientId = (Get-AzUserAssignedIdentity -ResourceGroupName myResourceGroup -Name myACRId).ClientId
Connect-AzAccount -Identity -AccountId $clientId
```

Next, authenticate to the registry by using [`Connect-AzContainerRegistry`][connect-azcontainerregistry]. When you use this command, Azure PowerShell uses the Active Directory token created when you ran `Connect-AzAccount` to seamlessly authenticate your session with the container registry.

```azurepowershell-interactive
Connect-AzContainerRegistry -Name myContainerRegistry
```

You should see a `Login succeeded` message. You can then run `docker` commands without providing credentials. For example, run [`docker pull`][docker-pull] to pull the `aci-helloworld:v1` image, specifying the login server name of your registry:

```bash
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

---

## Configure the VM with a system-assigned managed identity

A [system-assigned managed identity](/entra/identity/managed-identities-azure-resources/overview#managed-identity-types) is a feature of Azure that allows your virtual machine to automatically manage its own identity in Azure Active Directory. This section explains how to configure your VM with a system-assigned identity to securely access your Azure Container Registry.

### Enable the system-assigned managed identity on your VM

### [Azure CLI](#tab/azure-cli)

Use [`az vm identity assign`][az-vm-identity-assign] to configure your Docker VM with a system-assigned identity:

```azurecli-interactive
az vm identity assign --resource-group myResourceGroup --name myDockerVM
```

Use [`az vm show`][az-vm-show] to set a variable to the value of `principalId` (the service principal ID) of the VM's identity, to use in later steps:

```azurecli-interactive
spID=$(az vm show --resource-group myResourceGroup --name myDockerVM --query identity.principalId --out tsv)
```

### [Azure PowerShell](#tab/azure-powershell)

Use [`Update-AzVM`][update-azvm] to configure your Docker VM with a system-assigned identity:

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myDockerVM
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -IdentityType SystemAssigned
```

Use [`Get-AzVM`][get-azvm] to set a variable to the value of `principalId` (the service principal ID) of the VM's identity, to use in later steps:

```azurepowershell-interactive
$spID = (Get-AzVM -ResourceGroupName myResourceGroup -Name myDockerVM).Identity.PrincipalId
```

---

### Grant the system-assigned managed identity access to the container registry

Now configure the identity so that it has access to your container registry.

You must assign either `Container Registry Repository Reader` (for [ABAC-enabled registries](container-registry-rbac-abac-repository-permissions.md)) or `AcrPull` (for non-ABAC registries). This role assignment provides [pull permissions](container-registry-rbac-built-in-roles-overview.md) to the registry.

To provide both pull and push permissions, assign either the `Container Registry Repository Writer` role for ABAC-enabled registries, or the `AcrPush` role for non-ABAC-enabled registries.

### [Azure CLI](#tab/azure-cli)

Use [`az acr show`][az-acr-show] to get the resource ID of the registry:

```azurecli-interactive
resourceID=$(az acr show --resource-group myResourceGroup --name myContainerRegistry --query id --output tsv)
```

Use [`az role assignment create`][az-role-assignment-create] to assign the desired role to the identity. This example assigns the `Container Registry Repository Reader` role, which grants pull permissions only, on an ABAC-enabled registry:

```azurecli-interactive
az role assignment create --assignee $spID --scope $resourceID \
    --role "Container Registry Repository Reader"
```

### [Azure PowerShell](#tab/azure-powershell)

Use [`Get-AzContainerRegistry`][get-azcontainerregistry] to get the resource ID of the registry:

```azurepowershell-interactive
$resourceID = (Get-AzContainerRegistry -ResourceGroupName myResourceGroup -Name myContainerRegistry).Id
```

Use [`New-AzRoleAssignment`][new-azroleassignment] to assign the correct role to the identity. This example assigns the `Container Registry Repository Reader` role, which grants pull permissions only, on an ABAC-enabled registry:

```azurepowershell-interactive
New-AzRoleAssignment -ObjectId $spID -Scope $resourceID -RoleDefinitionName "Container Registry Repository Reader"
```

---

### Use the system-assigned managed identity to access the registry

SSH into the Docker virtual machine that the identity configures, and then run the following commands to enable the identity to access your container registry.

### [Azure CLI](#tab/azure-cli)

On the VM, authenticate the Azure CLI by using [`az login`][az-login], using the system-assigned identity.

```azurecli-interactive
az login --identity
```

Next, authenticate to the registry by using [`az acr login`][az-acr-login]. When you use this command, the CLI uses the Active Directory token created when you ran `az login` to seamlessly authenticate your session with the container registry.

```azurecli-interactive
az acr login --name myContainerRegistry
```

You should see a `Login succeeded` message. You can then run `docker` commands without providing credentials. For example, run [`docker pull`][docker-pull] to pull the `aci-helloworld:v1` image, specifying the login server name of your registry:

```bash
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

### [Azure PowerShell](#tab/azure-powershell)

On the VM, authenticate the Azure PowerShell by using [`Connect-AzAccount`][connect-azaccount], using the system-assigned identity.

```azurepowershell-interactive
Connect-AzAccount -Identity
```

Next, authenticate to the registry by using [`Connect-AzContainerRegistry`][connect-azcontainerregistry]. When you use this command, the PowerShell uses the Active Directory token created when you ran `Connect-AzAccount` to seamlessly authenticate your session with the container registry.

```azurepowershell-interactive
Connect-AzContainerRegistry -Name myContainerRegistry
```

You should see a `Login succeeded` message. You can then run `docker` commands without providing credentials. For example, run [`docker pull`][docker-pull] to pull the `aci-helloworld:v1` image, specifying the login server name of your registry:

```bash
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

---

## Related content

* Learn more about [managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/).
* Learn how to use a [system-assigned](https://github.com/Azure/app-service-linux-docs/blob/master/HowTo/use_system-assigned_managed_identities.md) or [user-assigned](https://github.com/Azure/app-service-linux-docs/blob/master/HowTo/use_user-assigned_managed_identities.md) managed identity with App Service and Azure Container Registry.
* Learn how to [deploy a container image from Azure Container Registry using a managed identity](/azure/container-instances/using-azure-container-registry-mi).

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/install
[docker-mac]: https://docs.docker.com/desktop/setup/install/mac-install/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-windows]: https://docs.docker.com/desktop/setup/install/windows-install/

<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az-login
[connect-azaccount]: /powershell/module/az.accounts/connect-azaccount
[az-acr-login]: /cli/azure/acr#az-acr-login
[connect-azcontainerregistry]: /powershell/module/az.containerregistry/connect-azcontainerregistry
[az-acr-show]: /cli/azure/acr#az-acr-show
[get-azcontainerregistry]: /powershell/module/az.containerregistry/get-azcontainerregistry
[az-vm-create]: /cli/azure/vm#az-vm-create
[new-azvm]: /powershell/module/az.compute/new-azvm
[az-vm-show]: /cli/azure/vm#az-vm-show
[get-azvm]: /powershell/module/az.compute/get-azvm
[az-identity-create]: /cli/azure/identity#az-identity-create
[new-azuserassignedidentity]: /powershell/module/az.managedserviceidentity/new-azuserassignedidentity
[az-vm-identity-assign]: /cli/azure/vm/identity#az-vm-identity-assign
[update-azvm]: /powershell/module/az.compute/update-azvm
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[new-azroleassignment]: /powershell/module/az.resources/new-azroleassignment
[az-identity-show]: /cli/azure/identity#az-identity-show
[get-azuserassignedidentity]: /powershell/module/az.managedserviceidentity/get-azuserassignedidentity
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-powershell-install]: /powershell/azure/install-az-ps
[powershell-install]: /powershell/scripting/install/install-ubuntu
