---
title: Use Dedicated Pool to Run Tasks in Azure Container Registry
description: Set up a dedicated compute pool (agent pool) in your registry to run an Azure Container Registry task.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 10/31/2023
ms.custom: references_regions, devx-track-azurecli
ms.service: azure-container-registry
# Customer intent: As a developer, I want to set up a dedicated compute pool in my Azure Container Registry so that I can run tasks in a scalable and managed environment tailored to my workload needs.
---

# Run an ACR task on a dedicated agent pool

Set up an Azure-managed VM pool (*agent pool*) to enable running your [Azure Container Registry tasks][acr-tasks] in a dedicated compute environment. After you've configured one or more pools in your registry, you can choose a pool to run a task in place of the service's default compute environment.

An agent pool provides:

- **Virtual network support** - Assign an agent pool to an Azure VNet, providing access to resources in the VNet such as a container registry, key vault, or storage.
- **Scale as needed** - Increase the number of instances in an agent pool for compute-intensive tasks, or scale to zero. Billing is based on pool allocation. For details, see [Pricing](https://azure.microsoft.com/pricing/details/container-registry/).
- **Flexible options** - Choose from different [pool tiers](#pool-tiers) and scale options to meet your task workload needs.
- **Azure management** - Task pools are patched and maintained by Azure, providing reserved allocation without the need to maintain the individual VMs.

This feature is available in the **Premium** container registry service tier. For information about registry service tiers and limits, see [Azure Container Registry SKUs][acr-tiers].

> [!IMPORTANT]
> This feature is currently in preview, and some [limitations apply](#preview-limitations). Previews are made available to you on the condition that you agree to the [supplemental terms of use][terms-of-use]. Some aspects of this feature may change prior to general availability (GA).
>

## Preview limitations

- Task agent pools currently support Linux nodes. Windows nodes aren't currently supported.
- Task agent pools are available in preview in the following regions: West US 2, South Central US, East US 2, East US, Central US, West Europe, North Europe, Canada Central, East Asia, Switzerland North, USGov Arizona, USGov Texas, and USGov Virginia.
- For each registry, the default total vCPU (core) quota is 16 for all standard agent pools and is 0 for isolated agent pools. Open a [support request][open-support-ticket] for additional allocation.

## Prerequisites

* To use the Azure CLI steps in this article, Azure CLI version 2.3.1 or later is required. If you need to install or upgrade, see [Install Azure CLI][azure-cli]. Or run in [Azure Cloud Shell](/azure/cloud-shell/quickstart).
* If you don't already have a container registry, [create one][create-reg-cli] (Premium tier required) in a preview region.

## Pool tiers

Agent pool tiers provide the following resources per instance in the pool.

| Tier | Type     | CPU | Memory (GB) |
| ---- | -------- | --- | ----------- |
| S1   | standard | 2   | 3           |
| S2   | standard | 4   | 8           |
| S3   | standard | 8   | 16          |
| I6   | isolated | 64  | 216         |


## Create and manage a task agent pool

### Set default registry (optional)

To simplify Azure CLI commands that follow, set the default registry by running the [az config][az-config] command:

```azurecli
az config set defaults.acr=<registryName>
```

The following examples assume that you've set the default registry. If not, pass a `--registry <registryName>` parameter in each `az acr` command.

### Create agent pool

Create an agent pool by using the [az acr agentpool create][az-acr-agentpool-create] command. The following example creates a tier S2 pool (4 CPU/instance). By default, the pool contains 1 instance.

```azurecli
az acr agentpool create \
    --registry MyRegistry \
    --name myagentpool \
    --tier S2
```

> [!NOTE]
> Creating an agent pool and other pool management operations take several minutes to complete.

### Scale pool

Scale the pool size up or down with the [az acr agentpool update][az-acr-agentpool-update] command. The following example scales the pool to 2 instances. You can scale to 0 instances.

```azurecli
az acr agentpool update \
    --registry MyRegistry \
    --name myagentpool \
    --count 2
```

## Create pool in a virtual network

### Add firewall rules

Task agent pools require access to the following Azure services. The following firewall rules must be added to any existing network security groups or user-defined routes.

| Direction | Protocol | Source         | Source Port | Destination          | Dest Port | Used    | Remarks                                           |
| --------- | -------- | -------------- | ----------- | -------------------- | --------- | ------- | ------------------------------------------------- |
| Outbound  | TCP      | VirtualNetwork | Any         | AzureKeyVault        | 443       | Default |                                                   |
| Outbound  | TCP      | VirtualNetwork | Any         | Storage              | 443       | Default |                                                   |
| Outbound  | TCP      | VirtualNetwork | Any         | EventHub             | 443       | Default |                                                   |
| Outbound  | TCP      | VirtualNetwork | Any         | AzureActiveDirectory | 443       | Default |                                                   |
| Outbound  | TCP      | VirtualNetwork | Any         | AzureMonitor         | 443,12000 | Default | Port 12000 is a unique port used for diagnostics  |

> [!NOTE]
> If your tasks require additional resources from the public internet, add the corresponding rules. For example, additional rules are needed to run a docker build task that pulls the base images from Docker Hub, or restores a NuGet package.

Customers basing their deployments with MCR can refer to [MCR/MAR firewall rules.](https://github.com/microsoft/containerregistry/blob/main/docs/client-firewall-rules.md)

#### Advanced network configuration

If the standard Firewall/NSG (Network Security Group) rules are deemed too permissive, and more fine-grained control is required for outbound connections, consider the following approach:

- Enable service endpoints on the agent pool subnet. This grants the agent pool access to its service dependencies while maintaining a secure network posture.
- It's important to note that outbound Firewall/NSG rules are still necessary. These rules facilitate the Virtual Network's ability to switch the source IP from public to private, which is an additional step beyond enabling service endpoints.
 
More information on service endpoints is documented [here][az-vnet-svc-ep].
 
At minimum, the following service endpoints will be required
 
- Microsoft.AzureActiveDirectory
- Microsoft.ContainerRegistry (only if the registry is not using a [private link](/azure/container-registry/container-registry-vnet))
- Microsoft.EventHub
- Microsoft.KeyVault
- Microsoft.Storage (or the corresponding storage regions taking geo-replication into account)
 
> [!NOTE] 
> Currently a service endpoint for Azure Monitor does not exist. If outbound traffic for Azure Monitor is not configured, the agent pool will be unable to emit diagnostic logs but may appear to still operate normally. In this case ACR will be unable to help fully troubleshoot any issues encountered so it is important that the network administrator take this into account when planning the network configuration.
 
Also, it is important to note that all of ACR Tasks have pre-cached images for some of the more common use cases. Tasks will only cache a single version at a time, meaning that if the full tagged image reference is used, then the build agent will attempt to pull the image. For example, a common use case is `cmd: mcr.microsoft.com/acr/acr-cli:<tag>`. However, the pre-cached version is frequently updated, which means the actual version on the machine will likely be higher. In this case, the network configuration must configure a route for outbound traffic to the target registry host which in the example above would be mcr.microsoft.com. The same rules would apply to any other external public registry (docker.io, quay.io, ghcr.io, etc.).

#### Using custom DNS

If the subnet has a custom DNS server configured, this will be inherited by the build agent during runtime. 

> [!IMPORTANT]
> The IP range used by the custom DNS endpoint must not conflict with docker's default range of `172.17.0.0/16`
>

### Create pool in VNet

The following example creates an agent pool in the *mysubnet* subnet of network *myvnet*:

```azurecli
# Get the subnet ID
subnetId=$(az network vnet subnet show \
        --resource-group myresourcegroup \
        --vnet-name myvnet \
        --name mysubnetname \
        --query id --output tsv)

az acr agentpool create \
    --registry MyRegistry \
    --name myagentpool \
    --tier S2 \
    --subnet-id $subnetId
```

## Run task on agent pool

The following examples show how to specify an agent pool when queuing a task.

> [!NOTE]
> To use an agent pool in an ACR task, ensure that the pool contains at least 1 instance.
>

### Quick task

Queue a quick task on the agent pool by using the [az acr build][az-acr-build] command and pass the `--agent-pool` parameter:

```azurecli
az acr build \
    --registry MyRegistry \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --file Dockerfile \
    https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
```

### Automatically triggered task

For example, create a scheduled task on the agent pool with [az acr task create][az-acr-task-create], passing the `--agent-pool` parameter.

```azurecli
az acr task create \
    --registry MyRegistry \
    --name mytask \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --schedule "0 21 * * *" \
    --file Dockerfile \
    --context https://github.com/Azure-Samples/acr-build-helloworld-node.git#main \
    --commit-trigger-enabled false
```

To verify task setup, run [az acr task run][az-acr-task-run]:

```azurecli
az acr task run \
    --registry MyRegistry \
    --name mytask
```

### Query pool status

To find the number of runs currently scheduled on the agent pool, run [az acr agentpool show][az-acr-agentpool-show].

```azurecli
az acr agentpool show \
    --registry MyRegistry \
    --name myagentpool \
    --queue-count
```

## Next steps

For more examples of container image builds and maintenance in the cloud, check out the [ACR Tasks tutorial series](container-registry-tutorial-quick-task.md).



[acr-tasks]:           container-registry-tasks-overview.md
[acr-tiers]:           container-registry-skus.md
[azure-cli]:           /cli/azure/install-azure-cli
[open-support-ticket]: https://aka.ms/acr/support/create-ticket
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[az-config]: /cli/azure#az_config
[az-acr-agentpool-create]: /cli/azure/acr/agentpool#az_acr_agentpool_create
[az-acr-agentpool-update]: /cli/azure/acr/agentpool#az_acr_agentpool_update
[az-acr-agentpool-show]: /cli/azure/acr/agentpool#az_acr_agentpool_show
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[create-reg-cli]: container-registry-get-started-azure-cli.md
[az-vnet-svc-ep]: /azure/virtual-network/virtual-network-service-endpoints-overview#secure-azure-services-to-virtual-networks
