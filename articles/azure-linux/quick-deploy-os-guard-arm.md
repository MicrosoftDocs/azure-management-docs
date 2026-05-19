---
title: "Quickstart: Deploy an Azure Linux with OS Guard for AKS (Preview) Cluster using an ARM Template"
description: Learn how to quickly deploy an Azure Linux with OS Guard for Azure Kubernetes Service (AKS) cluster using an Azure Resource Manager (ARM) template.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: quickstart
ms.date: 04/28/2026
---

# Quickstart: Deploy an Azure Linux with OS Guard (preview) Azure Kubernetes Service (AKS) cluster using an Azure Resource Manager (ARM) template

[!INCLUDE [os-guard replacement](./includes/os-guard-replacement.md)]

Get started with the Azure Linux Container Host by using an Azure Resource Manager (ARM) template to deploy an Azure Linux with OS Guard (preview) cluster on AKS.

In this quickstart, you learn how to:

> [!div class="checklist"]
>
> - Install the `aks-preview` Azure CLI extension.
> - Register the `AzureLinuxOSGuardPreview` feature flag.
> - Install the Kubernetes CLI, `kubectl`.
> - Create an SSH key pair.
> - Review the ARM template.
> - Deploy the ARM template and validate it.
> - Deploy a sample application to the cluster.

[!INCLUDE [About Azure Resource Manager](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-introduction.md)]

## Prerequisites

- [!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]
- Use the Bash environment in [Azure Cloud Shell](/azure/cloud-shell/overview). For more information, see [Azure Cloud Shell Quickstart - Bash](/azure/cloud-shell/quickstart).

  :::image type="icon" source="~/reusable-content/ce-skilling/azure/media/cloud-shell/launch-cloud-shell-button.png" border="false" link="https://portal.azure.com/#cloudshell/":::

- If you prefer to run CLI reference commands locally, [install the Azure CLI](/cli/azure/install-azure-cli). If you're running on Windows or macOS, consider running Azure CLI in a Docker container. For more information, see [How to run the Azure CLI in a Docker container](/cli/azure/run-azure-cli-docker).

  - If you're using a local installation, sign in to the Azure CLI using the [`az login`](/cli/azure/reference-index#az-login) command. To finish the authentication process, follow the steps displayed in your terminal. For other sign-in options, see [Sign in with the Azure CLI](/cli/azure/authenticate-azure-cli).

  - When you're prompted, install the Azure CLI extension on first use. For more information about extensions, see [Use extensions with the Azure CLI](/cli/azure/azure-cli-extensions-overview).

  - Run [`az version`](/cli/azure/reference-index?#az-version) to find the version and dependent libraries that are installed. To upgrade to the latest version, run [`az upgrade`](/cli/azure/reference-index?#az-upgrade).

- If you don't already have kubectl installed, install it through Azure CLI using the [`az aks install-cli`](/cli/azure/aks#az_aks_get_credentials) command or follow the [upstream instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
- To create an AKS cluster using an ARM template, you provide an SSH public key. If you need this resource, see the following section; otherwise skip to the [Review the template](#review-the-template) section.
- The identity you're using to create your cluster has the appropriate minimum permissions. For more information on access and identity for AKS, see [Access and identity options for Azure Kubernetes Service (AKS)](/azure/aks/concepts-identity).
- To deploy a Bicep file or ARM template, you need write access on the resources you're deploying and access to all operations on the Microsoft.Resources/deployments resource type. For example, to deploy a virtual machine (VM), you need _Microsoft.Compute/virtualMachines/write_ and _Microsoft.Resources/deployments/*_ permissions. For a list of roles and permissions, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles).

[!INCLUDE [azure linux with os guard limitations](./includes/os-guard-limitations.md)]

## Install the `aks-preview` Azure CLI extension

[!INCLUDE [preview features callout](~/reusable-content/ce-skilling/azure/includes/aks/includes/preview/preview-callout.md)]

Install the `aks-preview` extension using the [`az extension add`](/cli/azure/extension#az_extension_add) command.

```azurecli-interactive
az extension add --name aks-preview
```

Update the `aks-preview` extension to the latest version using the [`az extension update`](/cli/azure/extension#az_extension_update) command.

```azurecli-interactive
az extension update --name aks-preview
```

## Register the `AzureLinuxOSGuardPreview` feature flag

1. Register the `AzureLinuxOSGuardPreview` feature flag using the [`az feature register`](/cli/azure/feature#az-feature-register) command.

    ```azurecli-interactive
    az feature register --namespace "Microsoft.ContainerService" --name "AzureLinuxOSGuardPreview"
    ```

    It takes a few minutes for the status to show _Registered_.

1. Verify the registration status using the [`az feature show`](/cli/azure/feature#az-feature-show) command.

    ```azurecli-interactive
    az feature show --namespace "Microsoft.ContainerService" --name "AzureLinuxOSGuardPreview"
    ```

1. When the status reflects _Registered_, refresh the registration of the _Microsoft.ContainerService_ resource provider using the [`az provider register`](/cli/azure/provider#az-provider-register) command.

    ```azurecli-interactive
    az provider register --namespace "Microsoft.ContainerService"
    ```

### Create an SSH key pair

To access AKS nodes, you connect using an SSH key pair (public and private), which you generate using the `ssh-keygen` command. By default, these files are created in the _~/.ssh_ directory. Running the `ssh-keygen` command overwrites any SSH key pair with the same name already existing in the given location.

1. Navigate to [https://shell.azure.com](https://shell.azure.com) to open Cloud Shell in your browser.
1. Run the `ssh-keygen` command. The following example creates an SSH key pair using RSA encryption and a bit length of 4096:

    ```bash
    ssh-keygen -t rsa -b 4096
    ```

For more information about creating SSH keys, see [Create and manage SSH keys for authentication in Azure](/azure/virtual-machines/linux/create-ssh-keys-detailed).

## Review the template

The following deployment uses an ARM template from [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.kubernetes/aks-azure-linux-osguard):

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.1",
    "parameters": {
        "clusterName": {
            "defaultValue": "osguardakscluster",
            "type": "String",
            "metadata": {
                "description": "The name of the Managed Cluster resource."
            }
        },
        "location": {
            "defaultValue": "[resourceGroup().location]",
            "type": "String",
            "metadata": {
                "description": "The location of the Managed Cluster resource."
            }
        },
        "dnsPrefix": {
            "type": "String",
            "metadata": {
                "description": "Optional DNS prefix to use with hosted Kubernetes API server FQDN."
            }
        },
        "agentCount": {
            "defaultValue": 3,
            "minValue": 1,
            "maxValue": 50,
            "type": "Int",
            "metadata": {
                "description": "The number of nodes for the cluster."
            }
        },
        "agentVMSize": {
            "defaultValue": "Standard_DS2_v2",
            "type": "String",
            "metadata": {
                "description": "The size of the Virtual Machine."
            }
        },
        "osSKU": {
            "defaultValue": "AzureLinuxOSGuard",
            "allowedValues": [
                "AzureLinuxOSGuard",
                "AzureLinux3OSGuard"
            ],
            "type": "String",
            "metadata": {
                "description": "The Linux SKU to use."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.ContainerService/managedClusters",
            "apiVersion": "2025-05-01",
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
                        "mode": "System",
                        "count": "[parameters('agentCount')]",
                        "vmSize": "[parameters('agentVMSize')]",
                        "osType": "Linux",
                        "osSKU": "[parameters('osSKU')]",
                        "osDiskType": "Managed",
                        "enableFIPS": true,
                        "securityProfile": {
                            "enableSecureBoot": true,
                            "enableVTPM": true
                        },
                    }
                ]
            }
        }
    ],
    "outputs": {
        "controlPlaneFQDN": {
            "type": "String",
            "value": "[reference(parameters('clusterName')).fqdn]"
        }
    }
}
```

To add Azure Linux with OS Guard to an existing ARM template, you need to add the following properties to the `agentPoolProfiles` section of your template:

- `"osSKU": "AzureLinuxOSGuard"`
- `"mode": "System"` to `agentPoolProfiles`
- `"osDiskType": "Managed"` to `agentPoolProfiles`
- `"enableFIPS": true` to `agentPoolProfiles`
- `"securityProfile": {enableSecureBoot: true enableVTPM: true}` to `agentPoolProfiles`
- Set the `apiVersion` to `2025-05-01` or newer (`"apiVersion": "2025-05-01"`).

## Deploy the template

1. Select the following button to sign in to Azure and open a template:

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="<https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.kubernetes%2Faks-azure-linux-osguard%2Fazuredeploy.json>":::

1. Configure the template parameters in the **Custom deployment** page. For this quickstart, leave the default values for the _OS Disk Size GB_, _Agent Count_, _Agent VM Size_, _OS Type_, and _Kubernetes Version_. Provide your own values for the following template parameters:

    - **Subscription**: Select an Azure subscription.
    - **Resource group**: Select **Create new**. Enter a unique name for the resource group, such as _ttestAzureLinuxOSGuardResourceGroup_, then choose **OK**.
    - **Location**: Select a location, such as **East US**.
    - **Cluster name**: Enter a unique name for the AKS cluster, such as _testAzureLinuxOSGuardCluster_.
    - **DNS prefix**: Enter a unique DNS prefix for your cluster, such as _myAzureLinuxOSGuardCluster_.
    - **Linux Admin Username**: Enter a username to connect using SSH, such as _azureUser_.
    - **SSH RSA Public Key**: Copy and paste the _public_ part of your SSH key pair (by default, the contents of ~/.ssh/id_rsa.pub).

1. Select **Review + Create**.

It takes a few minutes to create the Azure Linux Container Host cluster. Wait for the cluster to be successfully deployed before you move on to the next step.

## Validate the deployment

### Connect to the cluster

To manage a Kubernetes cluster, use the Kubernetes command-line client, [kubectl](https://kubernetes.io/docs/reference/kubectl/).

1. Install `kubectl` locally using the [`az aks install-cli`](/cli/azure/aks#az_aks_get_credentials) command. If you're using Azure Cloud Shell, `kubectl` is already installed.

    ```azurecli-interactive
    az aks install-cli
    ```

1. Configure `kubectl` to connect to your Kubernetes cluster using the [`az aks get-credentials`](/cli/azure/aks#az_aks_get_credentials) command. This command downloads credentials and configures the Kubernetes CLI to use them.

    ```azurecli-interactive
    az aks get-credentials --resource-group testAzureLinuxOSGuardResourceGroup --name testAzureLinuxCluster
    ```

1. Verify the connection to your cluster using the `kubectl get` command. This command returns a list of the cluster nodes.

    ```bash
    kubectl get nodes
    ```

    The following output example shows the three nodes created in the previous steps. Make sure the node status is _Ready_:

    ```output
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-agentpool-12345678-0   Ready    agent   6m44s   v1.12.6
    aks-agentpool-12345678-1   Ready    agent   6m46s   v1.12.6
    aks-agentpool-12345678-2   Ready    agent   6m45s   v1.12.6
    ```

### Deploy the application

A [Kubernetes manifest file](/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests) defines a cluster's desired state, such as which container images to run.

In this quickstart, you use a manifest to create all objects needed to run the [Azure Vote application](https://github.com/Azure-Samples/azure-voting-app-redis). This manifest includes two Kubernetes deployments:

- The sample Azure Vote Python applications.
- A Redis instance.

Two [Kubernetes Services](/azure/aks/concepts-network-services) are also created:

- An internal service for the Redis instance.
- An external service to access the Azure Vote application from the internet.

1. Create a file named `azure-vote.yaml`. (If you use the Azure Cloud Shell, you can create the file using `code`, `vi`, or `nano` as if working on a virtual or physical system.)
1. Copy in the following YAML definition:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-back
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-back
      template:
        metadata:
          labels:
            app: azure-vote-back
        spec:
          nodeSelector:
            "kubernetes.io/os": linux
          containers:
          - name: azure-vote-back
            image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
            env:
            - name: ALLOW_EMPTY_PASSWORD
              value: "yes"
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
            - containerPort: 6379
              name: redis
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-back
    spec:
      ports:
      - port: 6379
      selector:
        app: azure-vote-back
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-front
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-front
      template:
        metadata:
          labels:
            app: azure-vote-front
        spec:
          nodeSelector:
            "kubernetes.io/os": linux
          containers:
          - name: azure-vote-front
            image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
            - containerPort: 80
            env:
            - name: REDIS
              value: "azure-vote-back"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-front
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        app: azure-vote-front
    ```

    For a breakdown of YAML manifest files, see [Deployments and YAML manifests](/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests).

1. Deploy the application using the [`kubectl apply`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) command and specify the name of your YAML manifest:

    ```bash
    kubectl apply -f azure-vote.yaml
    ```

    The following example resembles output showing the successfully created deployments and services:

    ```output
    deployment "azure-vote-back" created
    service "azure-vote-back" created
    deployment "azure-vote-front" created
    service "azure-vote-front" created
    ```

### Test the application

When the application runs, a Kubernetes service exposes the application front end to the internet. This process can take a few minutes to complete.

1. Monitor progress using the [`kubectl get service`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command with the `--watch` argument.

    ```bash
    kubectl get service azure-vote-front --watch
    ```

    The **EXTERNAL-IP** output for the `azure-vote-front` service initially shows as _pending_.

    ```output
    NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
    azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
    ```

1. Once the **EXTERNAL-IP** address changes from _pending_ to an actual public IP address, use `CTRL-C` to stop the `kubectl` watch process. The following example output shows a valid public IP address assigned to the service:

    ```output
    azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
    ```

1. To see the Azure Vote app in action, open a web browser to the external IP address of your service.

    :::image type="content" source="./media/azure-voting-application.png" alt-text="Screenshot of browsing to Azure Vote sample application.":::

## Delete the cluster

If you no longer need them, you can clean up unnecessary resources to avoid Azure charges.

Delete the Azure resource group and all related resources using the [`az group delete`](/cli/azure/group#az_group_delete) command.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

## Related content

In this quickstart, you deployed an Azure Linux with OS Guard cluster. To learn more about Azure Linux with OS Guard, see the following resources:

- [Azure Linux OS Guard tutorial series: Part 1](./tutorial-create-cluster-os-guard-aks.md)
- [Overview of Azure Linux OS Guard for AKS](./os-guard-overview.md)
