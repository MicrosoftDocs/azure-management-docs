---
title: Set up workload orchestration
description: Set up workload orchestration
ms.custom:
  - references_regions
  - build-2025
author: nathmanish
ms.author: nathmanish
ms.topic: setup-workload-orchestration
ms.date: 11/04/2025
# Customer intent: "As an IT admin, I want to onboard onto workload orchestration."
---

# Set up workload orchestration

This article walks you through the process of onboarding Workload Orchestration for Azure Arc.

---

## Pre-requisites

* An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
* Role-Based Access Control (RBAC) enabled user role assignment. For more information, see [Role-Based Access Control (RBAC) guide](rbac-guide.md).
* The latest version of Azure CLI installed on your development machine. For more information, see [How to install Azure CLI](/cli/azure/install-azure-cli).

* A kubectl client installed on your development machine. If you don't have kubectl installed, you can install it using the following command:

  ```bash	
  winget install -e --id Kubernetes.kubectl
  ```  

* An Arc-enabled cluster. For more information, see [Quickstart: Connect an existing Kubernetes cluster to Azure Arc](../kubernetes/quickstart-connect-cluster.md). The workload orchestration Arc extension doesn't support Arm-based architecture nodes, so make sure your cluster uses a non-Arm virtual machine and meets the following requirements:

    | Cluster Type | RAM | CPUs | Disk |
    |---|---|---|---|
    | Single node K3s | Minimum 4 GB | 2 | — |
    | Multi-node K8s | Minimum 4 GB per node | 2 per node | Extra 1 GB |

## Set up workload orchestration CLI

All sample input files required in this quide can be downloaded from the [workload orchestration GitHub repository](https://github.com/Azure/workload-orchestration). 

[![Download](https://img.shields.io/badge/Download%20zip%20file-0078D4?style=flat&labelColor=0078D4)](https://github.com/Azure/workload-orchestration/archive/refs/heads/main.zip)

### [Bash](#tab/bash)

1. Sign in to Azure CLI.

    ```bash
    az login
    ```

1. Choose the subscription you want to use from the list. Replace the placeholder variables with your values.

    ```bash
    subId="<SUBSCRIPTION_ID>"
    az account set --subscription "$subId"
    ```

1. Replace the placeholders for the following global variables with your values.

    ```bash
    rg="<AZURE_RESOURCE_GROUP_NAME>"
    # Use either "eastus" or "eastus2" as location for all workload orchestration resources
    l="<LOCATION>"
    clusterName="<CLUSTER_NAME>"
    siteName="<SITE_NAME>"
    customLocation="<CUSTOM_LOCATION_NAME>"
    contextName="<CONTEXT_NAME>"
    childName="<TARGET_NAME>"
    childDesc="This line is used for soap and conditioner production"
    level1="factory"
    level2="line"
    capChildList="[soap,shampoo]"
    ```

1. Install the workload orchestration and other required CLI extensions

    ```powershell
    az extension add --upgrade --name connectedk8s
    az extension add --upgrade --name k8s-extension
    az extension add --upgrade --name customlocation
    az extension add --name workload-orchestration
    ```

### [PowerShell](#tab/powershell)

1. Sign in to Azure CLI.

    ```powershell
    az login
    ```

1. Choose the subscription you want to use from the list. Replace the placeholder variables with your values.

    ```powershell
    $subId = "<SUBSCRIPTION_ID>"
    az account set --subscription $subId
    ```

1. Replace the placeholders for the following global variables with your values.

    ```powershell
    $rg = "<AZURE_RESOURCE_GROUP_NAME>"
    # Use either "eastus" or "eastus2" as location for all workload orchestration resources
    $l = "<LOCATION>"
    $clusterName = "<CLUSTER_NAME>"
    $siteName = "<SITE_NAME>"
    $customLocation="<CUSTOM_LOCATION_NAME>"
    $contextName="<CONTEXT_NAME>"
    $childName="<TARGET_NAME>"
    $childDesc="This line is used for soap and conditioner production"
    $level1="factory"
    $level2="line"
    $capChildList="[soap,shampoo]"
    ```

1. Install the workload orchestration and other required CLI extensions

    ```powershell
    az extension add --upgrade --name connectedk8s
    az extension add --upgrade --name k8s-extension
    az extension add --upgrade --name customlocation
    az extension add --name workload-orchestration
    ```

***


## Prepare your Arc cluster

1. Register Azure Resource Providers.

    ```azurecli
    az provider register --namespace Microsoft.Edge
    az provider register --namespace Microsoft.ContainerService
    az provider register --namespace Microsoft.ExtendedLocation
    az provider register --namespace Microsoft.KubernetesConfiguration
    az provider register --namespace Microsoft.Kubernetes
    ```

1. Initialize your Arc-connected Kubernetes cluster for application deployments using workload orchestration. This command installs the workload orchestration Arc extension along with cert-manager and trust-manager extensions, and creates a custom location on the cluster.

    ```azurecli
    az workload-orchestration cluster init -c "$clusterName" -g "$rg" -l "$l"
    ```

    > [!TIP]
    > You can choose to specify additional properties like name, version and release train of Arc extension, along with custom location details, by running `az workload-orchestration cluster init -c "$clusterName" -g "$rg" -l "$l" --release-train stable --extension-version 2.1.28  --extension-name "$extensionName" --custom-location-name "$customLocation"
    
    On successful run, the command writes `extended-location.json` containing the details of the custom location created, to the current directory.

    <details>
    <summary> Perform the cluster initialization steps individually </summary>

    You can also choose to manually set up your cluster for workload orchestration by following these steps:
    
    1. Install Cert Manager and Trust Manager.

        ```azurecli
        az k8s-extension create --resource-group "$rg" --cluster-name "$clusterName" --name "aio-certmgr" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager
        ```

    1. Run the following command and pick a storage class list to use as the persistent volume storage class for workload orchestration extension.
        ```azurecli
        kubectl get sc
        ```
    1. Run the following command to install the `microsoft.workloadorchestration` extension:
    
        ```azurecli
        storageClassName="<pick up one storage class from 'kubectl get sc'>"
        az k8s-extension create --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --extension-type Microsoft.workloadorchestration --scope cluster --release-train stable --config redis.persistentVolume.storageClass="$storageClassName" --config redis.persistentVolume.size=20Gi 
        ```
    1. Enable custom location for the cluster.

        ### [Bash](#tab/bash)
        ```bash
        clusterId=$(az connectedk8s show --resource-group "$rg" --name "$clusterName" --query id --output tsv)
        extensionId=$(az k8s-extension show --resource-group "$rg" --name "$extensionName" --cluster-type connectedClusters --cluster-name "$clusterName" --query id --output tsv)
        az customlocation create --resource-group "$rg" --name "${clusterName}-Location" --namespace default --host-resource-id "$clusterId" --cluster-extension-ids "$extensionId" --location "$l"
        ```

        ### [PowerShell](#tab/powershell)
        ```powershell
        $clusterId = az connectedk8s show --resource-group $rg --name $clusterName --query id --output tsv
        $extensionId = az k8s-extension show --resource-group $rg --name $extensionName --cluster-type connectedClusters --cluster-name $clusterName --query id --output tsv
        az customlocation create --resource-group $rg --name "$clusterName-Location" --namespace default --host-resource-id $clusterId --cluster-extension-ids $extensionId --location $l
        ```
        ---
    
    </details>

1. This step is applicable only if you are planning to set up an Azure Container Registry (ACR) to host your container images.

    1. Set up ACR Image Pull for the cluster. If you're using an AKS cluster, follow the instructions in [Authenticate with Azure Container Registry (ACR) from Azure Kubernetes Service (AKS)](/azure/aks/cluster-container-registry-integration). If you're using a different type of cluster, follow the instructions in [Pull images from an Azure container registry to a Kubernetes cluster using a pull secret](/azure/container-registry/container-registry-auth-kubernetes).

    1. Set up ACR Helm Chart Pull (any Arc connected cluster). Verify that the extension has a system managed identity. Run the following command:

        ```azurecli
        az k8s-extension show --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --query identity
        ```

    1. Assign the `AcrPull` role to the identity of the `microsoft.workloadorchestration` extension. Replace `<IDENTITY_ID>` with the identity ID from the previous step.

        ### [Bash](#tab/bash)
        ```bash
        extensionSPId=$(az k8s-extension show --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --query identity.principalId --output tsv)
        az role assignment create --assignee "$extensionSPId" --role "AcrPull" --scope "<ACR Resource ID>"
        ```
        
        ### [PowerShell](#tab/powershell)
        ```powershell
        $extensionSPId = az k8s-extension show --resource-group $rg --cluster-name $clusterName --cluster-type connectedClusters --name $extensionName --query identity.principalId --output tsv
        az role assignment create --assignee $extensionSPId --role "AcrPull" --scope "<ACR Resource ID>"
        ```
---

## Set up the workload orchestration resources

1. Create the [Site hierarchy](resource-model.md#hierarchy). You can choose between the following two types of hierarchies based on your requirements.

    ### [Resource Group Hierarchy](#tab/resource-group-hierarchy) 

    ```azurecli
    az workload-orchestration hierarchy create -g "$rg" --configuration-location "$l" --hierarchy-spec "name=$contextName level=$level1"
    ```

    You can also store the hierarchy details in a YAML file and pass it as an argument as shown below. The sample file `resource-group-hierarchy.yaml` can be downloaded from [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). You can specify both new and existing Sites to be used for your hierarchy in the YAML file.

    ```azurecli
    az workload-orchestration hierarchy create -g "$rg" --configuration-location "$l" --hierarchy-spec "hierarchy.yaml"
    ```

    ### [Service Group Hierarchy](#tab/service-group-hierarchy) 

    The following command creates a Service Group hierarchy according to the structure defined in `service-group-hierarchy.yaml`. You can refer to the sample file from [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). You can specify both new and existing Sites to be used for your hierarchy in the YAML file.

    ```azurecli
    az workload-orchestration hierarchy create -g "$rg" --configuration-location "$l" --hierarchy-spec "hierarchy.yaml"
    ```

    <details>
    <summary> Manually add Sites to hierarchy </summary>
    You also have the option to manually add a Site to any level of the hierarchy, using the following steps:
    
      1. Create the Service Group
          ```azurecli
          $sg = "<service-group-name>"
          $tenantId = "<tenant-id>"

          az rest --method put --header Content-Type=application/json --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$sg`?api-version=2024-02-01-preview --body "{'properties':{'displayName':'$sg','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$tenantId'}}}" --resource https://management.azure.com
          ```
      1. Create and tag a Site with the respective hierarchy level.

          ```azurecli
          az rest --method put --url "https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName?api-version=2025-03-01-preview" --body "{'properties':{'displayName':'$siteName','description': '$siteName','labels': {'level': 'Factory'}}}" --resource https://management.azure.com
          ```
      1. Create the configuration reference.
          ```azurecli
          configName="<configuration name>"
          configId="/subscriptions/$subId/resourceGroups/$rg/providers/microsoft.edge/configurations/$configName"
          az rest --method put --url "$configId?api-version=2025-08-01" --body "{'location':'$l'}"
          az rest --method put --url "$siteId/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'$configId'}}"
          ```
    </details>

1. Create the workload orchestration [context](resource-model.md#context) or environment with the hierarchy created in the previous step, and with the desired set of capabilities that you want to include in your targets. The hierarchy level names must match those specified in the previous step.

    ```azurecli
    az workload-orchestration context create -g "$rg" -n "$contextName" -l "$l" --capabilities "[{name:soap,description:Soap},{name:shampoo,description:Shampoo}]" --hierarchies "[0].name=$level1" "[0].description=$level1" "[1].name=$level2" "[1].description=$level2" --site-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$siteName
    ```

    Use the ARM ID of the parent Site of your hierarchy for the `--site-id` argument. If you do not have an existing hierarchy, you can create the context first and later link it to the parent site using:

    ```azurecli
    az workload-orchestration context site-reference create --subscription "$subId" --resource-group "$rg" --context-name "$contextName" --name "$siteReference" --site-id "$siteId"
    ```

    > [!NOTE]
    > You can also pass the list of capabilities in a JSON file using `--capabilities @capabilities.json` (sample file included in [GitHub repository](https://github.com/Azure/workload-orchestration)). Description for capabilities is optional and will default to the capability name if not specified.

    <details>
    <summary> Use existing context </summary>
    You can also use an already existing context by running the `context create` command with the `--context-id` parameter while passing the desired list of capabilities and hierarchies into it. You can add more capabilities, but removing and deleting isn't supported.
    
    ```azurecli
    az workload-orchestration context create --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/context/$contextName" --hierarchies "[0].name=factory" "[0].description=Factory" "[1].name=line" "[1].description=Line" --capabilities @capabilities.json
    ```

    You can add new capabilities to your existing context.
    ```azurecli
    az workload-orchestration context add-capability -g "$rg" --context-name "$contextName" --capabilities "[{name:soap,description:Soap},{name:shampoo,description:Shampoo}]" 
    ```

    Existing capabilities can also be deleted from a context by running:
    ```azurecli
    az workload-orchestration context remove-capability -g "$rg" --context-name "$contextName" --capabilities "[shampoo,detergent]" --yes
    ```
    </details>

    > [!NOTE]
    > Workload orchestration currently supports creation of only 1 context per Azure tenant. The context name must be between 3 and 61 characters in length and follow the naming pattern defined by the regular expression `^a-zA-Z0-9?(\.a-zA-Z0-9?)*$`. This means:
    > - Must start and end with an alphanumeric character.
    > - Can contain hyphens, but not at the start or end of any segment.
    > - Can contain dots to separate segments, but not consecutive dots or empty segments.
    > - Can't have any special characters other than hyphen and dot.

1. Create a [target](resource-model.md#target) reference. The attribute `--solution-scope` specifies the cluster namespace. The `--target-specification` attribute specifies that Helm charts are being used for the K8s deployment. The `--extended-location` attribute is used to specify the custom location of the Arc cluster.

    ### [Resource Group Hierarchy](#tab/resource-group-hierarchy)

    ```azurecli
    az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope "new" --target-specification "@targetspecs.json" --extended-location type=CustomLocation name="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.ExtendedLocation/customLocations/$customLocation" --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

    ### [Service Group Hierarchy](#tab/service-group-hierarchy)

    ```azurecli
    az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope "new" --target-specification "@targetspecs.json" --extended-location type=CustomLocation name="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.ExtendedLocation/customLocations/$customLocation" --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName --service-group $sg
    ```

    You can also create a target without the `--service-group` argument, and later link it to a service group in the hierarchy using:

    ```azurecli
    targetId=$(az workload-orchestration target show --resource-group "$rg" --name "$childName" --query id --output tsv)
    az rest --method put --url "$targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" --body "{'properties':{'targetId': '/providers/Microsoft.Management/serviceGroups/$sg'}}"
    ```
    ---

    > [!TIP]
    > You can update the list of capability tags for an existing target by rerunning the `az workload-orchestration target create` command with the new set of values for `--capabilities` argument, while keeping the other parameters same.

***

## Next steps

Once you have set up the infrastructure and the workload orchestration resources, you can start authoring solutions and managing deployments. To get started, refer to [Deploy a basic solution](solution-without-common-configuration.md) to learn how to create a basic solution, configure it, and deploy it to a target.


## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]