---
title: "Tutorial: Migrate Nodes to Azure Linux"
description: In this Azure Linux tutorial, you learn how to migrate your existing nodes to Azure Linux by removing existing node pools and adding new Azure Linux node pools, or by performing an in-place OS SKU migration.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Migrate nodes to Azure Linux

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321934)

In this tutorial, part _three of five_, you migrate your existing nodes to Azure Linux. You can migrate your existing nodes to Azure Linux using one of the following methods:

- Remove existing node pools and add new Azure Linux node pools.
- Perform an in-place operating system (OS) SKU migration.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).

If you don't have any existing nodes to migrate to Azure Linux, skip to the [next tutorial](./tutorial-monitor-azure-linux-aks.md). In later tutorials, you learn how to enable telemetry and monitoring in your clusters and upgrade Azure Linux nodes.

## Prerequisites

- In previous tutorials, you created and deployed an Azure Linux Container Host for AKS cluster. To complete this tutorial, you need to add an Azure Linux node pool to your existing cluster. If you haven't done this step and would like to follow along, start with [Tutorial 2: Add an Azure Linux node pool to your existing AKS cluster](./tutorial-add-azure-linux-node-pool.md).

> [!NOTE]
> When adding a new Azure Linux node pool, you need to add at least one as `--mode System`. Otherwise, AKS won't allow you to delete your existing node pool.

- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Set environment variables

Set the following environment variables to create unique resource names for each deployment. Replace the placeholder `<your-node-pool-name>` with a name of your choice. You can optionally append a random suffix to ensure uniqueness. The name of a node pool must start with a lowercase letter and can only contain alphanumeric characters. For Linux node pools the length must be between one and 12 characters.

```bash
# Set random suffix for uniqueness
export RANDOM_SUFFIX=$(openssl rand -hex 3)

# Set node pool name
export NODE_POOL_NAME="<your-node-pool-name>$RANDOM_SUFFIX"
```

## Add Azure Linux node pools and remove existing node pools

1. Add a new Azure Linux node pool using the [`az aks nodepool add`](/cli/azure/aks/nodepool#az_aks_nodepool_add) command. This command adds a new node pool to your cluster with the `--mode System` flag, which makes it a system node pool. System node pools are required for Azure Linux clusters.

    ```azurecli-interactive
    az aks nodepool add --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODE_POOL_NAME --mode System --os-sku AzureLinux
    ```

    Example output:

    <!-- expected_similarity=0.3 -->

    ```JSON
    {
      "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxx/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/nodePools/systempool",
      "name": "systempool",
      "provisioningState": "Succeeded"
    }
    ```

1. Remove your existing nodes using the [`az aks nodepool delete`](/cli/azure/aks/nodepool#az_aks_nodepool_delete) command.

## In-place OS SKU migration

You can migrate your existing Ubuntu node pools to Azure Linux by changing the OS SKU of the node pool, which rolls the cluster through the standard node image upgrade process. This new feature doesn't require the creation of new node pools.

### In-place OS SKU migration limitations

There are several settings that can block the OS SKU migration request. To ensure a successful migration, review the following guidelines and limitations:

- The OS SKU migration feature isn't available through PowerShell or the Azure portal.
- The OS SKU migration feature isn't able to rename existing node pools.
- Ubuntu, Azure Linux, and Azure Linux with OS Guard are the only supported Linux OS SKU migration targets.
- Trusted Launch is required by default for Azure Linux with OS Guard. You need to have Trusted Launch enabled to be able to migrate to Azure Linux with OS Guard. Since you can't enable Trusted Launch on existing node pools, you need to create a new node pool with Trusted Launch enabled and migrate your workloads to that node pool.
- Customers using Gen 1 only virtual machine (VM) sizes can't migrate to Azure Linux with OS Guard since there's no supported Gen 1 image. In this case, you need to create new node pools with a VM size that supports Gen 2.
- An Ubuntu OS SKU with `UseGPUDedicatedVHD` enabled can't perform an OS SKU migration.
- An Ubuntu OS SKU with CVM 20.04 enabled can't perform an OS SKU migration.
- Node pools with Kata enabled can't perform an OS SKU migration.
- Windows OS SKU migration isn't supported.

### In-place OS SKU migration prerequisites

- An existing AKS cluster with at least one Ubuntu node pool.
- We recommend that you ensure your workloads configure and run successfully on the Azure Linux container host before attempting to use the OS SKU migration feature by [deploying an Azure Linux cluster](./quick-create-azure-linux-aks.md) in dev/prod and verifying your service remains healthy.
- Ensure the migration feature is working for you in test/dev before using the process on a production cluster.
- Ensure that your pods have enough [Pod Disruption Budget (PDB)](/azure/aks/operator-best-practices-scheduler#plan-for-availability-using-pod-disruption-budgets) to allow AKS to move pods between VMs during the upgrade.
- You need Azure CLI version [2.61.0](/cli/azure/release-notes-azure-cli#may-21-2024) or higher. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).
- If you're using Terraform, you must have [v3.111.0](https://github.com/hashicorp/terraform-provider-azurerm/releases/tag/v3.111.0) or greater of the AzureRM Terraform module.

### [Azure CLI](#tab/azure-cli)

#### Migrate the OS SKU of your Ubuntu node pool

Migrate the OS SKU of your node pool to Azure Linux using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az_aks_nodepool_update) command. This command updates the OS SKU for your node pool from Ubuntu to Azure Linux. The OS SKU change triggers an immediate upgrade operation, which takes several minutes to complete.

```azurecli-interactive
az aks nodepool update --resource-group $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --name $NODE_POOL_NAME --os-sku AzureLinux
```

Example output:

<!-- expected_similarity=0.3 -->

```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxx/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/nodePools/nodepool1",
  "name": "nodepool1",
  "osSku": "AzureLinux",
  "provisioningState": "Succeeded"
}
```

> [!NOTE]
> If you experience issues during the OS SKU migration, you can [roll back to your previous OS SKU](#roll-back-to-your-previous-os-sku).

### [ARM template](#tab/arm-template)

#### Example ARM templates

##### 0base.json

```json
 {
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerService/managedClusters",
      "apiVersion": "2023-07-01",
      "name": "akstestcluster",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayname": "Demo of AKS Nodepool Migration"
      },
      "identity": {
        "type": "SystemAssigned"
      },
      "properties": {
        "enableRBAC": true,
        "dnsPrefix": "testcluster",
        "agentPoolProfiles": [
          {
            "name": "testnp",
            "count": 3,
            "vmSize": "Standard_D4a_v4",
            "osType": "Linux",
            "osSku": "Ubuntu",
            "mode": "System"
          }
        ]
      }
    }
  ]
}
```

##### 1mcupdate.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerService/managedClusters",
      "apiVersion": "2023-07-01",
      "name": "akstestcluster",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayname": "Demo of AKS Nodepool Migration"
      },
      "identity": {
        "type": "SystemAssigned"
      },
      "properties": {
        "enableRBAC": true,
        "dnsPrefix": "testcluster",
        "agentPoolProfiles": [
          {
            "name": "testnp",
            "osType": "Linux",
            "osSku": "AzureLinux",
            "mode": "System"
          }
        ]
      }
    }
  ]
} 
```

##### 2apupdate.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "apiVersion": "2023-07-01",
      "type": "Microsoft.ContainerService/managedClusters/agentPools",
      "name": "akstestcluster/testnp",
      "location": "[resourceGroup().location]",
      "properties": {
        "osType": "Linux",
        "osSku": "Ubuntu",
        "mode": "System"
      }
    }
  ]
}
```

#### Deploy a test cluster

1. Create a resource group for the test cluster using the [`az group create`](/cli/azure/group#az_group_create) command. The following example creates a resource group named _testRG_ in the _eastus_ region:

    ```azurecli-interactive
    az group create --name testRG --location eastus
    ```

1. Deploy a baseline Ubuntu OS SKU cluster with three nodes using the [`az deployment group create`](/cli/azure/deployment/group#az_deployment_group_create) command and the [0base.json example ARM template](#0basejson).

    ```azurecli-interactive
    az deployment group create --resource-group testRG --template-file 0base.json
    ```

1. Migrate the OS SKU of your system node pool to Azure Linux using the [`az deployment group create`](/cli/azure/deployment/group#az_deployment_group_create) command.

    ```azurecli-interactive
    az deployment group create --resource-group testRG --template-file 1mcupdate.json
    ```

1. Migrate the OS SKU of your system node pool back to Ubuntu using the [`az deployment group create`](/cli/azure/deployment/group#az_deployment_group_create) command.

    ```azurecli-interactive
    az deployment group create --resource-group testRG --template-file 2apupdate.json
    ```

### [Terraform](#tab/terraform)

#### Example Terraform template

1. Confirm that your `providers.tf` file is updated to pick up the required version of the Azure provider. For example:

    ```terraform
    terraform {
          required_version = ">=1.0"
    
          required_providers {
            azurerm = {
              source  = "hashicorp/azurerm"
              version = "~>3.111.0"
            }
            random = {
              source  = "hashicorp/random"
              version = "~>3.0"
            }
          }
        }
    
        provider "azurerm" {
          features {}
        }
    ```

1. The following example `base.tf` Terraform template is a condensed version of a full Terraform template that deploys an AKS cluster with a node pool of `os_sku` with **Ubuntu**.

    ```terraform
    resource "azurerm_kubernetes_cluster" "k8s" {
      location            = azurerm_resource_group.rg.location
      name                = var.cluster_name
      resource_group_name = azurerm_resource_group.rg.name
      dns_prefix          = var.dns_prefix
      tags                = {
        Environment = "Development"
      }
    
      default_node_pool {
        name       = "azurelinuxpool"
        vm_size    = "Standard_D2_v2"
        node_count = var.agent_count
        os_sku = "Ubuntu"
      }
      linux_profile {
        admin_username = "azurelinux"
    
        ssh_key {
          key_data = file(var.ssh_public_key)
        }
      }
      network_profile {
        network_plugin    = "kubenet"
        load_balancer_sku = "standard"
      }
      service_principal {
        client_id     = var.aks_service_principal_app_id
        client_secret = var.aks_service_principal_client_secret
      }
    }
    ```

1. To run an in-place OS SKU migration, replace the `os_sku` to **AzureLinux** and reapply the Terraform plan. For example:

    ```terraform
    resource "azurerm_kubernetes_cluster" "k8s" {
      location            = azurerm_resource_group.rg.location
      name                = var.cluster_name
      resource_group_name = azurerm_resource_group.rg.name
      dns_prefix          = var.dns_prefix
      tags                = {
        Environment = "Development"
      }
    
      default_node_pool {
        name       = "azurelinuxpool"
        vm_size    = "Standard_D2_v2"
        node_count = var.agent_count
        os_sku = "AzureLinux"
      }
      linux_profile {
        admin_username = "azurelinux"
    
        ssh_key {
          key_data = file(var.ssh_public_key)
        }
      }
      network_profile {
        network_plugin    = "kubenet"
        load_balancer_sku = "standard"
      }
      service_principal {
        client_id     = var.aks_service_principal_app_id
        client_secret = var.aks_service_principal_client_secret
      }
    }
    ```

---

### Verify the OS SKU migration

Once the migration is complete on your test clusters, you should verify the following to ensure a successful migration:

- If your migration target is Azure Linux, run the `kubectl get nodes -o wide` command. The output should show `Microsoft Azure Linux 3.0` as your OS image and `.azl3` at the end of your kernel version.
- Run the `kubectl get pods -o wide -A` command to verify that all of your pods and daemonsets are running on the new node pool.
- Run the `kubectl get nodes --show-labels` command to verify that all of the node labels in your upgraded node pool are what you expect.

> [!TIP]
> We recommend monitoring the health of your service for a couple weeks before migrating your production clusters.

### Run the OS SKU migration on your production clusters

1. Update your existing templates to set `OSSKU=AzureLinux`. Make sure that your `apiVersion` is set to `2023-07-01` or later.

    - **ARM templates**: Use `"OSSKU": "AzureLinux"` in the `agentPoolProfile` section.
    - **Bicep**: Use `osSku: "AzureLinux"` in the `agentPoolProfile` section.
    - **Terraform**: Use `os_sku = "AzureLinux"` in the `default_node_pool` section.

1. Redeploy your ARM, Bicep, or Terraform template for the cluster to apply the new `OSSKU` setting. During this deploy, your cluster behaves as if it's taking a node image upgrade. Your cluster surges capacity, and then reboots your existing nodes one by one into the latest AKS image from your new OS SKU.

### Roll back to your previous OS SKU

If you experience issues during the OS SKU migration, you can roll back to your previous OS SKU. To do this, you need to change the OS SKU field in your template and resubmit the deployment, which triggers another upgrade operation and restores the node pool to its previous OS SKU.

You can roll back to your previous OS SKU using the [`az aks nodepool update`](/cli/azure/aks/nodepool#az_aks_nodepool_add) command. This command updates the OS SKU for your node pool from Azure Linux back to Ubuntu.

## Next step

In this tutorial, you migrated existing nodes to Azure Linux by removing existing node pools and adding new Azure Linux node pools or by performing an in-place OS SKU migration.

In the next tutorial, you learn how to enable telemetry to monitor your clusters.

> [!div class="nextstepaction"]
> [Enable telemetry and monitoring](./tutorial-monitor-azure-linux-aks.md)
