---
title: Staging Resources Before Deployment
description: Learn how to stage resources before deployment in Azure Arc Workload Orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 07/10/2025
---

# Stage resources before deployment

Staging is a predeployment step that allows you to validate and download resources before they are deployed to the edge cluster. This process helps ensure that all configurations, images, and dependencies are correctly set up, ensuring reliable deployments within limited maintenance windows. 

Some common scenarios where staging is beneficial are:

- Users with factories located in remote areas where network latency can increase the risk of deployment failures.
- Users manage large-scale deployments, and downloading sizable container images from Azure Container Registry (ACR) during limited maintenance windows can be challenging.

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Set up Azure Container Registry (ACR)

To stage resources, you need to set up an Azure Container Registry (ACR) to store the container images and other artifacts required for your deployments. 

### [Bash](#tab/bash)

1. Create an ACR in the Azure portal with premium SKU.

    ```bash
    acrName="<acr_name>"
    rg="<resource_group>"

    az acr create --resource-group "$rg" --name "$acrName" --sku Premium
    az acr update --name "$acrName" --data-endpoint-enabled
    ```

1. Place any repository in the ACR. The connected registry requires at least one repository to be present in the ACR.

    ```bash
    # Login to the ACR for pulling operations
    az acr login --name "$acrName"
    docker pull hello-world
    docker tag hello-world:latest "$acrName.azurecr.io/hello-world:latest"
    docker push "$acrName.azurecr.io/hello-world:latest"
    ```

1. Create connected registry for the ACR.

    ```bash
    connectedRegistryName="<any connected registry name>" # leave "staging-temp" as default value, connected registry doesn't need this repo to be existing but it needs at least one repo.
    az acr connected-registry create --registry "$acrName" --name "$connectedRegistryName" --repository "staging-temp" --mode ReadOnly --log-level Debug --yes
    ```

1. Check the connected registry state on ACR. It should show as **Offline**.

    ```bash
    az acr connected-registry list --registry "$acrName" --output table # shows offline
    ```

1. Add **Contributor** and **Container Registry Contributor and Data Access Configuration Administrator** permissions to grant the service principal EdgeConfigurationManagerApp, with object ID `cba491bc-48c0-44a6-a6c7-23362a7f54a9`.

    ```bash
    az role assignment create --assignee "cba491bc-48c0-44a6-a6c7-23362a7f54a9" --role "Contributor" --scope --scope "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.ContainerRegistry/registries/$acrName"
    az role assignment create --assignee "cba491bc-48c0-44a6-a6c7-23362a7f54a9" --role "Container Registry Contributor and Data Access Configuration Administrator" --scope "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.ContainerRegistry/registries/$acrName"
    ```

1. Check available IP range on cluster to use for connected registry service.

    ```bash
    resourceGroup="<resource_group>"
    arcCluster="<arc_cluster>"

    az aks show --resource-group "$rg" --name "$arcCluster" --query "networkProfile.serviceCidr"
    # check IPs which are in use
    kubectl get services -A

    # Pick an IP which is within the available IP range and is not in use to host the connected registry service
    available_ip="<valid_IP>"
    ```

1. Configure a connection string for the connected registry and store it in a JSON file. This connection string is used to authenticate the local connected registry to the cloud.

    ```bash
    subId="<subscription_id>"

    connectionString=$(az acr connected-registry get-settings \
      --name "$connectedRegistryName" \
      --registry "$acrName" \
      --parent-protocol https \
      --generate-password 1 \
      --query ACR_REGISTRY_CONNECTION_STRING \
      --subscription "$subId" \
      --output tsv \
      --yes)
    # Remove carriage return characters (Linux/Mac)
    connectionString=$(echo "$connectionString" | tr -d '\r')
    # Create valid JSON and write it to the file
    echo "{\"connectionString\": \"$connectionString\"}" > protected-settings-extension.json
    ```

1. Open **protected-settings-extension.json** file in editor. If the file is encoded with "UTF-8 with BOM", change it to "UTF-8" and save the file.

1. Install connected registry CLI extension on your ARC cluster to enable staging.

    ```bash
    resourceGroup="<resource_group>"
    arcCluster="<arc_cluster>"

    az k8s-extension create --cluster-name "$arcCluster" --cluster-type connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name "$connectedRegistryName" --resource-group "$resourceGroup" --config service.clusterIP="$available_ip" --config pvc.storageRequest=20Gi --config cert-manager.install=false --config-protected-file protected-settings-extension.json
    # if you want to use a storage class other than the default one, add below flag: #--config pvc.storageClassName=<storage class name>

    # confirm installation successful: (you should see 3 pods running, one for connected-registry and others for containerd on each node)
    kubectl get pods -n connected-registry

    # check connected-registry state on ACR:
    az acr connected-registry list --registry "$acrName" --output table # shows online
    ```

    > [!NOTE]
    > If you use Tanzu Kubernetes Grid (TKG) clusters, you need to follow additional steps to install the connected registry CLI extension. See the [Install connected registry for Tanzu Kubernetes clusters](#install-connected-registry-for-tanzu-kubernetes-clusters-only-for-tkg-clusters) section.

1. (Optional) You can verify the installation status of the connected registry on [Azure portal](https://portal.azure.com/). Navigate to the **Azure Container Registry** resource, select **Connected Registries**, and check the status of your connected registry. It should show as **Online**.

1. Create a new client token or add an existing one to the connected registry.

    ```bash
    az acr scope-map create --name "all-repos-read" --registry "$acrName" --repository "staging-temp" content/read metadata/read --description "Scope map for pulling from ACR"

    az acr token create --name "all-repos-pull-token" --registry "$acrName" --scope-map "all-repos-read"

    az acr connected-registry update --name "$connectedRegistryName" --registry "$acrName" --add-client-token "all-repos-pull-token"
    ```

1. Save the client token in a k8s secret to use it later in a secret.yaml file. You can choose any of the two passwords generated by the previous command.

    ```yaml
    apiVersion: v1
    data:
        password: <base64-encoded-password>
        username: <base64-encoded-username>
    kind: Secret
    metadata:
        name: my-acr-secret
        namespace: default
    type: Opaque
    ```

    You can encode the password in base64 format using the following command:

    ```bash
    echo -n "<tokenname/value>" | base64
    ```

1.  Save the secret to cluster. 

    ```bash
    kubectl apply -f secret.yaml
    ```


### [PowerShell](#tab/powershell)

1. Create an ACR in the Azure portal with premium SKU.

    ```powershell
    $acrName = "<acr_name>"

    az acr create --resource-group $rg --name $acrName --sku Premium
    az acr update --name $acrName --data-endpoint-enabled
    ```

1. Place any repository in the ACR. The connected registry requires at least one repository to be present in the ACR.

    ```powershell
    # login to the ACR for pulling operations
    az acr login --name $acrName 
    docker pull hello-world
    docker tag hello-world:latest $acrName.azurecr.io/hello-world:latest
    docker push acrbb.azurecr.io/hello-world:latest
    ```

1. Create connected registry for the ACR.

    ```powershell 
    $connectedRegistryName = "<any connected registry name>"
    # leave "staging-temp" as default value, connected registry doesn't need this repo to be existing but it needs at least one repo.
    az acr connected-registry create --registry $acrName --name $connectedRegistryName --repository "staging-temp" --mode ReadOnly --log-level Debug --yes
    ```

1. Check the connected registry state on ACR. It should show as **Offline**.

    ```powershell
    az acr connected-registry list --registry $acrName --output table # shows offline
    ```

1. Add **Contributor** and **Container Registry Contributor and Data Access Configuration Administrator** permissions to grant the service principal EdgeConfigurationManagerApp, with object ID `cba491bc-48c0-44a6-a6c7-23362a7f54a9`.

    ```powershell
    az role assignment create --assignee "cba491bc-48c0-44a6-a6c7-23362a7f54a9" --role "Contributor" --scope "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.ContainerRegistry/registries/$acrName"
    az role assignment create --assignee "cba491bc-48c0-44a6-a6c7-23362a7f54a9" --role "Container Registry Contributor and Data Access Configuration Administrator" --scope "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.ContainerRegistry/registries/$acrName"
    ```

1. Check available IP range on cluster to use for connected registry service.

    ```powershell
    az aks show --resource-group $resourceGroup --name  $arcCluster --query "networkProfile.serviceCidr"
    # check IPs which are in use
    kubectl get services -A
    
    #Pick an IP which is within the available IP range and is not in use to host the connected registry service
    $available_ip = "<valid_IP>"
    ```

1. Configure a connection string for the connected registry and store it in a JSON file. This connection string is used to authenticate the local connected registry to the cloud.

    ```powershell
    $connectionString = az acr connected-registry get-settings `
    --name $connectedRegistryName `
    --registry $acrName `
    --parent-protocol https `
    --generate-password 1 `
    --query ACR_REGISTRY_CONNECTION_STRING `
    --subscription $subId `
    --output tsv `
    --yes
    # Remove carriage return characters (Windows)
    $connectionString = $connectionString -replace "`r", ""
    # Create valid JSON and write it to the file
    @{ connectionString = $connectionString } | ConvertTo-Json | Out-File protected-settings-extension.json -Encoding utf8
    ```

1. Open **protected-settings-extension.json** file in editor. If the file is encoded with "UTF-8 with BOM", change it to "UTF-8" and save the file. 

1. Install connected registry CLI extension on your ARC cluster to enable staging.

    ```powershell
    az k8s-extension create --cluster-name $arcCluster --cluster-type connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name $connectedRegistryName --resource-group $resourceGroup --config service.clusterIP=$available_ip --config pvc.storageRequest=20Gi --config cert-manager.install=false --config-protected-file protected-settings-extension.json
    # if you want to use a storage class other than the default one, add below flag: #--config pvc.storageClassName=$<storage class name>
    
    # confirm installation successful: (you should see 3 pods running, one for connected-registry and others for containerd on each node)
    kubectl get pods -n connected-registry
    
    # check connected-registry state on ACR:
    az acr connected-registry list --registry $acrName --output table # shows online
    ```

    > [!NOTE]
    > If you use Tanzu Kubernetes Grid (TKG) clusters, you need to follow additional steps to install the connected registry CLI extension. See the [Install connected registry for Tanzu Kubernetes clusters](#install-connected-registry-for-tanzu-kubernetes-clusters-only-for-tkg-clusters) section.


1. (Optional) You can verify the installation status of the connected registry on [Azure portal](https://portal.azure.com/). Navigate to the **Azure Container Registry** resource, select **Connected Registries**, and check the status of your connected registry. It should show as **Online**.

1. Create a new client token or add an existing one to the connected registry.

    ```powershell
    az acr scope-map create --name "all-repos-read" --registry $acrName --repository "staging-temp" content/read metadata/read --description "Scope map for pulling from ACR"

    az acr token create --name "all-repos-pull-token" --registry $acrName --scope-map "all-repos-read"

    az acr connected-registry update --name $connectedRegistryName --registry $acrName --add-client-token "all-repos-pull-token" 
    ```

1. Save the client token in a k8s secret to use it later in a secret.yaml file. You can choose any of the two passwords generated by the previous command.

    ```yaml
    apiVersion: v1
    data:
      password: <base64-encoded-password>
      username: <base64-encoded-username>
    kind: Secret
    metadata:
      name: my-acr-secret
      namespace: default
    type: Opaque
    ```

    You can encode the password in base64 format using the following command:

    ```powershell
    $encodedPassword = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("<tokenname/value>"))
    ```

1.  Save the secret to cluster. 

    ```powershell
    kubectl apply -f secret.yaml
    ```

***

### Install connected registry for Tanzu Kubernetes clusters (Only for TKG clusters)

If you are using Tanzu Kubernetes Grid (TKG) clusters, you need to follow these additional steps to install the connected registry CLI extension. Once the connected registry is installed, you can proceed with steps in the [Set up Azure Container Registry (ACR)](#set-up-azure-container-registry-acr) section.

#### [Bash](#tab/bash)

1. Add the privilege label to the connected registry namespace.

    ```bash
    kubectl label --overwrite ns connected-registry pod-security.kubernetes.io/enforce=privileged
    ```

1. Install the connected registry CLI extension. You must provide an IP within the valid IP range on Tanzu.

    ```bash
    az k8s-extension create --cluster-name "$cluster"  --cluster-type connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name "$storageName" --resource-group "$rg" --config service.clusterIP=<serviceIP> --config pvc.storageClassName=<storage_class_name> --config pvc.storageRequest=20Gi --config cert-manager.install=false --config-protected-file protected-settings-extension.json
    # you can find a valid storage class name in tanzu by the following command:
    # kubectl get sc
    ```

#### [PowerShell](#tab/powershell)

1. Add the privilege label to the connected registry namespace.

    ```powershell
    kubectl label --overwrite ns connected-registry pod-security.kubernetes.io/enforce=privileged
    ```

1. Install the connected registry CLI extension. You must provide an IP within the valid IP range on Tanzu.

   ```powershell
    az k8s-extension create --cluster-name $cluster  --cluster-type connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name $storageName --resource-group $rg --config service.clusterIP=<serviceIP> --config pvc.storageClassName=<storage_class_name> --config pvc.storageRequest=20Gi --config cert-manager.install=false --config-protected-file protected-settings-extension.json
    # you can find a valid storage class name in tanzu by the following command:
    # kubectl get sc
   ```

***

## Enable staging at solution level

To enable staging for a solution, you need to add the `staged` field to the `properties` field in the specs.json solution template file associated to a solution version. This field indicates that the solution should be staged before deployment. You need to specify the Azure Container Registry (ACR) resource ID and the image file paths.

```json
    properties: {
      "staged": {
        "acrResourceId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ContainerRegistry/registries/<acr-name>",
        "images": [
            "demo-image:latest"
        ]
      }
    }
```

Staging is triggered automatically once the solution is configured.

> [!NOTE]
> If you run into any issues while staging a solution, see the [Troubleshooting guide](troubleshooting.md#troubleshoot-staging).


## Prepare the solution template version for staging

### Upload an image

#### [Bash](#tab/bash)

1. Upload a container image to the Azure Container Registry (ACR) that you created in the previous step. 

    ```bash
    image="<image_name>"

    # Copy your existing file into the build context (optional, if not already there)
    cp /path/to/your/file ./bigfile
    # Build the Docker image
    docker build -t "$image" .
    # Tag the image for Azure Container Registry
    docker tag "$image" "$acrName.azurecr.io/$image:latest"
    # Push the image to ACR
    docker push "$acrName.azurecr.io/$image:latest"
    ```

1. You can confirm that the image is successfully uploaded.

    ```bash
    az acr repository list --name "$acrName" --output table
    ```

#### [PowerShell](#tab/powershell)

1. Upload a container image to the Azure Container Registry (ACR) that you created in the previous step.

    ```powershell
    export image = "<image_name>"
    
    # Copy your existing file into the build context (optional, if not already there)
    cp /path/to/your/file ./bigfile
    # Build the Docker image
    docker build -t $image .
    # Tag the image for Azure Container Registry
    docker tag $image $acrName.azurecr.io/$image:latest
    # Push the image to ACR
    docker push $acrName.azurecr.io/$image:latest
    ```

1. You can confirm that image is successfully uploaded.

    ```powershell
    az acr repository list --name $acrName --output table
    ```
***

### Create a target

#### [Bash](#tab/bash)

```bash
scopename="staging"
targetName="Line01"

az workload-orchestration target create \
    --resource-group "$rg" \
    --location "$l" \
    --name "$targetName" \
    --display-name "$targetName" \
    --hierarchy-level line \
    --capabilities "This is the capability" \
    --description "This is Line01 Site" \
    --solution-scope "$scopename" \
    --target-specification "@targetspecs.json" \
    --extended-location "@custom-location.json" \
    --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/private.edge/contexts/$contextName"
```

#### [PowerShell](#tab/powershell)

```powershell
$scopename = "staging"
$targetName = "Line01"

az workload-orchestration target create `
  --resource-group $rg `
  --location $l `
  --name $targetName `
  --display-name $targetName `
  --hierarchy-level line `
  --capabilities "This is the capability" `
  --description "This is Line01 Site" `
  --solution-scope $scopename `
  --target-specification '@targetspecs.json' `
  --extended-location '@custom-location.json' `
  --context-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/contexts/$contextName
```
***

### Create solution template

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "${resourcePrefix}-SS" --schema-file ./demo-app-schema.yaml -l "$l"
    ```

1. Make sure to set the correct schema name and input the image that you build in demo-app-config-template.yaml repository:

    ```yaml
    configs:
      image:
        repository: ${{$val(LocalConnectedRegistryIP)}}+/<image_name>
    ```

1. In spec.json, change the stage image to your own image:

    ```json
    "staged": {
        "acrResourceId": "<your_acr_resource_ID>",
        "images": [
            "<your-image-name>:<tag>"
        ]
    }
    ```

1. Create a solution template file.

    ```bash
    solutionTemplateName="Line01-Solution"

    az workload-orchestration solution-template create \
        --solution-template-name "$solutionTemplateName" \
        -g "$rg" \
        -l "$l" \
        --capabilities "This is the capability" \
        --description "This is Staging Solution" \
        --config-template-file "./demo-app-config-template.yaml" \
        --specification "@demo-app-spec.json" \
        --version "1.0.0"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "$resourcePrefix-SS" --schema-file .\demo-app-schema.yaml  -l $l
    ```

1. Make sure to set the correct schema name and input the image that you build in demo-app-config-template.yaml repository:

    ```yaml
    configs:
      image:
        repository: ${{$val(LocalConnectedRegistryIP)}}+/<image_name>
    ```

1. In spec.json, change the stage image to your own image:

    ```json
    "staged": {
        "acrResourceId": "<your_acr_resource_ID>",
        "images": [
            "<your-image-name>:<tag>"
        ]
    }
    ```

1. Create a solution template file.

    ```powershell
    $solutionTemplateName = "Line01-Solution"
    
    az workload-orchestration solution-template create `
        --solution-template-name $solutionTemplateName `
        -g $rg `
        -l $l `
        --capabilities "This is the capability" `
        --description "This is Staging Solution" `
        --config-template-file ".\demo-app-config-template.yaml" `  
        # --config-template ".\demo-app-config-template.yaml" `  # private RP
        --specification "@demo-app-spec.json" `
        --version "1.0.0" 
    ```
***

### Set configuration at target level

Specify the service IP address that you assigned to the connected registry service. This configuration step is required only once and does not need to be repeated for subsequent deployments.

#### [Bash](#tab/bash)

```bash
az workload-orchestration configuration set -g "$rg" --solution-template-name "$solutionTemplateName" --target-name "$targetName"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration configuration set -g $rg --solution-template-name $solutionTemplateName --target-name $targetName
```
***

### Resolve and review the template version

#### [Bash](#tab/bash)

Review the template version.

```bash
az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionTemplateName/versions/1.0.0 --resource-group "$rg" --target-name "$targetName"
```

#### [PowerShell](#tab/powershell)

Review the template version.

```powershell
az workload-orchestration target --solution-template-name $solutionTemplateName --solution-template-version 1.0.0 --resource-group $rg --target-name $targetName review
```
***

### Publish and install the solution

#### [Bash](#tab/bash)

1. Publish the template version.

    ```bash
    reviewId="<input the ID from previous step>"
    subId="<your subscription id>"

    az workload-orchestration target publish --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$targetName/solutions/$solutionName/versions/1.0.0 --resource-group "$rg" --target-name "$targetName"
    ```


1. Check the solution status. It should change from "inReview" to "staging".

    ```bash
    curl -H "Authorization: Bearer <access_token>" \
      "https://eastus2euap.management.azure.com/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$targetName/solutions/$solutionName/versions?api-version=2025-01-01-preview"
    ```

1. After publish completed, run the following commands to check that the images are staged locally:

    ```bash
    kubectl exec -it <connected_registry_pod> -n $cr -- bash
    # (inside the pod)
    # cd maestro-tmp
    # ./check-acr-images.sh "demo-image-0x:latest"
    ```

1. Install the solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$targetName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$targetName/solutions/$solutionName$/versions/1.0.0
    ```

1. After installation, check that the image is used in deployment.

    ```bash
    kubectl describe pod -n "$scopename"
    ```

#### [PowerShell](#tab/powershell)
1. Publish the template version.

    ```powershell
    $reviewId = "<input the ID from previous step>"
    $subId = "<your subscription id>"
    az workload-orchestration target publish --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$targetName/solutions/$solutionName/versions/1.0.0 --resource-group $rg --target-name $targetName
    ```

1. Check the solution status. It should change from "inReview" to "staging".

    ```powershell
    
    armclient get "https://eastus2euap.management.azure.com/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$targetName/solutions/$solutionName/versions?api-version=2025-01-01-preview"
    ```

1. After publish completed, run the following commands to check that the images are staged in local:

    ```powershell
    kubectl exec -it <connected_registry_pod> -n $cr -- bash
    # (inside the pod)
    # cd maestro-tmp
    # ./check-acr-images.sh "demo-image-0x:latest"
    ```

1. Install the solution.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $targetName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$targetName/solutions/$solutionName$/versions/1.0.0
    ```

1. After installation, check that the image is used in deployment.

    ```powershell
    kubectl describe pod -n $scopename
    ```
***

## View staged resources

You can view staging details in the **Published solutions** tab of the [workload orchestration portal](configure.md#view-the-published-solutions). 

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview) and go to the **Configure** tab on the left side of the page.
1. Select the **Published solutions** tab.
1. Choose a solution with status **Publishing in progress** and click on the alert icon to view the details of the staging process.

    :::image type="content" source="./media/staging-published-solutions.png" alt-text="Screenshot of the Published solutions tab in the workload orchestration portal." lightbox="./media/staging-published-solutions.png":::

1. The status of the staging process is displayed. If the staging is successful, you see a message indicating that images are downloaded to the edge cluster. 

    :::image type="content" source="./media/staging-downloading.png" alt-text="Screenshot of the successful staging status in the workload orchestration portal." lightbox="./media/staging-downloading.png":::

1. If the staging fails, you see an error message indicating the failure. For more information, contact your IT administrator.

    :::image type="content" source="./media/staging-failure.png" alt-text="Screenshot of the failed staging status in the workload orchestration portal." lightbox="./media/staging-failure.png":::


## Related content

- [Monitor your solutions with Azure portal](azure-portal-monitoring.md)
- [Diagnose of problems](diagnose-problems.md)
- [Service groups](service-group.md)
- [Bulk deployment](bulk-deployment.md)
