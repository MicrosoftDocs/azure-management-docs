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
    az acr create --resource-group $RG --name $ACR_NAME --sku Premium
    ```

1. Create connected registry for the ACR.

    ```bash
    az acr update --name $ACR_NAME --data-endpoint-enabled

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
    az acr create --resource-group $rg --name $acrName --sku Premium
    ```

1. Create connected registry for the ACR.

    ```powershell
    az acr update --name $acrName --data-endpoint-enabled
    
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