---
title: Troubleshooting for Workload Orchestration
description: Learn how to troubleshoot issues with Workload Orchestration.
author: sethmanheim
ms.author: sethm
ms.topic: troubleshooting-general
ms.date: 06/01/2025
---

# Troubleshooting workload orchestration

This article provides troubleshooting guidance for common issues encountered when using workload orchestration in Azure Arc.

## Troubleshoot staging

The section troubleshoots issues related to [staging resources before deployment](how-to-stage.md) in workload orchestration.

### Issue: Insufficient disk space on Edge device

Description: The local edge storage is nearing capacity or runs out of available space.

Solution: To resolve this issue, you can free up space on the local edge storage by removing unnecessary files. Run the following command:

```powershell
az acr scope-map update -n $$connectedRegistryName -r $ acrName --remove-repo $unwanted_repo_name metadata/read content/read
```

### Issue: Missing or expired authentication token 

Description: The authentication token required to synchronize Azure Container Registry (ACR) with the connected registry is missing or expired.

Solution: To resolve this issue, try to uninstall and reinstall the connected registry. This action generates a new authentication token. 

1. Run the following command to uninstall the connected registry:

    ```powershell
    az aks show --resource-group $resourceGroup --name $arcCluster --query "networkProfile.serviceCidr" 
    ```

1. Check which IP ranges are in used by the connected registry service.

    ```powershell
    kubectl get services -A 
    ```

1. To host the connected registry service, pick an IP within the available IP range and not in use.
 
    ```powershell
    $available_ip = "<input the available IP address>" 
    ```

1. Configure the connection string for the connected registry to sync with ACR.  

    ```powershell
    connectionString = az acr connected-registry get-settings --name $connectedRegistryName --registry $acrName --parent-protocol https --generate-password 1 --query ACR_REGISTRY_CONNECTION_STRING --subscription $subId --output tsv --yes
    ```

1. Remove carriage return characters.

    ```powershell
    $connectionString = $connectionString -replace "`n", "" 
    ```

1. Create a valid JSON file and write it to the file.
 
    ```powershell
    @{ connectionString = $connectionString } | ConvertTo-Json | Out-File protected-settings-extension.json -Encoding utf8 
    ```
 
1. Open *protected-settings-extension.json* file in editor and check it. If the file is encoded with "UTF-8 with BOM", save it with "UTF-8". 
1. Install the connected registry extension.

    ```powershell
    connectedClusters --extension-type Microsoft.ContainerRegistry.ConnectedRegistry --name $connectedRegistryName --resource-group $resourceGroup --config service.clusterIP=$available_ip --config pvc.storageRequest=20Gi --config cert-manager.install=false --config-protected-file protected-settings-extension.json 

    # if you want to use a storage class other than the default one, add below flag: 
     #--config pvc.storageClassName=$<storage class name> 
    ```

1. Confirm the installation. Run this command and you should see three pods running, one for connected-registry and others for container on each node. 

    ```powershell
    kubectl get pods -n connected-registry 
    ```
 
1. Check the connected-registry state on ACR.

    ```powershell
    az acr connected-registry show --name $connectedRegistryName --registry $acrName --output table
    ```
 
### Issue: Missing or expired Kubernetes client token

Description: The client token required for Kubernetes to access the connected registry is missing or expired.

Solution: To resolve this issue, configure the client token to access the connected registry. 

1. Create a scope map for the connected registry to allow read access to all repositories.

    ```powershell
    az acr scope-map create --name "all-repos-read" --registry $acrName --repository "*" content/read metadata/read --description "Scope map for pulling from ACR." 
    
    az acr token create --name "all-repos-pull-token" --registry $acrName --scope-map "all-repos-read" 
    
    # Please store your generated credentials safely. It generates two passwords and either of the two can be used.
    az acr connected-registry update --name $connectedRegistryName --registry $acrName --add-client-token "all-repos-pull-token" 
    ```

1. You can encode the value with Base64 encoding to store it as a secret in Kubernetes.

    ```powershell
    [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("<tokenname/value>")) 
    ```

1. Save the secret in a file named *secret.yaml*.

    ```powershell
    kubectl apply -f secret.yaml -n $customLocationName 
    ```

### Issue: Limited network access

Description: The connected registry isn't accessible due to network latency.

Solution: To resolve this issue, retry publishing the solution.

```powershell
$reviewId = "<input the id acquired from previous step>" 

az workload-orchestration target publish --resource-group $rg --target-name $targetName         --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$targetName/solutions/$solutionName/versions/$solutionVersion
```

### Issue: Container image file is corrupted or missing 

Description: Deployment error says it can't find the correct images or image file is corrupted. 

Solution: To resolve this issue, check the ImageUri/repository uri in your solution template helm values and make sure it's the correct value. If you are sure your template is correct, go to the edge cluster and follow these steps to check if the image staged in local is correct.

1. Inspect the image in local storage.

    ```powershell
    docker login -u $client_token_name -p $client_token_password $connected_registry_service_IP 
    
    docker pull $connected_registry_service_IP:443/$local_image_path 
    
    docker inspect --format='{{index .RepoDigests 0}}' $local_image_path 
    ```

1. Compare the `sha256` digest of the local image with the one in the cloud registry. If they don't match, delete the local image and rerun the publishing of the job to restage the image.

### Issue: Could not resolve connected registry IP

Description: The connected registry IP address isn't resolvable, which can happen if the connected registry service is not running or the IP address is incorrect.

Solution: To resolve this issue, check if you installed connected registry correctly. Run the following command to check the status of the connected registry service.

```powershell
Kubectl get service –n connected-registry 
```

### Issue: ACR resource ID configuration error

Description: The ACR resource ID isn't configured correctly, which can happen if the ACR resource ID isn't set or is set to an incorrect value.

Solution: To resolve this issue, fix your solution template input. 

## Troubleshoot service groups 

For [service group](service-group.md) related resources, only the user who created the resource can fetch/GET that resources due to [RBAC limitations](rbac-guide.md).

To fetch service groups, run the following command.

### [Bash](#tab/bash)

```bash
az rest \
    --method post \
    --url "https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01" \
    --body "{'query': 'resourcecontainers | where type contains \"microsoft.management/serviceGroups\"'}" \
    --resource "https://management.azure.com"
```

### [PowerShell](#tab/powershell)

```powershell
az rest `
    --method post `
    --url https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01 `
    --body "{'query': 'resourcecontainers | where type contains \'microsoft.management/serviceGroups\''}" `
    --resource https://management.azure.com
```
***

To fetch service group Sites, run the following command.

### [Bash](#tab/bash)

```bash
az rest \
    --method post \
    --url "https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01" \
    --body "{'query': 'extensibilityresources | where type =~ \"microsoft.edge/sites\" | where id startswith \"/providers/microsoft.management/servicegroups\"'}" \
    --resource "https://management.azure.com"
```

### [PowerShell](#tab/powershell)

```powershell
az rest `
  --method post `
  --url https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01 `
  --body "{'query': 'extensibilityresources | where type =~ \'microsoft.edge/sites\' | where id startswith \'/providers/microsoft.management/servicegroups\''}" `
  --resource https://management.azure.com
```
***

To fetch service group-relationships, run the following command.

### [Bash](#tab/bash)

```bash
az rest \
    --method post \
    --url "https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01" \
    --body "{'query': 'relationshipresources | where type =~ \"microsoft.relationships/servicegroupmember\" | where id startswith \"$targetId\"'}" \
    --resource "https://management.azure.com"
```

### [PowerShell](#tab/powershell)

```powershell
az rest `
    --method post `
    --url https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01 `
    --body "{'query': 'relationshipresources | where type =~ \'microsoft.relationships/servicegroupmember\' | where id startswith \'$targetId\''}" `
    --resource https://management.azure.com
```
***

## Troubleshoot GitHub actions 

### Schema creation failures 

If you encounter issues while creating a schema in GitHub actions, consider the following troubleshooting steps:

- Verify schema file syntax. 
- Check if schema name/version already exists.
- Ensure Azure credentials have proper permissions.

### Template issues 

If you encounter issues with templates in GitHub actions, consider the following troubleshooting steps:

- Verify schema version exists.
- Check specification file JSON format.
- Validate capabilities in metadata.
- Check external validation settings.

### File detection issues 

If you encounter issues with file detection in GitHub actions, consider the following troubleshooting steps:

- Ensure files use correct naming patterns.
- Verify files are in the correct directories:
    - **Apps:** `.pg/apps/<app>/workload-orchestration/`
    - **Common:** `.pg/apps/common/`
- Check workflow logs for file detection output.

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]