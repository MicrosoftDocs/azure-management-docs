---
title: Manage network bypass policy for tasks
description: This article provides guidance on managing network bypass policy for ACR tasks.
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to 
ms.date: 05/15/2025

# Customer intent: As a developer, I want to configure the network bypass policy for my ACR tasks, so that I can ensure secure access while managing necessary exceptions for trusted services.
---

# Manage network bypass policy for tasks

The ACR tasks `networkRuleBypassAllowedForTasks` setting is a policy setting that allows customers to opt in to network bypass for tasks. As discussed in [ACR trusted services](~/articles/container-registry/allow-access-trusted-services.md), some users require network restricted access to a container registry. This network restriction allows for certain services or identities to bypass network controls based on defined rule access.

ACR users can configure tasks to use System Assigned Managed Identity (SAMI) to authenticate with a container registry. However, when the registry is network isolated, the registry owner can specify Allow trusted Microsoft services to access this container registry. ACR tasks have transitioned from listing as a [trusted service](~/articles/container-registry/allow-access-trusted-services.md). Because registry owners can enable the trusted service setting to continue to allow tasks as a trusted service, the policy setting disables this by default ensuring customers' workflows remain secured.  

As of **June 1, 2025**, if the `networkRuleBypassAllowedForTasks` setting is set to `false`, network bypass for tasks using System-Assigned Managed Identity (SAMI) tokens will be denied by default and will require explicit configuration to restore functionality. Customers who rely on network bypass for their container registry tasks but have not explicitly set the new policy setting will encounter `403 forbidden errors`.

> [!IMPORTANT]
> There is no impact to customers using [User-Assigned Identity](~/articles/container-registry/container-registry-tasks-authentication-managed-identity.md)

To avoid any potential issues, ensure your configurations are updated to use this setting, or alternatively use Agent Pool. Customers who rely on network bypass for their container registry tasks but have not explicitly set the new policy setting will encounter `403 forbidden errors`. Alternatively, you may use the container registry Agent Pool feature to also restrict access. Review [Use Dedicated Pool to Run Tasks in Azure Container Registry](~/articles/container-registry/tasks-agent-pools.md) to configure firewall rules and/or advanced network configuration per your desired requirements.

## Enabling and disabling the network rule bypass policy setting

To enable the network rule bypass policy setting on your registry, run the following command, updating the variables as needed for your scenario.

```azurecli
registry="myregistry"
resourceGroup="myresourcegroup"  
 
az resource update \
--namespace Microsoft.ContainerRegistry \
--resource-type registries \
--name $registry \
--resource-group $resourceGroup \
--api-version 2025-06-01-preview \
--set properties.networkRuleBypassAllowedForTasks=true
```

To disable the network rule bypass policy setting on your registry, run the following command:

```azurecli
registry="myregistry"
resourceGroup="myresourcegroup"
 
az resource update \
--namespace Microsoft.ContainerRegistry \
--resource-type registries \ --name $registry \
--resource-group $resourceGroup \
--api-version 2025-06-01-preview \
--set properties.networkRuleBypassAllowedForTasks=false
```

You can check the status of the setting using the following command:

```azurecli
registry="myregistry"
resourceGroup="myresourcegroup"  

az resource show \  
--namespace Microsoft.ContainerRegistry \  
--resource-type registries \  
--name $registry \  
--resource-group $resourceGroup \  
--api-version 2025-06-01-preview \  
--query properties.networkRuleBypassAllowedForTasks
```

## Customer scenarios

Here are some scenarios which may be appropriate for your use case. Enable these scenarios with either the Azure CLI or ARM templates. The following examples focus on the Azure CLI.  

### Scenario 1: Use Agent Pool

Steps to enable:

Review [Use Dedicated Pool to Run Tasks in Azure Container Registry](~/articles/container-registry/tasks-agent-pools.md) to configure firewall rules and/or advanced network configuration per your desired method.  

Provision a dedicated agent pool:

```azurecli
agentpool="myagentpool"
registry="myregistry"
subnet="/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/<NetworkName>/subnets/<SubnetName>"

az acr agentpool create \
--name $agentpool \
--registry $registry \
--subnet-id $subnet
```

Configure a quick task to run in the agent pool using acr build or automatically triggered task using acr task commands.

```azurecli
image="myimage:mytag"

az acr build \
--registry $registry \
--agent-pool $agentpool \
--image $image \
--file Dockerfile \
https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
```

```azurecli
task="mytask"
schedule="0 21 * * *"

az acr task create \
--name $task \
--agent-pool $agentpool \
--registry $registry \
--file Dockerfile \
--context https://github.com/Azure-Samples/acr-build-helloworld-node.git#main \
--image $image \
--schedule $schedule

az acr task run \
--name $task \
--registry $registry
```

### Scenario 2: Opt in to enable the new network bypass policy setting

```azurecli
registry="myregistry"
resourceGroup="myresourcegroup"  

az resource update \
--namespace Microsoft.ContainerRegistry \
--resource-type registries \
--name $registry \
--resource-group $resourceGroup \
--api-version 2025-06-01-preview \
--set properties.networkRuleBypassAllowedForTasks=true
```

Verify that tasks can continue bypassing network restrictions successfully by running `az acr task run` commands and viewing the [streamed logs](~/articles/container-registry/container-registry-tasks-logs.md).

> [!IMPORTANT]
> When enabling the new network bypass policy for ACR tasks, understand the implications of using a System Assigned Managed Identity (SAMI). This identity allows ACR tasks to authenticate securely without embedding credentials in your workflows. The SAMI token used by ACR tasks is a sensitive credential. If mishandled, such as being written to logs, it could be intercepted and misused.
> **Best practices for safeguarding tokens include**:
> - Avoid outputting the token to logs or exposing it in any way.
> - Implement strict logging hygiene and monitor for accidental token leakage.
> - Regularly audit task definitions and logs for compliance.

### Scenario 3: No action is taken (default behavior) to enable the new network bypass policy setting

If the `networkRuleBypassAllowedForTasks` setting isn't explicitly set, network bypass for tasks is denied by default, resulting in `403 errors` for tasks requiring network bypass.

### Scenario 4: Use [az acr purge](~/articles/container-registry/container-registry-auto-purge.md) locally for image cleanup

If you prefer not to use the policy setting and aren't using network bypass, you can manage ACR cleanup tasks locally using the `az acr purge` command. To do this, download the ACR CLI binary from [Azure ACR CLI GitHub](https://github.com/azure/acr-cli) and execute commands on your own machine. This lets you remove unneeded or stale images from your registry without relying on ACR tasks or altering your current configuration. Running the purge locally ensures all operations occur within your trusted environment (customer managed trust boundary), avoiding any dependency on network bypass.

### Scenario 5: Build and manage images on self-hosted environments

To build container images or manage registries without enabling the setting, use a self-hosted environment. By running Docker or container runtime commands (such as `docker build` and `docker push`) on your own agents or machines that have direct access to the ACR registry, you can perform these tasks securely. This approach eliminates the need for ACR Tasks and/or network bypass, as operations are conducted entirely within your infrastructure, maintaining full control over your workflows (customer manages trust boundary).

## Help and support

For technical help, create a support request in the ⁠[Azure portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

