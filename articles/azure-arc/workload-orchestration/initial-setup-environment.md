---
title: Prepare the Environment for Workload Orchestration 
description: Learn how to set up the environment for workload orchestration. This procedure is done by IT admins.
ms.custom:
  - references_regions
  - build-2025
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: install-set-up-deploy
ms.date: 06/24/2025
---

# Prepare the environment for workload orchestration

IT admins are responsible for the initial setup of workload orchestration, which includes creating a custom location, downloading the workload orchestration extension, and installing the required components. The IT admin also needs to set up the Azure resources for workload orchestration, including the Azure Kubernetes Service (AKS) cluster, site, and site address.

This article describes how to prepare the environment for workload orchestration. The following steps are shared across all Azure resources. 

> [!TIP]
> You can follow the instructions in this article and run through each command, or if you prefer, you can run the [onboarding scripts](onboarding-scripts.md) for a one-click setup.

## Prerequisites

* An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
* Role-Based Access Control (RBAC) enabled user role assignment. For more information, see [Role-Based Access Control (RBAC) guide](rbac-guide.md).
* An Arc-enabled cluster. For more information, see [Quickstart: Connect an existing Kubernetes cluster to Azure Arc](../kubernetes/quickstart-connect-cluster.md).

    > [!NOTE]
    > The workload orchestration Arc extension doesn't support Arm-based architecture nodes. If you're using Azure Kubernetes Service for your cluster, make sure that it uses a non-Arm virtual machine.

* The latest version of Azure CLI installed on your development machine. For more information, see [How to install Azure CLI](/cli/azure/install-azure-cli).
* The latest version of the following Azure CLI extensions:

  ```bash
  az extension add --upgrade --name connectedk8s
  az extension add --upgrade --name k8s-extension
  az extension add --upgrade --name customlocation
  ```

* A kubectl client installed on your development machine. If you don't have kubectl installed, you can install it using the following command:

  ```bash	
  winget install -e --id Kubernetes.kubectl
  ```  
 
> [!IMPORTANT]
> Standard Azure resources, such as Arc-enabled Kubernetes clusters and custom location, and workload orchestration resources, such as context, targets, and solutions, must be created in the same Azure region. 

## System requirements

The following system footprint requirements are needed to run workload orchestration.

- Single node K3s cluster:
    - Minimum 4GB of RAM and 2 CPUs.                                               
    - Most memory consumed by Kubernetes components, ARC agents, Strato, and Cert Manager. 
- Multi-node K8s cluster:
    - Each node requires a minimum of 4GB of RAM and 2 CPUs.                        
    - Extra 1GB of disk space required for storing internal state.             

## Global availability

Workload orchestration is available for Arc-enabled clusters in the following Azure regions: 
- East US
- East US 2

## Prepare the basics to run workload orchestration

The following steps show how to prepare your environment to configure workload orchestration. 

Global variables, JSON files, and other configuration resources can be downloaded from this [ZIP folder in GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip). You can extract the downloaded artifacts from the compressed into a particular folder. 

Run the following commands to extract the files from the zip file. Skip if you already extracted files.

### [Bash](#tab/bash)

```bash
Expand-Archive -Force <enter zip file path e.g. "C:\path\to\archive.zip"> <enter folder path e.g. "C:\path\to\cm\workspace">
#Point to the directory
cd <enter folder path e.g. "C:\path\to\cm\workspace">
```

### [PowerShell](#tab/powershell)

```powershell
Expand-Archive -Path "<enter zip file path e.g. 'C:\path\to\archive.zip'>" -DestinationPath "<enter folder path e.g. 'C:\path\to\cm\workspace'>" -Force
# Point to the directory
Set-Location -Path "<enter folder path e.g. 'C:\path\to\cm\workspace'>"
```

***

### Set up the Azure CLI commands 

#### [Bash](#tab/bash)

1. Sign in to Azure CLI.

    ```bash
    az login
    ```

1. After you sign in, Azure CLI displays all of your subscriptions and indicates your default subscription with an asterisk `*`. 
1. Choose the subscription you want to use from the list. Replace the placeholder variables with your values.

    ```bash
    subId="<SUBSCRIPTION_ID>"
    az account set --subscription "$subId"
    ```

1. Global variables are commonly used variables that can help with the next set of commands. Replace the placeholder variables with your values.

    ```bash
    rg="<RESOURCE_GROUP_NAME>"
    l="<LOCATION>"
    clusterName="<CLUSTER_NAME>"
    siteName="<SITE_NAME>"
    ```

#### [PowerShell](#tab/powershell)

1. Sign in to Azure CLI.

    ```powershell
    az login
    ```

1. After you sign in, Azure CLI displays all of your subscriptions and indicates your default subscription with an asterisk `*`. 
1. Choose the subscription you want to use from the list. Replace the placeholder variables with your values.

    ```powershell
    $subId = "<SUBSCRIPTION_ID>"
    az account set --subscription $subId
    ```

1. Global variables are commonly used variables that can help with the next set of commands. Replace the placeholder variables with your values.

    ```powershell
    $rg = "<RESOURCE_GROUP_NAME>"
    $l = "<LOCATION>"
    $clusterName = "<CLUSTER_NAME>"
    $siteName = "<SITE_NAME>"
    ```

***

## Install workload orchestration CLI extension

The workload orchestration CLI extension is required to run the commands for workload orchestration. The extension is available in the Azure CLI extension index.

[!INCLUDE [cli-version-note](includes/cli-version-note.md)]

### [Bash](#tab/bash)

```bash
az extension add --name workload-orchestration
```

### [PowerShell](#tab/powershell)

```powershell
az extension add --name workload-orchestration
```
    
***

## Set up the required Azure resources 

The following steps are required to set up the Azure resources for workload orchestration.

### [Bash](#tab/bash)

1. Run the following command to check the installed extensions and update them if necessary:

    ```bash
    az extension add --name connectedk8s
    az extension add --name customlocation
    az extension add --name k8s-extension
    az extension update --name connectedk8s
    az extension update --name customlocation
    az extension update --name k8s-extension
    ```

1. Register the resource provider for the custom location. Run the following commands:

    ```bash
    az provider register --namespace Microsoft.Edge
    az provider register --namespace Microsoft.ContainerService && az provider register --namespace Microsoft.ExtendedLocation 
    az provider register --namespace Microsoft.KubernetesConfiguration && az provider register --namespace Microsoft.Kubernetes
    ```

1. Create resource group for the custom location.

    ```bash
    az group create --location "$l" --name "$rg"
    ```

1. Create the Azure Kubernetes Service (AKS) cluster. The location of your Azure Arc enabled cluster, custom location and workload orchestration objects should be the same. For information about virtual machines available sizes, see [Sizes for virtual machines in Azure](/azure/virtual-machines/sizes/overview).

    ```bash
    az identity create --resource-group "$rg" --name "$clusterName"
    clusterIdentity=$(az identity show --resource-group "$rg" --name "$clusterName" --query id --output tsv)
    az aks create --resource-group "$rg" --location "$l" --name "$clusterName" --node-count "<node-count>" --assign-identity "$clusterIdentity" --generate-ssh-keys --node-vm-size "<node size>"
    ```

    > [!NOTE]
    > To connect to an AKS cluster through the Azure portal, follow these steps:
    > 
    > 1. Open the [Azure portal](https://ms.portal.azure.com) and log in with your Azure account.
    > 1. In the search bar at the top of the portal, type **Kubernetes services** and select it from the search results to access the Kubernetes services page.
    > 1. From the list of Kubernetes services, select the **AKS cluster** you want to connect to.
    > 1. On the **Overview page** of your AKS cluster, click on the **Connect** button from the top menu.

### [PowerShell](#tab/powershell)

1. Run the following command to check the installed extensions and update them if necessary:

    ```powershell
    az extension add --name connectedk8s
    az extension add --name customlocation
    az extension add --name k8s-extension
    az extension update --name connectedk8s
    az extension update --name customlocation
    az extension update --name k8s-extension
    ```

1. Register the resource provider for the custom location. Run the following commands:

    ```powershell
    az provider register --namespace Microsoft.Edge
    az provider register --namespace Microsoft.ContainerService
    az provider register --namespace Microsoft.ExtendedLocation
    az provider register --namespace Microsoft.KubernetesConfiguration
    az provider register --namespace Microsoft.Kubernetes
    ```

1. Create resource group for the custom location.

    ```powershell
    az group create --location $l --name $rg
    ```

1. Create the Azure Kubernetes Service (AKS) cluster. The location of your Azure Arc enabled cluster, custom location and workload orchestration objects should be the same. For information about virtual machines available sizes, see [Sizes for virtual machines in Azure](/azure/virtual-machines/sizes/overview).

    ```powershell
    az identity create --resource-group $rg --name $clusterName
    $clusterIdentity = az identity show --resource-group $rg --name $clusterName --query id --output tsv
    az aks create --resource-group $rg --location $l --name $clusterName --node-count <node-count> --assign-identity $clusterIdentity --generate-ssh-keys --node-vm-size <node size>
    ```

    > [!NOTE]
    > To connect to an AKS cluster through the Azure portal, follow these steps:
    > 
    > 1. Open the [Azure portal](https://ms.portal.azure.com) and log in with your Azure account.
    > 1. In the search bar at the top of the portal, type **Kubernetes services** and select it from the search results to access the Kubernetes services page.
    > 1. From the list of Kubernetes services, select the **AKS cluster** you want to connect to.
    > 1. On the **Overview page** of your AKS cluster, click on the **Connect** button from the top menu.

***

Once the resources are created, they are visible in the Overview page of the Resource Group in the Azure portal. You can also view the resources in the [Resource Explorer page](https://resources.azure.com).

## Install the required components for workload orchestration 

The following steps are required to run workload orchestration service component.

### [Bash](#tab/bash)

1. Enable Azure Arc on the Kubernetes cluster.

    ```bash
    az aks get-credentials --resource-group "$rg" --name "$clusterName"
    kubectl config use-context "$clusterName" 
    az connectedk8s connect --resource-group "$rg" --location "$l" --name "$clusterName"
    az connectedk8s enable-features --resource-group "$rg" --name "$clusterName" --features cluster-connect custom-locations
    ```

1. Install Cert Manager and Trust Manager.

    ```bash
    az k8s-extension create --resource-group "$rg" --cluster-name "$clusterName" --name "aio-certmgr" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager
    ```

1. Determine if you installed the `microsoft.workloadorchestration` Arc extension on the Arc cluster. 

    ```bash
    az k8s-extension list --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --query "[?extensionType=='microsoft.workloadorchestration'].name"
    ```
 
    1. If the output returns an empty list, it means you don't have the `microsoft.workloadorchestration` extension installed on your Arc cluster. Run the following command and pick a storage class list to use as the persistent volume storage class for workload orchestration extension.

        ```bash
        kubectl get sc
        ```
        Run the following command to install the `microsoft.workloadorchestration` extension:
    
        ```bash
        storageClassName="<pick up one storage class from 'kubectl get sc'>"
        az k8s-extension create --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --extension-type Microsoft.workloadorchestration --scope cluster --release-train stable --config redis.persistentVolume.storageClass="$storageClassName" --config redis.persistentVolume.size=20Gi --extension-version "2.0.10" # or latest workload orchestration Arc version
        ```

    1. If you already installed the `microsoft.workloadorchestration` Arc extension, you can update it. Make sure to replace `<extensionName>` with the name of your existing extension. 
    
        ```bash
        az k8s-extension update --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --release-train stable  --auto-upgrade true
        ``` 

1. Enable custom location for the cluster.

    ```bash
    clusterId=$(az connectedk8s show --resource-group "$rg" --name "$clusterName" --query id --output tsv)
    extensionId=$(az k8s-extension show --resource-group "$rg" --name "$extensionName" --cluster-type connectedClusters --cluster-name "$clusterName" --query id --output tsv)
    az customlocation create --resource-group "$rg" --name "${clusterName}-Location" --namespace default --host-resource-id "$clusterId" --cluster-extension-ids "$extensionId" --location "$l"
    ```

    > [!NOTE]
    > If you are using `az cli 2.70.0` and experience any issues with the `az customlocation create` command, then you can create custom location from Azure portal.
    > 1. Open the Azure portal and log in with your Azure account.
    > 1. Click **+ Create a resource** and search for **custom location**.
    > 1. Follow the instructions in portal to create custom location.
    > 1. In the **Basics** tab, choose your subscription and resource group.
    > 1. Enter your custom location name and select your Arc cluster.
    > 1. Select the `microsoft.workloadorchestration` Arc extension and enter your namespace.

1. Set up Azure Container Registry (ACR) Image Pull for the cluster. If you're using an AKS cluster, follow the instructions in [Authenticate with Azure Container Registry (ACR) from Azure Kubernetes Service (AKS)](/azure/aks/cluster-container-registry-integration). If you're using a different type of cluster, follow the instructions in [Pull images from an Azure container registry to a Kubernetes cluster using a pull secret](/azure/container-registry/container-registry-auth-kubernetes).
1. Set up ACR Helm Chart Pull (any Arc connected cluster). Verify that the extension has a system managed identity. Run the following command:

    ```bash
    az k8s-extension show --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --query identity
    ```

1. Assign the `AcrPull` role to the identity of the `microsoft.workloadorchestration` extension. Replace `<IDENTITY_ID>` with the identity ID from the previous step.

    ```bash
    extensionSPId=$(az k8s-extension show --resource-group "$rg" --cluster-name "$clusterName" --cluster-type connectedClusters --name "$extensionName" --query identity.principalId --output tsv)
    az role assignment create --assignee "$extensionSPId" --role "AcrPull" --scope "<ACR Resource ID>"
    ```

    > [!NOTE]
    > If you don't have the ACR resource ID, run the steps in [Authenticate with Azure Container Registry (ACR) from Azure Kubernetes Service (AKS)](/azure/aks/cluster-container-registry-integration#create-a-new-acr) to create a new ACR.

1. Assign access to workload orchestration service. On the resource group where all workload orchestration resources are placed, provide contributor access to the Azure AD application “EdgeConfigurationManagerApp (cba491bc-48c0-44a6-a6c7-23362a7f54a9)” from Azure portal. 

### [PowerShell](#tab/powershell)

1. Enable Azure Arc on the Kubernetes cluster.

    ```powershell
    az aks get-credentials --resource-group $rg --name $clusterName
    kubectl config use-context $clusterName 
    az connectedk8s connect --resource-group $rg --location $l --name $clusterName
    az connectedk8s enable-features --resource-group $rg --name $clusterName --features cluster-connect custom-locations
    ```

1. Install Cert Manager and Trust Manager.

    ```powershell
    az k8s-extension create --resource-group $rg --cluster-name $clusterName --name "aio-certmgr" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager
    ```

1. Determine if you installed the `microsoft.workloadorchestration` Arc extension on the Arc cluster. 

    ```powershell
    az k8s-extension list --resource-group $rg --cluster-name $clusterName --cluster-type connectedClusters --query "[?extensionType=='microsoft.workloadorchestration'].name"
    ```
 
    1. If the output returns an empty list, it means you don't have the `microsoft.workloadorchestration` extension installed on your Arc cluster. Run the following command and pick a storage class list to use as the persistent volume storage class for workload orchestration extension.

        ```bash
        kubectl get sc
        ```
        Run the following command to install the `microsoft.workloadorchestration` extension:
    
        ```powershell
        $storageClassName = "<pick up one storage class from 'kubectl get sc'>"
        az k8s-extension create --resource-group $rg --cluster-name $clusterName --cluster-type connectedClusters --name $extensionName --extension-type Microsoft.workloadorchestration --scope cluster --release-train stable --config redis.persistentVolume.storageClass=$storageClassName --config redis.persistentVolume.size=20Gi --extension-version "2.0.10" # or latest workload orchestration Arc version
        ```      

    1. If you already installed the `microsoft.workloadorchestration` Arc extension, you can update it. Make sure to replace `<extensionName>` with the name of your existing extension. 
    
        ```powershell
        az k8s-extension update --resource-group $rg --cluster-name $clusterName --cluster-type connectedClusters --name $extensionName --release-train stable  --auto-upgrade true
        ```

1. Enable custom location for the cluster.

    ```powershell
    $clusterId = az connectedk8s show --resource-group $rg --name $clusterName --query id --output tsv
    $extensionId = az k8s-extension show --resource-group $rg --name $extensionName --cluster-type connectedClusters --cluster-name $clusterName --query id --output tsv
    az customlocation create --resource-group $rg --name "$clusterName-Location" --namespace default --host-resource-id $clusterId --cluster-extension-ids $extensionId --location $l
    ```

    > [!NOTE]
    > If you are using `az cli 2.70.0` and experience any issues with the `az customlocation create` command, then you can create custom location from Azure portal.
    > 1. Open the Azure portal and log in with your Azure account.
    > 1. Click **+ Create a resource** and search for **custom location**.
    > 1. Follow the instructions in portal to create custom location.
    > 1. In the **Basics** tab, choose your subscription and resource group.
    > 1. Enter your custom location name and select your Arc cluster.
    > 1. Select the `microsoft.workloadorchestration` Arc extension and enter your namespace.

1. Set up Azure Container Registry (ACR) Image Pull for the cluster. If you're using an AKS cluster, follow the instructions in [Authenticate with Azure Container Registry (ACR) from Azure Kubernetes Service (AKS)](/azure/aks/cluster-container-registry-integration). If you're using a different type of cluster, follow the instructions in [Pull images from an Azure container registry to a Kubernetes cluster using a pull secret](/azure/container-registry/container-registry-auth-kubernetes).
1. Set up ACR Helm Chart Pull (any Arc connected cluster). Verify that the extension has a system managed identity.

    ```powershell
    az k8s-extension show --resource-group $rg --cluster-name $clusterName --cluster-type connectedClusters --name $extensionName --query identity
    ```

1. Assign the `AcrPull` role to the identity of the `microsoft.workloadorchestration` extension. Replace `<IDENTITY_ID>` with the identity ID from the previous step.

    ```powershell
    $extensionSPId = az k8s-extension show --resource-group $rg --cluster-name $clusterName --cluster-type connectedClusters --name $extensionName --query identity.principalId --output tsv
    az role assignment create --assignee $extensionSPId --role "AcrPull" --scope "<ACR Resource ID>"
    ```

    > [!NOTE]
    > If you don't have the ACR resource ID, run the steps in [Authenticate with Azure Container Registry (ACR) from Azure Kubernetes Service (AKS)](/azure/aks/cluster-container-registry-integration#create-a-new-acr) to create a new ACR.

1. Assign access to workload orchestration service. On the resource group where all workload orchestration resources are placed, provide contributor access to the Azure AD application “EdgeConfigurationManagerApp (cba491bc-48c0-44a6-a6c7-23362a7f54a9)” from Azure portal. 

***

At this point, the environment and infrastructure for workload orchestration should be set up with the required permissions, Arc-connected cluster, extensions and plugins to support configuration and management of the solution/applications.


## Create site address and site

Sites and site addresses represent the physical hierarchy of your organization, such as a plant, factory, or store. Hierarchies are defined as name-description pairs that map to the levels of your organization's resource structure. For example, a manufacturing organization might use two levels: Factory and Line.

You can create sites using either a resource group or a service group. The choice depends on the hierarchy of your organization and how you want to logically group resources.

|Group type|Description|Note|
|---|---|---|
|Service group|A service group is a resource type in Azure Resource Manager (ARM) that helps you custom grouping your services for organizational or monitoring purposes, while maintaining your existing setup. For more information, see [Service groups](service-group.md)| For hierarchies with more than two levels, you need to use service groups to create sites. If you want to logically group resources without modifying the existing resource group, consider using service groups.|
|Resource group|A resource group is a core Azure management logical container that holds related Azure resources.| For hierarchies with two levels, you can use either service groups or resource groups. |

To use a service group, follow the steps in [Create a service group](service-group.md#create-a-service-group). 

To use a resource group, run the following commands:

### [Bash](#tab/bash)

1. Create a JSON file for site address and contact details. Name the file `<SITE_NAME_ADDRESS>.json` and save it in the same directory as your CLI commands. The JSON file must contain the following information:

    ```json
       {
           "contactDetails": {
               "contactName": "<CONTACT_NAME>",
               "emailList": [
                   "<EMAIL_LIST>"        
               ],
               "phone": "<PHONE>",
               "phoneExtension": "<PHONE_EXTENSION>"
           },
           "shippingAddress": {
               "addressType": "<ADDRESS_TYPE>",
               "city": "<CITY>",
               "companyName": "<COMPANY_NAME>",
               "country": "<COUNTRY>",
               "postalCode": "<POSTAL_CODE>",
               "stateOrProvince": "<STATE_OR_PROVINCE>",
               "streetAddress1": "<STREET_ADDRESS_1>",
               "streetAddress2": "<STREET_ADDRESS_2>"
           }
        }
    ```

1. Create a JSON file for site and name it */<SITE_NAME>/.json*. The JSON file must contain the following information:

    ```json
        {
            "properties": {
                "description": "<DESCRIPTION>",
                "addressResourceId": "<ADDRESS_RESOURCE_ID>",
                "displayName": "<SITE_NAME>"
            }
        }
    ```

    > [!TIP]
    > You can refer to *redmond-site-address.json* and *redmond-site.json* files in the downloaded ZIP folder for examples of how to create the JSON files.

1. Define the global variables for site address and site. Replace the placeholder variables with the values from your JSON files.

    ```bash
    siteJson="<SITE_NAME>.json"
    siteAddressJson="<SITE_NAME_ADDRESS>.json"
    siteUri="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$siteName?api-version=2024-02-01-preview"
    siteId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$siteName"
    siteReference="<SITE_NAME>"
    extensionVersion="2.0.10" # or latest Arc version
    extensionName="<PREFERRED_EXTENSION_NAME>"
    ```

1. Create the site address and site using the following commands:

    ```bash
    #Create site address
    az resource create --resource-type Microsoft.EdgeOrder/addresses --resource-group "$rg" --location "$l" --name "$siteName" --properties "@$siteAddressJson"
    
    #Create site 
    az rest --method PUT --uri "$siteUri" --body "@$siteJson"
    ```

### [PowerShell](#tab/powershell)

1. Create a JSON file for site address and contact details. Name the file `<SITE_NAME_ADDRESS>.json` and save it in the same directory as your CLI commands. The JSON file must contain the following information:

    ```json
       {
           "contactDetails": {
               "contactName": "<CONTACT_NAME>",
               "emailList": [
                   "<EMAIL_LIST>"        
               ],
               "phone": "<PHONE>",
               "phoneExtension": "<PHONE_EXTENSION>"
           },
           "shippingAddress": {
               "addressType": "<ADDRESS_TYPE>",
               "city": "<CITY>",
               "companyName": "<COMPANY_NAME>",
               "country": "<COUNTRY>",
               "postalCode": "<POSTAL_CODE>",
               "stateOrProvince": "<STATE_OR_PROVINCE>",
               "streetAddress1": "<STREET_ADDRESS_1>",
               "streetAddress2": "<STREET_ADDRESS_2>"
           }
        }
    ```

1. Create a JSON file for site and name it */<SITE_NAME>/.json*. The JSON file must contain the following information:

    ```json
        {
            "properties": {
                "description": "<DESCRIPTION>",
                "addressResourceId": "<ADDRESS_RESOURCE_ID>",
                "displayName": "<SITE_NAME>"
            }
        }
    ```

    > [!TIP]
    > You can refer to *redmond-site-address.json* and *redmond-site.json* files in the downloaded ZIP folder for examples of how to create the JSON files.

1. Define the global variables for site address and site. Replace the placeholder variables with the values from your JSON files.

    ```powershell
    $siteJson = "<SITE_NAME>.json"
    $siteAddressJson = "<SITE_NAME_ADDRESS>.json"
    $siteUri = "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$siteName?api-version=2024-02-01-preview"
    $siteId = "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$siteName"
    $siteReference = "<SITE_NAME>"
    $extensionVersion = "2.0.10" # or latest Arc version
    $extensionName = "<PREFERRED_EXTENSION_NAME>"
    ```

1. Create the site address and site using the following commands:

    ```powershell
    # Create site address
    az resource create --resource-type Microsoft.EdgeOrder/addresses --resource-group $rg --location $l --name $siteName --properties "@$siteAddressJson"
    
    # Create site
    az rest --method PUT --uri $siteUri --body "@$siteJson"
    ```

***

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

## Next steps

Once you have prepared the environment and the global variables, you can proceed to [Set up workload orchestration](initial-setup-configuration.md).
