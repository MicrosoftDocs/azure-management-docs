---
title: "Quickstart: Deploy an Azure Container Linux (ACL) AKS Cluster using an ARM Template"
description: Learn how to quickly deploy an Azure Container Linux (ACL) AKS cluster using an Azure Resource Manager (ARM) template and connect to it using `kubectl`.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: quickstart
ms.date: 04/28/2026
---

# Quickstart: Deploy an Azure Container Linux (ACL) Azure Kubernetes Service (AKS) cluster using an ARM template

In this quickstart, you use an Azure Resource Manager (ARM) template to create an Azure Kubernetes Service (AKS) cluster that runs Azure Container Linux (ACL) as the node operating system (OS). After installing the prerequisites, you create an SSH key pair, review the template, deploy the template, and connect to the cluster.

[!INCLUDE [azure container linux limitations](./includes/acl-limitations.md)]

## Prerequisites

> [!NOTE]
> You can use either [Azure Cloud Shell](/azure/cloud-shell/overview) or a local installation of the Azure CLI to run the commands in this quickstart.

- If you're running the Azure CLI locally, [install the Azure CLI](/cli/azure/install-azure-cli). If you're running on Windows or macOS, consider running Azure CLI in a Docker container. For more information, see [How to run the Azure CLI in a Docker container](/cli/azure/run-azure-cli-docker).
- If you're using a local installation, sign in to the Azure CLI using the [`az login`](/cli/azure/reference-index#az-login) command. To finish the authentication process, follow the steps displayed in your terminal. For other sign-in options, see [Sign in with the Azure CLI](/cli/azure/authenticate-azure-cli).
- If you're prompted, install the Azure CLI extension on first use. For more information about extensions, see [Use extensions with the Azure CLI](/cli/azure/azure-cli-extensions-overview).
- Azure Container Linux requires Azure CLI version 2.86.0 or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the Azure CLI version and dependent libraries that are installed. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.
- If you don't already have kubectl installed, install it through Azure CLI using the [`az aks install-cli`](https://learn.microsoft.com/cli/azure/aks#az-aks-install-cli) command or follow the [upstream instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
- To create an AKS cluster using an ARM template, you need to provide an SSH public key. If you need this resource, go to the [Create an SSH key pair](#create-an-ssh-key-pair) section to generate one before deploying the template. If you already have an SSH key pair, you can skip to the [Review the template](#review-the-template) section.
- The identity you're using to create your cluster needs to have the appropriate minimum permissions. For more information on access and identity for AKS, see [Access and identity options for Azure Kubernetes Service (AKS)](/azure/aks/concepts-identity).
- To deploy an ARM template, you need write access on the resources you're deploying and access to all operations on the Microsoft.Resources/deployments resource type. For example, to deploy a virtual machine (VM), you need Microsoft.Compute/virtualMachines/write and Microsoft.Resources/deployments/* permissions. For a list of roles and permissions, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles).

### Create an SSH key pair

To access AKS nodes, you connect using an SSH key pair (public and private), which you generate using the `ssh-keygen` command. By default, these files are created in the _~/.ssh_ directory. Running the `ssh-keygen` command overwrites any SSH key pair with the same name already existing in the given location.

1. Go to [https://shell.azure.com](https://shell.azure.com) to open Cloud Shell in your browser.
1. Run the `ssh-keygen` command. The following example creates an SSH key pair using RSA encryption and a bit length of 4096:

    ```console
    ssh-keygen -t rsa -b 4096
    ```

For more information about creating SSH keys, see [Create and manage SSH keys for authentication in Azure](/azure/virtual-machines/linux/create-ssh-keys-detailed).

## Review the template

The following deployment uses an ARM template from [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates/):

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "defaultValue": "aclakscluster",
      "metadata": {
        "description": "The name of the Managed Cluster resource."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "The location of the Managed Cluster resource."
      }
    },
    "dnsPrefix": {
      "type": "string",
      "metadata": {
        "description": "Optional DNS prefix to use with hosted Kubernetes API server FQDN."
      }
    },
    "osDiskSizeGB": {
      "type": "int",
      "defaultValue": 0,
      "minValue": 0,
      "maxValue": 1023,
      "metadata": {
        "description": "Disk size (in GB) to provision for each of the agent pool nodes. Specifying 0 applies the default disk size for that agentVMSize."
      }
    },
    "agentCount": {
      "type": "int",
      "defaultValue": 3,
      "minValue": 1,
      "maxValue": 50,
      "metadata": {
        "description": "The number of nodes for the cluster."
      }
    },
    "agentVMSize": {
      "type": "string",
      "defaultValue": "standard_d2s_v3",
      "metadata": {
        "description": "The size of the Virtual Machine."
      }
    },
    "linuxAdminUsername": {
      "type": "string",
      "metadata": {
        "description": "User name for the Linux Virtual Machines."
      }
    },
    "sshRSAPublicKey": {
      "type": "string",
      "metadata": {
        "description": "Configure all linux machines with the SSH RSA public key string."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.ContainerService/managedClusters",
      "apiVersion": "2026-03-01",
      "name": "[parameters('clusterName')]",
      "location": "[parameters('location')]",
      "identity": {
        "type": "SystemAssigned"
      },
      "properties": {
        "dnsPrefix": "[parameters('dnsPrefix')]",
        "agentPoolProfiles": [
          {
            "name": "agentpool",
            "osDiskSizeGB": "[parameters('osDiskSizeGB')]",
            "count": "[parameters('agentCount')]",
            "vmSize": "[parameters('agentVMSize')]",
            "osType": "Linux",
            "osSKU": "AzureContainerLinux",
            "mode": "System"
          }
        ],
        "linuxProfile": {
          "adminUsername": "[parameters('linuxAdminUsername')]",
          "ssh": {
            "publicKeys": [
              {
                "keyData": "[parameters('sshRSAPublicKey')]"
              }
            ]
          }
        }
      }
    }
  ],
  "outputs": {
    "controlPlaneFQDN": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.ContainerService/managedClusters', parameters('clusterName')), '2024-02-01').fqdn]"
    }
  }
}
```

The resource type defined in the ARM template is [**Microsoft.ContainerService/managedClusters**](/azure/templates/microsoft.containerservice/managedclusters?pivots=deployment-language-arm-template).

## Deploy the template

1. Select **Deploy to Azure** to sign in and open a template.

    [![Button to deploy the Resource Manager template to Azure.](../../reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.kubernetes%2Faks-azure-container-linux%2Fazuredeploy.json)

1. On the **Basics** page, leave the default values for the **OS Disk Size GB**, **Agent Count**, **Agent VM Size**, and **OS Type**, and configure the following template parameters:

    - **Subscription**: Select an Azure subscription.
    - **Resource group**: Select **Create new**. Enter a unique name for the resource group, such as _myACLResourceGroup_, and then select **OK**.
    - **OS SKU**: Specify **AzureContainerLinux**.
    - **Location**: Select a location, such as **West US**.
    - **Cluster name**: Enter a unique name for the AKS cluster, such as _myACLCluster_.
    - **DNS prefix**: Enter a unique DNS prefix for your cluster, such as _myaclcluster_.
    - **Linux Admin Username**: Enter a username to connect using SSH, such as _azureuser_.
    - **SSH public key source**: Select **Use existing public key**.
    - **Key pair name**: Copy and paste the _public_ part of your SSH key pair (by default, the contents of _~/.ssh/id\_rsa.pub_).

1. Select **Review + Create** > **Create**.

    It takes a few minutes to create the AKS cluster. Wait for the deployment to complete before you [connect to the cluster](#connect-to-the-cluster).

## Connect to the cluster

To manage a Kubernetes cluster, use the Kubernetes command-line client, [kubectl](https://kubernetes.io/docs/reference/kubectl/). `kubectl` is already installed if you use Azure Cloud Shell. To install `kubectl` locally, use the [`az aks install-cli`](/cli/azure/aks#az-aks-install-cli) command.

1. Configure `kubectl` to connect to your Kubernetes cluster using the [`az aks get-credentials`](/cli/azure/aks#az-aks-get-credentials) command. This command downloads credentials and configures the Kubernetes CLI to use them.

    ```azurecli-interactive
    az aks get-credentials --resource-group myACLResourceGroup --name myACLCluster
    ```

1. Verify the connection to your cluster using the [`kubectl get`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command. This command returns a list of the cluster nodes.

    ```bash
    kubectl get nodes
    ```

    The following example output shows the three nodes created in the previous steps. Make sure the node status is _Ready_:

    ```output
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-agentpool-12345678-0   Ready    agent   6m44s   v1.34.0
    aks-agentpool-12345678-1   Ready    agent   6m46s   v1.34.0
    aks-agentpool-12345678-2   Ready    agent   6m45s   v1.34.0
    ```

## Delete the cluster

If you no longer need the resources you created in this quickstart, you can clean them up to avoid Azure charges.

Delete the Azure resource group and all related resources using the [`az group delete`](/cli/azure/group#az-group-delete) command:

```azurecli-interactive
az group delete --name myACLResourceGroup --yes --no-wait
```

## Related content

In this quickstart, you deployed an Azure Container Linux (ACL) AKS cluster using an ARM template. To learn more about ACL, see the following resources:

- [What is Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)?](../1.%20acl-overview.md)
- [Deploy an ACL AKS cluster using the Azure CLI](./1.%20deploy-acl-cli.md)
