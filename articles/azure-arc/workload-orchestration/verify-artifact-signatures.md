---
title: Verify deployment artifact signatures
description: Configure Workload Orchestration to verify Helm charts, YAML files, scripts, and container images before deployment.
author: nathmanish
ms.author: nathmanish
ms.topic: how-to
ms.date: 08/31/2026
ai-usage: ai-assisted
# Customer intent: As a platform engineer, I want to verify deployment artifacts so that only trusted software runs on my edge clusters.
---

# Verify deployment artifact signatures

Workload Orchestration can verify that deployment artifacts are signed by a trusted key or identity before the artifacts run on your edge clusters. Verification protects against artifacts that are unsigned, modified after signing, or signed by an untrusted identity.

Workload Orchestration provides two verification layers:

| Layer | Verifies | Enforcement point |
| --- | --- | --- |
| Layer 1 | Helm charts, remote Kubernetes YAML files, and scripts | The Workload Orchestration agent verifies each artifact before the provider applies or runs it. |
| Layer 2 | Container images | Ratify and Gatekeeper verify each image when Kubernetes admits a pod. |

You can enable either layer independently. Use both layers for end-to-end verification of deployment artifacts and the images that they reference.

> [!IMPORTANT]
> Keyless signatures use different formats in each layer. Layer 1 requires the new Sigstore bundle format. Ratify 1.4.x in Layer 2 requires the legacy cosign format. Use the commands shown in the relevant section.

## Prerequisites

Before you begin, make sure that you have:

- An Azure Arc-enabled Kubernetes cluster with Workload Orchestration extension version 2.1.41 or later. For setup instructions, see [Set up workload orchestration](set-up-workload-orchestration.md).
- Azure CLI with the `k8s-extension` and `connectedk8s` extensions.
- cosign 2.6 or later.
- Helm 3.8 or later for Helm chart and container image verification.
- Permission to create role assignments for the resources used in this article.

## Define the variables

The examples in this article use PowerShell. Set the shared variables once:

```powershell
$subId = "<subscription-id>"
$rg = "<resource-group>"
$location = "<azure-region>"
$clusterName = "<arc-connected-cluster-name>"
$extensionName = "<workload-orchestration-extension-name>"
$workloadOrchestrationNamespace = "<arc-connected-cluster-namespace>"

az account set --subscription $subId

$workloadOrchestrationPrincipalId = az k8s-extension show `
    --cluster-name $clusterName `
    --resource-group $rg `
    --cluster-type connectedClusters `
    --name $extensionName `
    --query identity.principalId `
    --output tsv
```

## Configure Layer 1 verification

Layer 1 verifies artifacts that the `helm.v3`, `yaml.k8s`, and script providers download. Choose one trust mode for all Layer 1 artifacts:

- **Certificate**: Sign with an Azure Key Vault key. The extension identity verifies the signature by using Key Vault.
- **Keyless**: Sign through an OpenID Connect (OIDC) provider. The extension trusts the exact issuer and subject in the signing certificate.

### Prepare the trust source

### [Certificate](#tab/certificate)

1. Create an Azure Key Vault and signing key. Key Vault names must be globally unique.

    ```powershell
    $keyVaultName = "<key-vault-name>"
    $keyName = "workload-signing-key"

    az keyvault create `
        --resource-group $rg `
        --name $keyVaultName `
        --location $location `
        --enable-rbac-authorization true

    $keyVaultId = az keyvault show `
        --resource-group $rg `
        --name $keyVaultName `
        --query id `
        --output tsv
    $keyVaultUri = az keyvault show `
        --resource-group $rg `
        --name $keyVaultName `
        --query properties.vaultUri `
        --output tsv
    $signedInUserId = az ad signed-in-user show --query id --output tsv

    az role assignment create `
        --assignee-object-id $signedInUserId `
        --assignee-principal-type User `
        --role "Key Vault Crypto Officer" `
        --scope $keyVaultId

    az keyvault key create `
        --vault-name $keyVaultName `
        --name $keyName `
        --kty EC `
        --curve P-256 `
        --ops sign verify

    $keyReference = "azurekms://$keyVaultName.vault.azure.net/$keyName"
    ```

1. Allow the Workload Orchestration extension identity to verify signatures.

    ```powershell
    az role assignment create `
        --assignee-object-id $workloadOrchestrationPrincipalId `
        --assignee-principal-type ServicePrincipal `
        --role "Key Vault Crypto User" `
        --scope $keyVaultId
    ```

### [Keyless](#tab/keyless)

Keyless mode doesn't require a key resource. When you sign the artifact in the next section, sign in through the OIDC provider that your organization trusts.

***

### Sign an artifact

Use the tab for the provider in your solution template.

### [Helm chart](#tab/helm-chart)

1. Push the packaged chart to an Azure Container Registry (ACR), and grant the extension identity permission to pull it.

    ```powershell
    $containerRegistryName = "<container-registry-name>"
    $chartPackage = "<chart-package.tgz>"

    az acr create `
        --resource-group $rg `
        --name $containerRegistryName `
        --location $location `
        --sku Standard

    $containerRegistryId = az acr show `
        --resource-group $rg `
        --name $containerRegistryName `
        --query id `
        --output tsv
    $containerRegistryLoginServer = az acr show `
        --resource-group $rg `
        --name $containerRegistryName `
        --query loginServer `
        --output tsv

    az role assignment create `
        --assignee-object-id $workloadOrchestrationPrincipalId `
        --assignee-principal-type ServicePrincipal `
        --role AcrPull `
        --scope $containerRegistryId

    az acr login --name $containerRegistryName
    helm push $chartPackage "oci://$containerRegistryLoginServer/charts"

    $chartReference = "$containerRegistryLoginServer/charts/<chart-name>:<chart-version>"
    ```

1. Sign the chart by using the trust mode you configured.

    ```powershell
    # Certificate mode
    cosign sign --key $keyReference --tlog-upload=false $chartReference

    # Keyless mode. Complete the browser sign-in when prompted.
    cosign sign --new-bundle-format $chartReference
    ```

### [Remote YAML](#tab/remote-yaml)

1. Create private blob storage for the YAML and its detached signature. Grant the extension identity permission to read the blobs.

    ```powershell
    $storageAccountName = "<storage-account-name>"
    $yamlFile = "app.yaml"

    az storage account create `
        --resource-group $rg `
        --name $storageAccountName `
        --location $location `
        --sku Standard_LRS `
        --allow-blob-public-access false

    $storageAccountId = az storage account show `
        --resource-group $rg `
        --name $storageAccountName `
        --query id `
        --output tsv

    az storage container create `
        --account-name $storageAccountName `
        --name artifacts `
        --auth-mode login

    az role assignment create `
        --assignee-object-id $workloadOrchestrationPrincipalId `
        --assignee-principal-type ServicePrincipal `
        --role "Storage Blob Data Reader" `
        --scope $storageAccountId
    ```

1. Sign and upload the YAML by using the trust mode you configured.

    ```powershell
    # Certificate mode
    cosign sign-blob `
        --key $keyReference `
        --tlog-upload=false `
        --output-signature "$yamlFile.sig" `
        $yamlFile
    $signatureFile = "$yamlFile.sig"

    # Keyless mode. Use these two lines instead of the certificate commands.
    # cosign sign-blob --new-bundle-format --yes --bundle "$yamlFile.bundle" $yamlFile
    # $signatureFile = "$yamlFile.bundle"

    az storage blob upload `
        --account-name $storageAccountName `
        --container-name artifacts `
        --auth-mode login `
        --file $yamlFile `
        --name $yamlFile `
        --overwrite
    az storage blob upload `
        --account-name $storageAccountName `
        --container-name artifacts `
        --auth-mode login `
        --file $signatureFile `
        --name $signatureFile `
        --overwrite

    $yamlUrl = "https://$storageAccountName.blob.core.windows.net/artifacts/$yamlFile"
    ```

    Set the component's `properties.yaml` value to `$yamlUrl`. The provider reads the signature from `<url>.sig` in certificate mode or `<url>.bundle` in keyless mode.

### [Scripts](#tab/scripts)

1. Create private blob storage and grant **Storage Blob Data Reader** as shown on the **Remote YAML** tab.

1. Sign every script in the provider's `scriptFolder`, and then upload each script and signature.

    ```powershell
    $scriptFiles = @("deploy.sh", "remove.sh", "get.sh")

    foreach ($scriptFile in $scriptFiles) {
        # Certificate mode
        cosign sign-blob `
            --key $keyReference `
            --tlog-upload=false `
            --output-signature "$scriptFile.sig" `
            $scriptFile
        $signatureFile = "$scriptFile.sig"

        # Keyless mode. Use these two lines instead of the certificate commands.
        # cosign sign-blob --new-bundle-format --yes --bundle "$scriptFile.bundle" $scriptFile
        # $signatureFile = "$scriptFile.bundle"

        az storage blob upload `
            --account-name $storageAccountName `
            --container-name artifacts `
            --auth-mode login `
            --file $scriptFile `
            --name $scriptFile `
            --overwrite
        az storage blob upload `
            --account-name $storageAccountName `
            --container-name artifacts `
            --auth-mode login `
            --file $signatureFile `
            --name $signatureFile `
            --overwrite
    }

    $scriptFolder = "https://$storageAccountName.blob.core.windows.net/artifacts"
    ```

    Set the script provider's `scriptFolder` value to `$scriptFolder`.

***

### Enable Layer 1 on the extension

### [Certificate](#tab/enable-certificate)

```powershell
az k8s-extension update `
    --cluster-name $clusterName `
    --resource-group $rg `
    --cluster-type connectedClusters `
    --name $extensionName `
    --config "signing.enabled=true" `
    --config "signing.mode=cert" `
    --config "signing.trustVaultUri=$keyVaultUri" `
    --config "signing.keyNames=$keyName"
```

### [Keyless](#tab/enable-keyless)

Record the issuer and subject that cosign reports after signing the artifact. Don't infer these values from your Azure account because the certificate values depend on the selected identity provider.

```powershell
$keylessIssuer = "<issuer-from-cosign-certificate>"
$keylessSubject = "<subject-from-cosign-certificate>"

az k8s-extension update `
    --cluster-name $clusterName `
    --resource-group $rg `
    --cluster-type connectedClusters `
    --name $extensionName `
    --config "signing.enabled=true" `
    --config "signing.mode=keyless" `
    --config "signing.keylessIssuer=$keylessIssuer" `
    --config "signing.keylessSubject=$keylessSubject"
```

***

### Validate Layer 1

Deploy the solution through your normal Workload Orchestration workflow. Then confirm that verification ran:

```powershell
kubectl logs `
    --namespace $workloadOrchestrationNamespace `
    deployment/symphony-api | `
    Select-String -Pattern "(chart|YAML|script) signature verification (succeeded|skipped)"
```

The following results confirm that Layer 1 is enforcing your trust configuration:

| Test | Expected result |
| --- | --- |
| Deploy a correctly signed artifact. | Deployment succeeds, and the log contains `signature verification succeeded`. |
| Modify an artifact after signing. | Deployment fails with error `50100`, signature mismatch. |
| Remove the `.sig` or `.bundle` file, or use an unsigned chart. | Deployment fails with error `50101`, signature not found. |
| Sign with a different OIDC identity. | Deployment fails with error `50111`, identity mismatch. |

## Configure Layer 2 verification

Layer 2 uses Ratify and the Azure Policy add-on's Gatekeeper installation to verify container images at pod admission. The order in this section is required.

> [!CAUTION]
> After you remove the default Ratify configuration objects, don't upgrade or reinstall the Ratify Helm release. The chart recreates those objects and can remove the service account annotation. If you reinstall Ratify, repeat the annotation and deletion steps before updating the Workload Orchestration extension.

### Prepare the cluster and image

1. Confirm that OIDC issuer and workload identity are enabled on the Arc-enabled cluster and that Gatekeeper is running with external data enabled.

    ```powershell
    $oidcIssuer = az connectedk8s show `
        --resource-group $rg `
        --name $clusterName `
        --query oidcIssuerProfile.issuerUrl `
        --output tsv

    kubectl get deployment --namespace gatekeeper-system
    ```

    If `$oidcIssuer` is empty, enable the required features:

    ```powershell
    az connectedk8s update `
        --resource-group $rg `
        --name $clusterName `
        --enable-oidc-issuer `
        --enable-workload-identity

    $oidcIssuer = az connectedk8s show `
        --resource-group $rg `
        --name $clusterName `
        --query oidcIssuerProfile.issuerUrl `
        --output tsv
    ```

1. Set the Layer 2 variables. Use an image that's already stored in ACR, and ensure that your cluster nodes can pull it.

    ```powershell
    $containerRegistryName = "<container-registry-name>"
    $imageReference = "<registry-name>.azurecr.io/<repository>@sha256:<digest>"
    $gatekeeperNamespace = "gatekeeper-system"
    $ratifyServiceAccount = "ratify-admin"
    $ratifyIdentityName = "id-ratify"
    $providerName = "symphony-ratify-provider"
    $tenantId = az account show --query tenantId --output tsv

    $containerRegistryId = az acr show `
        --resource-group $rg `
        --name $containerRegistryName `
        --query id `
        --output tsv

    $signedInUserId = az ad signed-in-user show --query id --output tsv
    az role assignment create `
        --assignee-object-id $signedInUserId `
        --assignee-principal-type User `
        --role AcrPush `
        --scope $containerRegistryId
    ```

1. Sign the image in the legacy format required by Ratify 1.4.x. Record the issuer and subject reported by cosign.

    ```powershell
    $registryToken = az acr login `
        --name $containerRegistryName `
        --expose-token `
        --query accessToken `
        --output tsv

    cosign sign `
        --new-bundle-format=false `
        --yes `
        --registry-username "00000000-0000-0000-0000-000000000000" `
        --registry-password $registryToken `
        $imageReference

    $keylessIssuer = "<issuer-from-cosign-certificate>"
    $keylessSubject = "<subject-from-cosign-certificate>"
    ```

### Install and configure Ratify

1. Create a user-assigned managed identity for Ratify, grant it pull access to ACR, and federate it with the Ratify service account.

    ```powershell
    az identity create `
        --resource-group $rg `
        --name $ratifyIdentityName `
        --location $location

    $ratifyClientId = az identity show `
        --resource-group $rg `
        --name $ratifyIdentityName `
        --query clientId `
        --output tsv
    $ratifyPrincipalId = az identity show `
        --resource-group $rg `
        --name $ratifyIdentityName `
        --query principalId `
        --output tsv

    az role assignment create `
        --assignee-object-id $ratifyPrincipalId `
        --assignee-principal-type ServicePrincipal `
        --role AcrPull `
        --scope $containerRegistryId

    az identity federated-credential create `
        --resource-group $rg `
        --identity-name $ratifyIdentityName `
        --name ratify-federated-credential `
        --issuer $oidcIssuer `
        --subject "system:serviceaccount:${gatekeeperNamespace}:${ratifyServiceAccount}" `
        --audience api://AzureADTokenExchange
    ```

1. Install Ratify with mutation disabled. The Azure Policy add-on blocks the Gatekeeper mutation objects that the chart otherwise creates.

    ```powershell
    helm repo add ratify https://ratify-project.github.io/ratify
    helm repo update
    helm install ratify ratify/ratify `
        --namespace $gatekeeperNamespace `
        --version 1.15.6 `
        --set azureWorkloadIdentity.clientId=$ratifyClientId `
        --set serviceAccount.name=$ratifyServiceAccount `
        --set provider.enableMutation=false `
        --wait `
        --timeout 180s
    ```

1. Annotate the service account, restart Ratify, and remove the chart's default configuration objects. Workload Orchestration creates its own `symphony-*` objects.

    ```powershell
    kubectl annotate serviceaccount $ratifyServiceAccount `
        --namespace $gatekeeperNamespace `
        "azure.workload.identity/client-id=$ratifyClientId" `
        --overwrite
    kubectl rollout restart deployment/ratify --namespace $gatekeeperNamespace
    kubectl rollout status deployment/ratify `
        --namespace $gatekeeperNamespace `
        --timeout 120s

    kubectl delete provider.externaldata.gatekeeper.sh `
        ratify-provider ratify-mutation-provider `
        --ignore-not-found
    kubectl delete store.config.ratify.deislabs.io `
        store-oras `
        --ignore-not-found
    kubectl delete verifier.config.ratify.deislabs.io `
        verifier-cosign verifier-notation `
        --ignore-not-found

    $ratifyCaBundle = kubectl get secret ratify-tls `
        --namespace $gatekeeperNamespace `
        --output "jsonpath={.data.ca\.crt}"
    ```

### Enable Layer 2 on the extension

Update the extension only after Ratify is healthy and the default objects are removed:

```powershell
az k8s-extension update `
    --cluster-name $clusterName `
    --resource-group $rg `
    --cluster-type connectedClusters `
    --name $extensionName `
    --config "signing.enabled=true" `
    --config "signing.mode=keyless" `
    --config "signing.keylessIssuer=$keylessIssuer" `
    --config "signing.keylessSubject=$keylessSubject" `
    --config "signing.imageVerification.enabled=true" `
    --config "signing.imageVerification.manageRatify=check" `
    --config "signing.imageVerification.providerName=$providerName" `
    --config "signing.imageVerification.tenantId=$tenantId" `
    --config "signing.imageVerification.clientId=$ratifyClientId" `
    --config "signing.imageVerification.ratifyCABundle=$ratifyCaBundle" `
    --config "signing.imageVerification.policyDelivery=azurePolicy"
```

Confirm that the extension created the Ratify configuration:

```powershell
kubectl get store.config.ratify.deislabs.io,verifier.config.ratify.deislabs.io --all-namespaces
kubectl get provider.externaldata.gatekeeper.sh $providerName
```

### Assign the admission policy

Enabling Layer 2 creates the Ratify configuration but doesn't enforce admission. Assign the Workload Orchestration Ratify policy supplied with your release to the cluster.

> [!IMPORTANT]
> On a dual-registered AKS cluster, assign the policy to the AKS managed cluster resource. Assigning it to the Arc connected cluster doesn't enable enforcement.

1. Determine the policy scope and register the policy definition. Set `$policyFile` to the policy definition supplied with your Workload Orchestration release.

    ```powershell
    $arcClusterId = az connectedk8s show `
        --resource-group $rg `
        --name $clusterName `
        --query id `
        --output tsv
    $aksClusterId = az resource list `
        --name $clusterName `
        --resource-type Microsoft.ContainerService/managedClusters `
        --query "[0].id" `
        --output tsv
    $policyScope = if ($aksClusterId) { $aksClusterId } else { $arcClusterId }

    $policyDefinitionName = "workload-orchestration-ratify-verification"
    $policyFile = "<path-to-ratify-policy-definition.json>"

    az rest `
        --method put `
        --url "https://management.azure.com/subscriptions/$subId/providers/Microsoft.Authorization/policyDefinitions/${policyDefinitionName}?api-version=2023-04-01" `
        --resource https://management.azure.com/ `
        --headers "Content-Type=application/json" `
        --body "@$policyFile"

    $policyDefinitionId = az policy definition show `
        --name $policyDefinitionName `
        --query id `
        --output tsv
    ```

1. Assign the policy in `Audit` mode first. The Azure Policy add-on can take about 15 minutes to create the Gatekeeper constraint.

    ```powershell
    $policyAssignmentName = "workload-orchestration-ratify-verification"
    $excludedNamespaces = @(
        "kube-system",
        "kube-public",
        "kube-node-lease",
        "azure-arc",
        "azure-arc-release",
        "cert-manager",
        $gatekeeperNamespace,
        $workloadOrchestrationNamespace
    )
    $policyParameters = @{
        effect = @{ value = "Audit" }
        providerName = @{ value = $providerName }
        workloadOrchestrationNamespace = @{ value = $workloadOrchestrationNamespace }
        gatekeeperNamespace = @{ value = $gatekeeperNamespace }
        excludedNamespaces = @{ value = $excludedNamespaces }
    } | ConvertTo-Json -Depth 10 -Compress

    az policy assignment create `
        --name $policyAssignmentName `
        --scope $policyScope `
        --policy $policyDefinitionId `
        --params $policyParameters
    ```

1. After you confirm that signed images pass verification, change `effect` to `Deny` and rerun the assignment command with the same name and scope.

    ```powershell
    $policyParameters = @{
        effect = @{ value = "Deny" }
        providerName = @{ value = $providerName }
        workloadOrchestrationNamespace = @{ value = $workloadOrchestrationNamespace }
        gatekeeperNamespace = @{ value = $gatekeeperNamespace }
        excludedNamespaces = @{ value = $excludedNamespaces }
    } | ConvertTo-Json -Depth 10 -Compress

    az policy assignment create `
        --name $policyAssignmentName `
        --scope $policyScope `
        --policy $policyDefinitionId `
        --params $policyParameters
    ```

### Validate Layer 2

Wait for the Gatekeeper constraint to show `deny`, and then create new pods. Gatekeeper evaluates pod creation, not pods that are already running.

```powershell
kubectl get constraints --all-namespaces | Select-String ratify

$testNamespace = "artifact-verification-test"
$unsignedImage = "<registry-name>.azurecr.io/<repository>:<unsigned-tag>"

kubectl create namespace $testNamespace
kubectl delete pod signed unsigned `
    --namespace $testNamespace `
    --ignore-not-found

# Expected: Pod created.
kubectl run signed --image=$imageReference --namespace $testNamespace

# Expected: Gatekeeper denies admission.
kubectl run unsigned --image=$unsignedImage --namespace $testNamespace
```

If both pods are denied, inspect the Ratify logs:

```powershell
kubectl logs deployment/ratify `
    --namespace $gatekeeperNamespace `
    --tail 50
```

- A `401 Unauthorized` or descriptor resolution error indicates an ACR identity or role assignment problem.
- `no valid Cosign signatures` or `nil certificate` usually indicates that the image wasn't signed with `--new-bundle-format=false`.

## Disable verification

### Disable Layer 1

```powershell
az k8s-extension update `
    --cluster-name $clusterName `
    --resource-group $rg `
    --cluster-type connectedClusters `
    --name $extensionName `
    --config "signing.enabled=false"
```

### Disable Layer 2

Remove enforcement before you remove Ratify or its extension configuration. Reversing this order can cause Gatekeeper to deny every image because its verification provider is unavailable.

1. Delete the policy assignment.

    ```powershell
    az policy assignment delete `
        --name $policyAssignmentName `
        --scope $policyScope
    ```

1. Wait until both commands return no Ratify resources. The Azure Policy add-on can take about 15 minutes to synchronize.

    ```powershell
    kubectl get constraints --all-namespaces | Select-String ratify
    kubectl get constrainttemplate | Select-String ratify
    ```

1. Disable image verification, and then optionally uninstall Ratify.

    ```powershell
    az k8s-extension update `
        --cluster-name $clusterName `
        --resource-group $rg `
        --cluster-type connectedClusters `
        --name $extensionName `
        --config "signing.imageVerification.enabled=false"

    helm uninstall ratify --namespace $gatekeeperNamespace
    ```
