---
title: Use the Azure Key Vault Secret Store extension to sync secrets to the Kubernetes secret store for offline access in Azure Arc-enabled Kubernetes clusters
description: The Azure Key Vault Secret Store extension for Kubernetes ("SSE") automatically synchronizes secrets from an Azure Key Vault to a Kubernetes cluster for offline access.
ms.date: 09/26/2024
ms.topic: how-to
ms.custom: references_regions, ignite-2024
# Customer intent: "As a Kubernetes administrator, I want to automatically synchronize secrets from Azure Key Vault to my Kubernetes cluster for offline access, so that I can manage critical business assets securely, even in semi-disconnected environments."
---

# Use the Secret Store extension to fetch secrets for offline access in Azure Arc-enabled Kubernetes clusters

The Azure Key Vault Secret Store extension for Kubernetes ("SSE") automatically synchronizes secrets from an [Azure Key Vault](/azure/key-vault/general/overview) to an [Azure Arc-enabled Kubernetes cluster](overview.md) for offline access. This means you can use Azure Key Vault to store, maintain, and rotate your secrets, even when running your Kubernetes cluster in a semi-disconnected state. Synchronized secrets are stored in the cluster [secret store](https://Kubernetes.io/docs/concepts/configuration/secret/), making them available as Kubernetes secrets to be used in all the usual ways: mounted as data volumes, or exposed as environment variables to a container in a pod.

Synchronized secrets are critical business assets, so the SSE secures them through isolated namespaces, role-based access control (RBAC) policies, and limited permissions for the synchronization controller. For extra protection, [encrypt](https://Kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) the Kubernetes secret store on your cluster.

This article shows you how to install and configure the SSE as an [Azure Arc-enabled Kubernetes extension](conceptual-extensions.md).

> [!TIP]
> The SSE is recommended for clusters outside of Azure cloud where connectivity to Azure Key Vault may not be perfect. SSE by its nature creates copies of your secrets in the Kubernetes secrets store. If you  prefer to avoid creating local copies of secrets and your cluster has perfect connectivity to Azure Key Vault, then you can use the online-only [Azure Key Vault Secrets Provider extension](tutorial-akv-secrets-provider.md) for secret access in your Arc-enabled Kubernetes clusters. It is not recommended to run both the online Azure Key Vault Secrets Provider extension and the offline SSE side-by-side in the same cluster.

## Prerequisites

- An Arc-enabled cluster. This can be one that you [connected to yourself](quickstart-connect-cluster.md) (this guide assumes a [K3s](https://k3s.io/) cluster, and provides guidance on how to Arc-enable it.) or a Microsoft-managed [AKS enabled by Azure Arc](/azure/aks/hybrid/aks-overview) cluster. The cluster must be running Kubernetes version 1.27 or higher.
- Ensure you meet the [general prerequisites for cluster extensions](extensions.md#prerequisites), including the latest version of the `k8s-extension` Azure CLI extension.
- cert-manager is required to support TLS for intracluster log communication. The examples later in this guide direct you though installation. For more information about cert-manager, see [cert-manager.io](https://cert-manager.io/)

Install the [Azure CLI](/cli/azure/install-azure-cli-linux?pivots=apt) and sign in, if you haven't already:

```azurecli
az login
```

Before you begin, set environment variables to be used for configuring Azure and cluster resources. If you already have a managed identity, Azure Key Vault, or other resource listed here, update the names in the environment variables to reflect those resources. Note that the KEYVAULT_NAME must be globally unique; keyvault creation will fail later if this name is already in use within Azure.

```azurecli
export RESOURCE_GROUP="AzureArcTest"
export CLUSTER_NAME="AzureArcTest1"
export LOCATION="EastUS"
export SUBSCRIPTION="$(az account show --query id --output tsv)"
export AZURE_TENANT_ID="$(az account show -s $SUBSCRIPTION --query tenantId --output tsv)"
export CURRENT_USER="$(az ad signed-in-user show --query userPrincipalName --output tsv)"
export KEYVAULT_NAME="my-UNIQUE-kv-name"
export KEYVAULT_SECRET_NAME="my-secret"
export USER_ASSIGNED_IDENTITY_NAME="my-identity"
export FEDERATED_IDENTITY_CREDENTIAL_NAME="my-credential"
export KUBERNETES_NAMESPACE="my-namespace"
export SERVICE_ACCOUNT_NAME="my-service-account"
```

Create the resource group if necessary:

```azurecli
az group create --name ${RESOURCE_GROUP}  --location ${LOCATION}
```


## Activate workload identity federation in your cluster

The SSE uses a feature called [workload identity federation](conceptual-workload-identity.md) to access and synchronize Azure Key Vault secrets. This section describes how to set the feature up. The following sections will explain how it's used in detail.

### [Arc-enabled Kubernetes](#tab/arc-k8s)

> [!TIP]
> The following steps are based on the [How-to guide](/azure/azure-arc/kubernetes/workload-identity) for configuring Arc-enabled Kubernetes with workload identity federation. Refer to that documentation for any additional assistance.

If your cluster isn't yet connected to Azure Arc, [follow these steps](quickstart-connect-cluster.md). During these steps, enable workload identity federation as part of the `connect` command:

```azurecli
az connectedk8s connect --name ${CLUSTER_NAME} --resource-group ${RESOURCE_GROUP} --enable-oidc-issuer
```

If your cluster is already connected to Azure Arc, enable workload identity using the `update` command:

```azurecli
az connectedk8s update --name ${CLUSTER_NAME} --resource-group ${RESOURCE_GROUP} --enable-oidc-issuer
```

Now configure your cluster to issue Service Account tokens with a new issuer URL (`service-account-issuer`) that enables Microsoft Entra ID to find the public keys necessary for it to validate these tokens. These public keys are for the cluster's own service account token issuer, and they were obtained and cloud-hosted at this URL as a result of the `--enable-oidc-issuer` option that you set earlier.

1. Configure your [kube-apiserver](https://Kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) with the issuer URL field and permissions enforcement. The following example is for a k3s cluster. Your cluster may have different means for changing API server arguments: `--kube-apiserver-arg="--service-account-issuer=${SERVICE_ACCOUNT_ISSUER}" and --kube-apiserver-arg="--enable-admission-plugins=OwnerReferencesPermissionEnforcement"`.

   - Get the service account issuer URL.

      ```console
      export SERVICE_ACCOUNT_ISSUER="$(az connectedk8s show --name ${CLUSTER_NAME} --resource-group ${RESOURCE_GROUP} --query "oidcIssuerProfile.issuerUrl" --output tsv)"
      echo $SERVICE_ACCOUNT_ISSUER
      ```

   - Open `/etc/rancher/k3s/config.yaml` in a text editor. By default there is no configuration for K3s and a blank file is opened.

     ```console
     sudo nano /etc/systemd/system/k3s.service
     ```

    - Add the following configuration settings, then save and close `nano`.
       ```yaml
        kube-apiserver-arg:
          - 'service-account-issuer=${SERVICE_ACCOUNT_ISSUER}'
          - 'service-account-max-token-expiration=24h'
       ```
      Note: You must replace `${SERVICE_ACCOUNT_ISSUER}` with the output from `echo $SERVICE_ACCOUNT_ISSUER` above. `nano` does not substitute variables automatically.

1. Restart your kube-apiserver.

    ```console
   sudo systemctl restart k3s
   ```

1. (optional) Verify the service account issuer has been configured correctly:

```console
kubectl cluster-info dump | grep service-account-issuer
```

### [AKS on Azure Local](#tab/aks-local)

Use the [How-to guide](/azure/aks/hybrid/workload-identity) to activate workload identity federation on AKS on Azure Local by using the `--enable-oidc-issuer` flag.

Return to these steps after the initial activation. There's no need to complete the remainder of that guide.

Validate the activation has been successful by obtaining the cluster's service account issuer URL. You'll use this URL in the following steps:  

   ```console
   export SERVICE_ACCOUNT_ISSUER="$(az connectedk8s show --name ${CLUSTER_NAME} --resource-group ${RESOURCE_GROUP} --query "oidcIssuerProfile.issuerUrl" --output tsv)"
   echo $SERVICE_ACCOUNT_ISSUER
   ```

### [AKS Edge Essentials](#tab/aks-ee)

Use the [How-to guide](/azure/aks/hybrid/aks-edge-workload-identity) to activate workload identity federation on AKS Edge Essentials. 

Return to these steps after the initial activation. There's no need to complete the remainder of that guide.

Validate the activation has been successful by obtaining the cluster's service account issuer URL. You'll use this URL in the following steps:  

   ```console
   export SERVICE_ACCOUNT_ISSUER="$(az connectedk8s show --name ${CLUSTER_NAME} --resource-group ${RESOURCE_GROUP} --query "oidcIssuerProfile.issuerUrl" --output tsv)"
   echo $SERVICE_ACCOUNT_ISSUER
   ```

---

## Create a secret and configure an identity to access it

To access and synchronize a given Azure Key Vault secret, the SSE requires access to an Azure managed identity with appropriate Azure permissions to access that secret. The managed identity must be linked to a Kubernetes service account using the workload identity feature that you activated earlier. The SSE uses the associated federated Azure managed identity to pull secrets from Azure Key Vault to your Kubernetes secret store. The following sections describe how to set this up.

### Create an Azure Key Vault

[Create an Azure Key Vault](/azure/key-vault/secrets/quick-create-cli) and add a secret. If you already have an Azure Key Vault and secret, you can skip this section.

1. Create an Azure Key Vault:

   ```azurecli
   az keyvault create --resource-group "${RESOURCE_GROUP}" --location "${LOCATION}" --name "${KEYVAULT_NAME}" --enable-rbac-authorization
   ```

1. Give yourself 'Secrets Officer' permissions on the vault, so you can create a secret:

   ```azurecli
   az role assignment create --role "Key Vault Secrets Officer" --assignee ${CURRENT_USER} --scope /subscriptions/${SUBSCRIPTION}/resourcegroups/${RESOURCE_GROUP}/providers/Microsoft.KeyVault/vaults/${KEYVAULT_NAME}
   ```

1. Create a secret and update it so you have two versions:

   ```azurecli
   az keyvault secret set --vault-name "${KEYVAULT_NAME}" --name "${KEYVAULT_SECRET_NAME}" --value 'Hello!'
   az keyvault secret set --vault-name "${KEYVAULT_NAME}" --name "${KEYVAULT_SECRET_NAME}" --value 'Hello2'
   ```

### Create a user-assigned managed identity

Next, create a user-assigned managed identity and give it permissions to access the Azure Key Vault. If you already have a managed identity with Key Vault Reader and Key Vault Secrets User permissions to the Azure Key Vault, you can skip this section. For more information, see [Create a user-assigned managed identities](/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities?pivots=identity-mi-methods-azp#create-a-user-assigned-managed-identity) and [Using Azure RBAC secret, key, and certificate permissions with Key Vault](/azure/key-vault/general/rbac-guide?tabs=azure-cli#using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault).

1. Create the user-assigned managed identity:

   ```azurecli
   az identity create --name "${USER_ASSIGNED_IDENTITY_NAME}" --resource-group "${RESOURCE_GROUP}" --location "${LOCATION}" --subscription "${SUBSCRIPTION}"
   ```

1. Give the identity Key Vault Reader and Key Vault Secrets User permissions. You may need to wait a moment for replication of the identity creation before these commands succeed:

   ```azurecli
   export USER_ASSIGNED_CLIENT_ID="$(az identity show --resource-group "${RESOURCE_GROUP}" --name "${USER_ASSIGNED_IDENTITY_NAME}" --query 'clientId' -otsv)"
   az role assignment create --role "Key Vault Reader" --assignee "${USER_ASSIGNED_CLIENT_ID}" --scope /subscriptions/${SUBSCRIPTION}/resourcegroups/${RESOURCE_GROUP}/providers/Microsoft.KeyVault/vaults/${KEYVAULT_NAME}
   az role assignment create --role "Key Vault Secrets User" --assignee "${USER_ASSIGNED_CLIENT_ID}" --scope /subscriptions/${SUBSCRIPTION}/resourcegroups/${RESOURCE_GROUP}/providers/Microsoft.KeyVault/vaults/${KEYVAULT_NAME}
   ```

### Create a federated identity credential

Create a Kubernetes service account for the workload that needs access to secrets. Then, create a [federated identity credential](https://azure.github.io/azure-workload-identity/docs/topics/federated-identity-credential.html) to link between the managed identity, the OIDC service account issuer, and the Kubernetes Service Account.

1. Create a Kubernetes Service Account that will be federated to the managed identity. Annotate it with details of the associated user-assigned managed identity.

   ``` console
   kubectl create ns ${KUBERNETES_NAMESPACE}
   ```

   ``` console
   cat <<EOF | kubectl apply -f -
     apiVersion: v1
     kind: ServiceAccount
     metadata:
       name: ${SERVICE_ACCOUNT_NAME}
       namespace: ${KUBERNETES_NAMESPACE}
   EOF
   ```

1. Create a federated identity credential:

   ```azurecli
   az identity federated-credential create --name ${FEDERATED_IDENTITY_CREDENTIAL_NAME} --identity-name ${USER_ASSIGNED_IDENTITY_NAME} --resource-group ${RESOURCE_GROUP} --issuer ${SERVICE_ACCOUNT_ISSUER} --subject system:serviceaccount:${KUBERNETES_NAMESPACE}:${SERVICE_ACCOUNT_NAME} --audience api://AzureADTokenExchange
   ```

## Install the SSE

The SSE is available as an Azure Arc extension. An [Azure Arc-enabled Kubernetes cluster](overview.md) can be extended with [Azure Arc-enabled Kubernetes extensions](extensions.md). Extensions enable Azure capabilities on your connected cluster and provide an Azure Resource Manager-driven experience for the extension installation and lifecycle management.

[cert-manager](https://cert-manager.io/) and [trust-manager](https://cert-manager.io/docs/trust/trust-manager/) are also required for secure communication of logs between cluster services and must be installed before the Arc extension.

1. Install cert-manager.
   ```azurecli
   helm repo add jetstack https://charts.jetstack.io/ --force-update
   helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set crds.enabled=true 
   ```

1. Install trust-manager.

   ```azurecli
   helm upgrade trust-manager jetstack/trust-manager --install --namespace cert-manager --wait
   ```

1. Install the SSE to your Arc-enabled cluster using the following command:

   ``` console
   az k8s-extension create \
     --cluster-name ${CLUSTER_NAME} \
     --cluster-type connectedClusters \
     --extension-type microsoft.azure.secretstore \
     --resource-group ${RESOURCE_GROUP} \
     --release-train preview \
     --name ssarcextension \
     --scope cluster \
     --auto-upgrade-minor-version false
   ```

   If desired, you can optionally modify the default rotation poll interval by adding `--configuration-settings rotationPollIntervalInSeconds=<time_in_seconds>` (see [configuration reference](secret-store-extension-reference.md#arc-extension-configuration-settings)).

## Configure the SSE

Configure the installed extension with information about your Azure Key Vault and which secrets to synchronize to your cluster by defining instances of Kubernetes [custom resources](https://Kubernetes.io/docs/concepts/extend-Kubernetes/api-extension/custom-resources/). You create two types of custom resources:

- A `SecretProviderClass` object to define the connection to the Key Vault.
- A `SecretSync` object for each secret to be synchronized.

### Create a `SecretProviderClass` resource

The `SecretProviderClass` resource is used to define the connection to the Azure Key Vault, the identity to use to access the vault, which secrets to synchronize, and the number of versions of each secret to retain locally.

You need a separate `SecretProviderClass` for each Azure Key Vault you intend to synchronize, for each identity used for access to an Azure Key Vault, and for each target Kubernetes namespace.

Create one or more `SecretProviderClass` YAML files with the appropriate values for your Key Vault and secrets by following this example.

``` yaml
cat <<EOF > spc.yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: secret-provider-class-name                       # Name of the class; must be unique per Kubernetes namespace
  namespace: ${KUBERNETES_NAMESPACE}                     # Kubernetes namespace to make the secrets accessible in
spec:
  provider: azure
  parameters:
    clientID: "${USER_ASSIGNED_CLIENT_ID}"               # Managed Identity Client ID for accessing the Azure Key Vault with.
    keyvaultName: ${KEYVAULT_NAME}                       # The name of the Azure Key Vault to synchronize secrets from.
    objects: |
      array:
        - |
          objectName: ${KEYVAULT_SECRET_NAME}            # The name of the secret to synchronize.
          objectType: secret
          objectVersionHistory: 2                        # [optional] The number of versions to synchronize, starting from latest.
    tenantID: "${AZURE_TENANT_ID}"                       # The tenant ID of the Key Vault 
EOF
```

See [SecretProviderClass reference](secret-store-extension-reference.md#secretproviderclass-resources) for additional configuration guidance.

### Create a `SecretSync` object

A `SecretSync` object is needed to define how items fetched by the `SecretsProviderClass` are stored in Kubernetes. Kubernetes secrets are key-value maps, just like `ConfigMaps`, and the `SecretSync` object tells SSE how to map items defined in the linked `SecretsProviderClass` into keys in the Kubernetes secret. SSE will create a Kubernetes secret with the same name as the `SecretSync` that describes it.

Create one `SecretSync` object YAML file for each kubernetes secret, following this template. The Kubernetes namespace should match the namespace of the `SecretProviderClass`.

```yaml
cat <<EOF > ss.yaml
apiVersion: secret-sync.x-k8s.io/v1alpha1
kind: SecretSync
metadata:
  name: secret-sync-name                                   # Name of the object; must be unique per Kubernetes namespace
  namespace: ${KUBERNETES_NAMESPACE}                       # Kubernetes namespace
spec:
  serviceAccountName: ${SERVICE_ACCOUNT_NAME}              # The Kubernetes service account to be given permissions to access the secret.
  secretProviderClassName: secret-provider-class-name      # The name of the matching SecretProviderClass with the configuration to access the AKV storing this secret
  secretObject:
    type: Opaque
    data:
    - sourcePath: ${KEYVAULT_SECRET_NAME}/0                # Name of the secret in Azure Key Vault with an optional version number (defaults to latest)
      targetKey: ${KEYVAULT_SECRET_NAME}-data-key0         # Target name of the secret in the Kubernetes secret store (must be unique)
    - sourcePath: ${KEYVAULT_SECRET_NAME}/1                # [optional] Next version of the AKV secret. Note that versions of the secret must match the configured objectVersionHistory in the secrets provider class 
      targetKey: ${KEYVAULT_SECRET_NAME}-data-key1         # [optional] Next target name of the secret in the K8s secret store
EOF
```

> [!TIP]
> Do not include "/0" when referencing a secret from the `SecretProviderClass` where `objectVersionHistory` < 2. The latest version is used implicitly.

See [SecretSync reference](secret-store-extension-reference.md#secretsync-resources) for additional configuration guidance.

### Apply the configuration CRs

Apply the configuration custom resources (CRs) using the `kubectl apply` command:

``` bash
kubectl apply -f ./spc.yaml
kubectl apply -f ./ss.yaml
```

The SSE automatically looks for the secrets and begins syncing them to the cluster.

## Observe secrets synchronizing to the cluster

Once the configuration is applied, secrets begin syncing to the cluster automatically at the cadence specified when installing the SSE.

### View synchronized secrets

View the secrets synchronized to the cluster by running the following command:

```bash
# View a list of all secrets in the namespace
kubectl get secrets -n ${KUBERNETES_NAMESPACE}
```

> [!TIP]
> Add `-o yaml` or `-o json` to change the output format of `kubectl get` and `kubectl describe` commands.

### View secrets values

To view the synchronized secret values, now stored in the Kubernetes secret store, use the following command:

```bash
kubectl get secret secret-sync-name -n ${KUBERNETES_NAMESPACE} -o jsonpath="{.data.${KEYVAULT_SECRET_NAME}-data-key0}" | base64 -d && echo
kubectl get secret secret-sync-name -n ${KUBERNETES_NAMESPACE} -o jsonpath="{.data.${KEYVAULT_SECRET_NAME}-data-key1}" | base64 -d && echo
```

## Troubleshooting

See the [troubleshooting guide](secret-store-extension-troubleshooting.md) for assistance diagnosing and resolving issues.

## Remove the SSE

To remove the SSE and stop synchronizing secrets, uninstall it with the `az k8s-extension delete` command:

```console
az k8s-extension delete --name ssarcextension --cluster-name $CLUSTER_NAME  --resource-group $RESOURCE_GROUP  --cluster-type connectedClusters    
```

Uninstalling the extension doesn't remove secrets, `SecretSync` objects, or CRDs from the cluster. These objects must be removed directly with `kubectl`.

Deleting the SecretSync CRD removes all `SecretSync` objects, and by default removes all owned secrets, but secrets may persist if:

- You modified ownership of any of the secrets.
- You changed the [garbage collection](https://kubernetes.io/docs/concepts/architecture/garbage-collection/) settings in your cluster, including setting different [finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/).

In these cases, secrets must be deleted directly using `kubectl`.

## Next steps

- Learn more about [Azure Arc extensions](extensions.md).
- Learn more about [Azure Key Vault](/azure/key-vault/general/overview).
- Help to protect your cluster in other ways by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).
