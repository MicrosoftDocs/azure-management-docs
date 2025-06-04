---
title: Staging Resources Before Deployment
description: Learn how to stage resources before deployment in Azure Arc Workload Orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/02/2025
---

# Stage resources before deployment

Staging is a pre-deployment step that allows you to validate and download resources before they are deployed to the edge cluster. This process helps ensure that all configurations, images, and dependencies are correctly set up, ensuring reliable deployments within limited maintenance windows. 

Some common scenarios where staging is beneficial are:

- Users with factories located in remote areas where network latency can increase the risk of deployment failures.
- Users manage large-scale deployments, and downloading sizable container images from Azure Container Registry (ACR) during limited maintenance windows can be challenging.

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Set up Azure Container Registry (ACR)

To stage resources, you need to set up an Azure Container Registry (ACR) to store the container images and other artifacts required for your deployments. 

### [Bash](#tab/bash)

1. Create an ACR in the Azure portal with premium SKU to ensure high availability and performance.

    ```bash
    $ACR_NAME = "<acr_name>"

    az acr create --resource-group $RG --name $ACR_NAME --sku Premium
    az acr update --name $ACR_NAME --data-endpoint-enabled
    ```

1. Create connected registry for the ACR.

    ```bash
    az acr connected-registry create --registry $ACR_NAME --name $CONNECTED_REGISTRY_NAME --repository "staging-temp" --mode ReadOnly --log-level Debug --yes

    az acr connected-registry list --registry $ACR_NAME --output table # shows offline
    ```

1. Go to [Azure portal](https://portal.azure.com/), navigate to the **Azure Container Registry** resource, select **Connected Registries**, and Click on the **Connected register** you just created.
1. In the new pane, under **Sync properties**, click on the connected registry name.

    :::image type="content" source="./media/staging-token.png" alt-text="Screenshot of the Azure portal showing how to open the token pane." lightbox="./media/staging-token.png":::

1. Click on **Password1** or **Password2** to generate a token. Save the password and the name of the token, which will be used in connection string.

    :::image type="content" source="./media/staging-password.png" alt-text="Screenshot of the Azure portal showing how to generate and copy a new token." lightbox="./media/staging-password.png":::

1. Create a **protected-settings-extension.json** file which contains the `connectionString` to authenticate the local connected registry to the cloud. Replace the placeholders with your values:

    ```json
    {
      "connectionString": "ConnectedRegistryName=<cr_name>;SyncTokenName=<token_name>;SyncTokenPassword=<password>;ParentGatewayEndpoint=<acr_name>.eastus.data.azurecr.io;ParentEndpointProtocol=https"
    }
    ```

1. Install connected registry CLI extension on your ARC cluster to enable staging.

    ```bash
    az k8s-extension create --resource-group $RG --cluster-name $ARC_CLUSTER --name "<name>" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager

    az k8s-extension create --cluster-name $ARC_CLUSTER --cluster-type connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name $CONNECTED_CR_NAME --resource-group $RG --config service-clusterIP=$VALID_IP --config pvc-storageClassName=$STORAGE_CLASS --config pvc.storageRequest=$STORAGE_CAPACITY --config cert-manager.install=false --config-protected-file protected-settings-extension.json

    az acr connected-registry list --registry $ACR_NAME --output table # shows online
    ```

1. (Optional) You can verify the installation status of the connected registry on Azure portal. To do this, navigate to the **Azure Container Registry** resource, select **Connected Registries**, and check the status of your connected registry. It should show as **Online**.

1. Create a new client token or add an existing one to the connected registry.

    ```bash
    az acr scope-map create --name "all-repos-read" --registry $ACR_NAME --repository "staging-temp" content/read metadata/read --description "Scope map for pulling from ACR"

    az acr token create --name "all-repos-pull-token" --registry $ACR_NAME --scope-map "all-repos-read"

    az acr connected-registry update --name $CONNECTED_REGISTRY_NAME --registry $ACR_NAME --add-client-token "all-repos-pull-token"
    ```

1. Save the client token in a k8s secret to use it later by referring to it using `$SECRET`.

    ```bash
    kubectl create secret generic $SECRET --from-literal=acr-username=admin --from-literal=acr-password="<token password>"
    ```

### [PowerShell](#tab/powershell)

1. Create an ACR in the Azure portal with premium SKU to ensure high availability and performance.

    ```powershell
    $acrName="<acr_name>"

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
    az acr connected-registry list --registry $acrName --output table # shows offline
    ```

1. Go to [Azure portal](https://portal.azure.com/), navigate to the **Azure Container Registry** resource, select **Connected Registries**, and Click on the **Connected register** you just created.
1. In the new pane, under **Sync properties**, click on the connected registry name.

    :::image type="content" source="./media/staging-token.png" alt-text="Screenshot of the Azure portal showing how to open the token pane." lightbox="./media/staging-token.png":::

1. Click on **Password1** or **Password2** to generate a token. Save the password and the name of the token, which will be used in connection string.

    :::image type="content" source="./media/staging-password.png" alt-text="Screenshot of the Azure portal showing how to generate and copy a new token." lightbox="./media/staging-password.png":::

1. Create a **protected-settings-extension.json** file which contains the `connectionString` to authenticate the local connected registry to the cloud. Replace the placeholders with your values:

    ```json
    {
      "connectionString": "ConnectedRegistryName=<cr_name>;SyncTokenName=<token_name>;SyncTokenPassword=<password>;ParentGatewayEndpoint=<acr_name>.eastus.data.azurecr.io;ParentEndpointProtocol=https"
    }
    ```

1. Install connected registry CLI extension on your ARC cluster to enable staging.

    ```powershell
    az k8s-extension create --resource-group $rg --cluster-name $arcCluster --name "<name>" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager

    az k8s-extension create --cluster-name $arcCluster --cluster-type connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name $connectedCrName --resource-group $rg --config service-clusterIP=$valid_IP --config pvc-storageClassName=$storageClass --config pvc.storageRequest=$storage_capacity --config cert-manager.install=false --config-protected-file protected-settings-extension.json 

    az acr connected-registry-list --registry $acrName --output table # shows online
    ```

1. (Optional) You can verify the installation status of the connected registry on [Azure portal](https://portal.azure.com/). To do this, navigate to the **Azure Container Registry** resource, select **Connected Registries**, and check the status of your connected registry. It should show as **Online**.

1. Create a new client token or add an existing one to the connected registry.

    ```powershell
    az acr scope-map create --name "all-repos-read" --registry $acrName --repository "staging-temp" content/read metadata/read --description "Scope map for pulling from ACR"

    az acr token create --name "all-repos-pull-token" --registry $acrName --scope-map "all-repos-read"

    az acr connected-registry update --name $connectedRegistryName --registry $acrName --add-client-token "all-repos-pull-token" 
    ```

1. Save the client token in a k8s secret to use it later by referring to it using `$secret`.

    ```powershell
    kubectl create secret generic $secret --from-literal=acr-username=admin --from-literal=acr-password="<token password>"
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

1. Upload a container image to the Azure Container Registry (ACR) that you created in the previous step. This image will be used during the staging process.

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

1. Upload a container image to the Azure Container Registry (ACR) that you created in the previous step. This image will be used during the staging process.

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
    --extended-location "@custom-location.json"
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
  --extended-location '@custom-location.json'
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
        --configuration-template-file "./demo-app-config-template.yaml" \
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
        --configuration-template-file ".\demo-app-config-template.yaml" `  
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

1. Resolve the solution template version.

    ```bash
    az workload-orchestration target resolve --solution-template-name "$solutionTemplateName" --solution-template-version "1.0.0" --resource-group "$rg" --target-name "$targetName"
    ```

1. Review the template version.

    ```bash
    az workload-orchestration target review --solution-template-name "$solutionTemplateName" --solution-template-version "1.0.0" --resource-group "$rg" --target-name "$targetName"
    ```

#### [PowerShell](#tab/powershell)

1. Resolve the solution template version.

    ```powershell
    az workload-orchestration target resolve  --solution-template-name $solutionTemplateName --solution-template-version 1.0.0 --resource-group $rg --target-name $targetName
    ```

1. Review the template version.

    ```powershell
    az workload-orchestration target --solution-template-name $solutionTemplateName --solution-template-version 1.0.0 --resource-group $rg --target-name $targetName review
    ```
***

### Publish and install the solution

#### [Bash](#tab/bash)

1. Publish the template version.

    ```bash
    reviewId="<input the ID from previous step>"
    az workload-orchestration target publish --solution-name "$solutionName" --solution-version "1.0.0" --review-id "$reviewId" --resource-group "$rg" --target-name "$targetName"
    ```

1. Check the solution status. It should change from "inReview" to "staging".

    ```bash
    subscriptionId="<your subscription id>"
    curl -H "Authorization: Bearer <access_token>" \
      "https://eastus2euap.management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$targetName/solutions/$solutionName/versions?api-version=2025-01-01-preview"
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
    az workload-orchestration target install --solution-name "$solutionName" --solution-version "1.0.0" --resource-group "$rg" --target-name "$targetName"
    ```

1. After installation, check that the image is used in deployment.

    ```bash
    kubectl describe pod -n "$scopename"
    ```


#### [PowerShell](#tab/powershell)
1. Publish the template version.

    ```powershell
    $reviewId = "<input the ID from previous step>"
    az workload-orchestration target publish  --solution-name $solutionName --solution-version "1.0." --review-id $reviewId --resource-group $rg --target-name $targetName
    ```

1. Check the solution status. It should change from "inReview" to "staging".

    ```powershell
    $subscriptionId = "<your subscription id>"
    armclient get "https://eastus2euap.management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$targetName/solutions/$solutionName/versions?api-version=2025-01-01-preview"
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
    az workload-orchestration target install  --solution-name $solutionName --solution-version "1.0.0" --resource-group $rg --target-name $targetName
    ```

1. After installation, check that the image is used in deployment.

    ```powershell
    kubectl describe pod -n $scopename
    ```
***



















## View staged resources

You can view staging details in the **Published solutions** tab of the [workload orchestration portal](configure.md#view-the-published-solutions). 

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview) and go to the **Configure** tab on the left side of the page.
1. Select the **Published solutions** tab, which lists all the solutions that have been published in your environment.
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