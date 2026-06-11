---
title: Migrate from the Azure Key Vault Secrets Provider extension to the Secret Store extension
description: Plan and perform a migration from the Azure Key Vault Secrets Provider extension to the Secret Store extension on an Azure Arc-enabled Kubernetes cluster.
ms.date: 06/09/2026
ms.topic: how-to
ms.custom: devx-track-azurecli
# Customer intent: As a cluster administrator running the Azure Key Vault Secrets Provider extension on an Azure Arc-enabled Kubernetes cluster, I want to understand what migrating to the Secret Store extension involves and how to do it safely, so that I can decide whether to migrate and execute the migration with confidence.
---

# Migrate from the Azure Key Vault Secrets Provider extension to the Secret Store extension

This guide is for cluster administrators who already use the [Azure Key Vault Secrets Provider extension](tutorial-akv-secrets-provider.md) (AKV SPE), and are considering moving to the [Secret Store extension (SSE)](secret-store-extension.md).

The two extensions make secrets from Azure Key Vault available to Kubernetes workloads, but the SSE improves on the Azure Key Vault Secrets Provider extension in three ways:

- **Offline resilience.** SSE keeps a local copy of each secret in the Kubernetes secret store, so a temporary loss of connectivity to Azure Key Vault doesn't interrupt workloads.
- **Stronger authentication.** SSE uses workload identity federation, so the cluster authenticates to Azure with short-lived service account tokens and no long-lived credentials are stored on the cluster.
- **Support for large fleets.** SSE's `jitterSeconds` setting adds a randomized delay to each sync, so coordinated bursts of requests don't overwhelm Azure Key Vault when many clusters poll on the same schedule.

Migration from the Azure Key Vault Secrets Provider extension to the SSE requires changes to how the cluster authenticates to Azure, changes to how applications consume their secrets, and on some Kubernetes distributions it requires cluster-level configuration changes.

> [!WARNING]
> Perform a trial migration on a non-production cluster before rolling out to production. Some steps in this guide are difficult to reverse.

## Prerequisites and considerations

### Prerequisites

- **Activate workload identity federation on the cluster.** SSE authenticates to Azure with short-lived service-account tokens issued by the cluster and validated by Microsoft Entra ID. See [Deploy and configure workload identity federation in Azure Arc-enabled Kubernetes](workload-identity.md) for third-party Arc clusters, or the dedicated guides for [AKS enabled by Azure Arc](/azure/aks/hybrid/workload-identity) or [AKS Edge Essentials](/azure/aks/hybrid/aks-edge-workload-identity). This step requires Kubernetes 1.27 or later and the ability to set `service-account-issuer` on the kube-apiserver.
- **Install the [cert-manager for Arc-enabled Kubernetes (preview)](cert-manager-overview.md) extension.** SSE uses it for intracluster TLS. Check its own [supported regions](cert-manager-overview.md#regional-support) and [validated distributions](cert-manager-overview.md#validated-arc-enabled-kubernetes-distributions). If the cluster already runs open source cert-manager or trust-manager, [uninstall them first](cert-manager-deploy.md#migrate-from-open-source-cert-manager-and-trust-manager).
- **Plan one service account and one federated credential per consuming namespace.** SSE is namespace-scoped. A single managed identity supports a [maximum of 20 federated credentials](/entra/workload-id/workload-identity-federation-considerations#general-federated-identity-credential-considerations); use additional managed identities for clusters with more consuming namespaces.
- **Plan a switch-over window.** Migration uninstalls the Azure Key Vault Secrets Provider extension before installing SSE. The two aren't supported side by side in the same cluster, so secret-consuming workloads are disrupted during the switchover.

### Considerations

- **Windows containers aren't supported.** SSE supports Linux only.
- **Sovereign clouds aren't supported.** SSE doesn't accept a `cloudName` parameter. Use the Azure Key Vault Secrets Provider extension's `cloudName` for vaults in `AzureUSGovernmentCloud` or `AzureChinaCloud`.
- **Secrets are persisted in the Kubernetes secret store.** SSE writes Azure Key Vault contents into native Kubernetes Secret objects. The cluster's RBAC, audit posture, and [secret-store encryption at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) now apply to those values. Confirm they meet your requirements before migrating.

## Migration at a glance

The migration workflow is in six stages:

1. **Inventory.** Capture the existing `SecretProviderClass` resources and the workloads that mount them.
2. **Prepare SSE prerequisites.** Create a managed identity and enable workload identity on the cluster. Install the cert-manager for Arc-enabled Kubernetes extension.
3. **Switch over.** Uninstall the Azure Key Vault Secrets Provider extension and install the SSE.
4. **Translate configuration.** Edit each existing `SecretProviderClass` for SSE and add a `SecretSync` for every Kubernetes Secret you want SSE to produce, or replace both with `AKVSync` (preview).
5. **Update workloads.** Re-point workloads at the SSE-produced Kubernetes Secret. Workloads that already read the AKV SPE–synced Secret may need no change; CSI-mounting workloads switch to a Secret-backed volume or `secretKeyRef`.
6. **Verify and clean up.** Confirm secrets are syncing. Revoke the old service principal and remove the leftover credentials and local inventory files.

## Set up your environment

The following environment variables will be used during migration steps. Complete the variables appropriately for your deployment:

```azurecli
export RESOURCE_GROUP="<your-resource-group>"
export CLUSTER_NAME="<your-arc-connected-cluster-name>"
export LOCATION="<your-azure-region>"
export SUBSCRIPTION="$(az account show --query id --output tsv)"
export AZURE_TENANT_ID="$(az account show -s $SUBSCRIPTION --query tenantId --output tsv)"
export KEYVAULT_NAME="<your-key-vault-name>"
export USER_ASSIGNED_IDENTITY_NAME="<name-for-the-new-managed-identity>"
export FEDERATED_IDENTITY_CREDENTIAL_NAME="<name-for-the-federated-credential>"
export KUBERNETES_NAMESPACE="<namespace-where-secrets-will-be-synced>"
export SERVICE_ACCOUNT_NAME="<kubernetes-service-account-name>"
```

Use the latest [`k8s-extension`](/cli/azure/k8s-extension) Azure CLI extension. The YAML examples in Stages 4 and 5 reference these variables; pipe them through `envsubst` (as shown in the `kubectl apply` commands) to expand them at apply time.

## Stage 1: Inventory the existing setup

Discover and record where and how the Azure Key Vault Secrets Provider extension is used. This guide assumes AKV SPE was configured with service-principal authentication (each consuming pod has a `nodePublishSecretRef` pointing to a Kubernetes Secret with `clientid` and `clientsecret`). If you use managed-identity or workload-identity authentication instead, skip the credential-handling steps but otherwise follow the same flow.

### Capture every SecretProviderClass

The `SecretProviderClass` CRD (`secrets-store.csi.x-k8s.io/v1`) and your existing `SecretProviderClass` instances remain on the cluster after the Stage 3 switchover. SSE consumes them after the edits in Stage 4.

```bash
kubectl get secretproviderclass --all-namespaces -o yaml > akv-spe-spc-backup.yaml
```

### Identify workloads that mount via CSI

Capture both the running pods and the controllers that own them. Stage 5 updates each controller's pod template; the pod-level capture records where CSI volumes are mounted today, including the `nodePublishSecretRef` to clean up in Stage 6.

```bash
kubectl get pods --all-namespaces -o json | \
  jq '[.items[] | select(.spec.volumes[]?.csi?.driver=="secrets-store.csi.k8s.io")]' \
  > akv-spe-csi-consumers.json

kubectl get deploy,sts,ds,job,cronjob --all-namespaces -o json | \
  jq '[.items[] | select(.spec.template.spec.volumes[]?.csi?.driver=="secrets-store.csi.k8s.io")]' \
  > akv-spe-csi-owners.json
```

### Record the extension configuration

Capture any non-default settings applied to the Azure Key Vault Secrets Provider extension, for example rotation interval, `syncSecret.enabled`, or `linux.kubeletRootDir`:

```azurecli
az k8s-extension show \
  --cluster-type connectedClusters \
  --cluster-name $CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --name <akv-spe-extension-name> \
  --query configurationSettings \
  > akv-spe-extension-config.json
```

### Note the service principal and its Azure Key Vault access

Each consuming pod's `nodePublishSecretRef` names a Kubernetes Secret holding the service principal credentials. Extract the client ID from each unique Secret:

```bash
kubectl get secret <name> -n <namespace> -o jsonpath='{.data.clientid}' | base64 -d
```

For each client ID, locate the Azure Key Vault access policy or role assignment that grants it `Get` on secrets, keys, and certificates (use `az role assignment list --assignee <client-id> --scope <vault-resource-id>` for RBAC-enabled vaults, or `az keyvault show --name <vault-name> --query properties.accessPolicies` for the legacy permission model). You'll revoke this access in Stage 6, after SSE is fully operational.

## Stage 2: Prepare SSE prerequisites

### Create a user-assigned managed identity

```azurecli
az identity create \
  --name "${USER_ASSIGNED_IDENTITY_NAME}" \
  --resource-group "${RESOURCE_GROUP}" \
  --location "${LOCATION}" \
  --subscription "${SUBSCRIPTION}"

export USER_ASSIGNED_CLIENT_ID="$(az identity show \
  --resource-group "${RESOURCE_GROUP}" \
  --name "${USER_ASSIGNED_IDENTITY_NAME}" \
  --query 'clientId' --output tsv)"
```

Grant the managed identity access to the Azure Key Vault that holds your secrets. These commands assume the vault uses Azure RBAC; if it uses access policies, grant `Get` on secrets, certificates, and keys instead.

```azurecli
az role assignment create \
  --role "Key Vault Reader" \
  --assignee "${USER_ASSIGNED_CLIENT_ID}" \
  --scope "/subscriptions/${SUBSCRIPTION}/resourcegroups/${RESOURCE_GROUP}/providers/Microsoft.KeyVault/vaults/${KEYVAULT_NAME}"

az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee "${USER_ASSIGNED_CLIENT_ID}" \
  --scope "/subscriptions/${SUBSCRIPTION}/resourcegroups/${RESOURCE_GROUP}/providers/Microsoft.KeyVault/vaults/${KEYVAULT_NAME}"
```

### Enable workload identity on the cluster

SSE needs the OIDC issuer to be enabled so Microsoft Entra ID can validate the cluster's service account tokens.

```azurecli
az connectedk8s update \
  --name ${CLUSTER_NAME} \
  --resource-group ${RESOURCE_GROUP} \
  --enable-oidc-issuer
```

Retrieve the issuer URL and apply it to the kube-apiserver. The example below is for K3s; consult your distribution's documentation for the equivalent step on other clusters.

```bash
export SERVICE_ACCOUNT_ISSUER="$(az connectedk8s show \
  --name ${CLUSTER_NAME} \
  --resource-group ${RESOURCE_GROUP} \
  --query "oidcIssuerProfile.issuerUrl" --output tsv)"
echo $SERVICE_ACCOUNT_ISSUER
```

> [!CAUTION]
> Don't replace the existing kube-apiserver configuration. Merge these arguments into whatever already exists on your cluster.

For K3s, merge the following into `/etc/rancher/k3s/config.yaml` (substituting the issuer URL printed above, and replacing any existing `service-account-issuer` or `service-account-max-token-expiration` entries under `kube-apiserver-arg`) and restart the service:

```yaml
kube-apiserver-arg:
  - 'service-account-issuer=<SERVICE_ACCOUNT_ISSUER>'
  - 'service-account-max-token-expiration=24h'
```

```bash
sudo systemctl restart k3s
```

For more detail and other distributions, see [Deploy and configure workload identity federation in Azure Arc-enabled Kubernetes](workload-identity.md).

### Create the Kubernetes service account and federated credential

> [!NOTE]
> Repeat this step for each consuming namespace, using distinct `KUBERNETES_NAMESPACE`, `SERVICE_ACCOUNT_NAME`, and `FEDERATED_IDENTITY_CREDENTIAL_NAME` values per namespace.

```bash
kubectl create namespace ${KUBERNETES_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ${SERVICE_ACCOUNT_NAME}
  namespace: ${KUBERNETES_NAMESPACE}
EOF
```

```azurecli
az identity federated-credential create \
  --name ${FEDERATED_IDENTITY_CREDENTIAL_NAME} \
  --identity-name ${USER_ASSIGNED_IDENTITY_NAME} \
  --resource-group ${RESOURCE_GROUP} \
  --issuer ${SERVICE_ACCOUNT_ISSUER} \
  --subject system:serviceaccount:${KUBERNETES_NAMESPACE}:${SERVICE_ACCOUNT_NAME} \
  --audience api://AzureADTokenExchange
```

### Install the cert-manager for Arc-enabled Kubernetes extension

SSE uses [cert-manager](cert-manager-overview.md) and trust-manager for intracluster TLS between its components.

If the cluster already has open source cert-manager or trust-manager, [uninstall them first](cert-manager-deploy.md#migrate-from-open-source-cert-manager-and-trust-manager). The configuration will be picked up by the Arc extension when installed.

```azurecli
az k8s-extension create \
  --resource-group ${RESOURCE_GROUP} \
  --cluster-name ${CLUSTER_NAME} \
  --cluster-type connectedClusters \
  --name "azure-cert-management" \
  --extension-type "microsoft.certmanagement"
```

After installation, [confirm the cert-manager components are running](cert-manager-monitor-troubleshoot.md#confirm-that-pods-and-components-are-running) before continuing. For more detail, see [Deploy cert-manager for Arc-enabled Kubernetes](cert-manager-deploy.md).

## Stage 3: Switch to the Secret Store extension

> [!WARNING]
> Once the Azure Key Vault Secrets Provider extension is uninstalled, its CSI driver components are removed from the cluster. From this point, pods that start, restart, or reschedule won't be able to mount their `secrets-store.csi.k8s.io` volumes and will fail to start until you complete Stage 4 and Stage 5. Already-running pods may briefly continue to access their mounted secrets, but pod behavior after uninstall is unpredictable; treat any continued access as best-effort and plan as if all CSI-backed secret access is unavailable from this point.

### Uninstall the Azure Key Vault Secrets Provider extension

Find the installed name of the extension:

```azurecli
az k8s-extension list \
  --cluster-name $CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --cluster-type connectedClusters \
  --query "[?extensionType=='microsoft.azurekeyvaultsecretsprovider'].name" \
  -o tsv
```

Delete it (replace `akvsecretsprovider` with the name returned above):

```azurecli
az k8s-extension delete \
  --cluster-type connectedClusters \
  --cluster-name $CLUSTER_NAME \
  --resource-group $RESOURCE_GROUP \
  --name akvsecretsprovider
```

The `nodePublishSecretRef` Secrets that held the service-principal credentials are now inert (the CSI driver that consumed them is gone). Leave them in place until Stage 6 so you have a fast rollback path if Stage 4 or Stage 5 fails.

### Install the Secret Store extension

```azurecli
az k8s-extension create \
  --cluster-name ${CLUSTER_NAME} \
  --cluster-type connectedClusters \
  --extension-type microsoft.azure.secretstore \
  --resource-group ${RESOURCE_GROUP} \
  --name ssarcextension \
  --scope cluster
```

See the [extension configuration reference](secret-store-extension-reference.md#arc-extension-configuration-settings) for possible extension configuration settings.

## Stage 4: Update configuration for the Secret Store Extension

Each Azure Key Vault Secrets Provider `SecretProviderClass` you inventoried in Stage 1 becomes one SSE `SecretProviderClass` plus one `SecretSync` per Kubernetes Secret you want SSE to produce. If you prefer a single resource per Azure Key Vault, the simplified `AKVSync` resource (preview) replaces both.

### [Direct (recommended for migration)](#tab/direct)

#### Update each SecretProviderClass

The `SecretProviderClass` resources from the Azure Key Vault Secrets Provider remain in the cluster after Stage 3. Edit each one (or you can extract and modify them from `akv-spe-spc-backup.yaml` taken in Stage 1):

| Change | Why |
| --- | --- |
| Add `spec.parameters.clientID` and set it to the user-assigned managed identity's client ID. | This is how SSE knows which managed identity to federate to the service account. |
| Remove `spec.parameters.usePodIdentity`, `useVMManagedIdentity`, and `userAssignedIdentityID` if present. | SSE doesn't read these legacy auth flags; `clientID` plus the federated credential is the only auth path. |
| Optionally add `objectVersionHistory: <n>` to entries in `objects` to sync multiple versions of each secret from Azure Key Vault. | The Azure provider supports this in both AKV SPE and SSE, but only SSE exposes each version as a distinct Kubernetes Secret data key (see SecretSync / AKVSync `v0`, `v1`, ...). |

> [!NOTE]
> If any `spec.secretObjects` remain in the `SecretProviderClass`, SSE ignores them. This feature stored secrets into the Kubernetes secret store, which SSE's basic functionality provides.

**Before** (existing `SecretProviderClass` from the Azure Key Vault Secrets Provider):

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: my-akv-provider
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    keyvaultName: my-key-vault
    objects: |
      array:
        - |
          objectName: my-secret
          objectType: secret
        - |
          objectName: my-certificate
          objectType: cert
    tenantID: "${AZURE_TENANT_ID}"
```

**After** (same `SecretProviderClass`, edited for SSE):

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: my-akv-provider
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  provider: azure
  parameters:
    clientID: "${USER_ASSIGNED_CLIENT_ID}"
    keyvaultName: my-key-vault
    objects: |
      array:
        - |
          objectName: my-secret
          objectType: secret
        - |
          objectName: my-certificate
          objectType: cert
    tenantID: "${AZURE_TENANT_ID}"
```

#### Create a SecretSync

For each Kubernetes Secret you want SSE to produce, create a `SecretSync`. The Kubernetes Secret takes its name from `metadata.name` of the `SecretSync`.

```yaml
apiVersion: secret-sync.x-k8s.io/v1alpha1
kind: SecretSync
metadata:
  name: my-app-secrets
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  serviceAccountName: ${SERVICE_ACCOUNT_NAME}
  secretProviderClassName: my-akv-provider
  secretObject:
    type: Opaque
    data:
      - sourcePath: my-secret
        targetKey: my-secret-value
      - sourcePath: my-certificate
        targetKey: my-certificate-value
```

`sourcePath` matches the `objectName` in the `SecretProviderClass`. `targetKey` is the data key within the resulting Kubernetes Secret. If your `SecretProviderClass` entries set `objectVersionHistory > 1`, reference specific versions with `<objectName>/0`, `<objectName>/1`, and so on (`/0` is the most recent). See the [SecretSync reference](secret-store-extension-reference.md#secretsync-resources).

If you need the same secret in workloads in different namespaces, create a copy of the `SecretSync` resource for each namespace, alongside a Kubernetes service account and federated identity credential for that namespace.

#### Apply the configuration

```bash
envsubst < spc.yaml | kubectl apply -f -
envsubst < secretsync.yaml | kubectl apply -f -
```

### [Simplified using AKVSync (preview)](#tab/simplified)

`AKVSync` collapses the `SecretProviderClass` and `SecretSync` into one resource. SSE generates the required `SecretProviderClass` and `SecretSync` instances automatically; don't edit the generated resources.

```yaml
apiVersion: secret-sync.x-k8s.io/v1alpha1
kind: AKVSync
metadata:
  name: my-app-secrets
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  keyvaultName: ${KEYVAULT_NAME}
  clientID: "${USER_ASSIGNED_CLIENT_ID}"
  tenantID: "${AZURE_TENANT_ID}"
  serviceAccountName: ${SERVICE_ACCOUNT_NAME}
  objects:
    - secretInAKV: my-secret
```

Each entry in `objects` produces one Kubernetes Secret named after the AKV object (here, `my-secret`), with the value stored in data key `v0`. Use `kubernetesSecretName` to override the name.

`AKVSync` is a preview feature; the direct (`SecretProviderClass` + `SecretSync`) style is generally available. See the [AKVSync reference](secret-store-extension-reference.md#akvsync-resources-preview) for compound secrets, version history, and additional fields.

```bash
envsubst < akvsync.yaml | kubectl apply -f -
```

---

### Verify synchronization

Check the status:

```bash
# Direct style:
kubectl describe secretsync my-app-secrets -n ${KUBERNETES_NAMESPACE}

# AKVSync style:
kubectl describe akvsync my-app-secrets -n ${KUBERNETES_NAMESPACE}
```

A healthy sync reports a status reason of `UpdateNoValueChangeSucceeded` or `UpdateValueChangeOrForceUpdateSucceeded`. Investigate further if the status is any other value. `ProviderError` indicates that SSE couldn't reach Azure Key Vault; causes include connectivity issues, insufficient permissions on the identity, or `SecretProviderClass` misconfiguration. Cross-reference the [troubleshooting guide](secret-store-extension-troubleshooting.md#secretsync-status-reasons) before changing any configuration.

When the sync reports success, confirm the Kubernetes Secret exists:

```bash
# Direct style:
kubectl get secret my-app-secrets -n ${KUBERNETES_NAMESPACE}
kubectl get secret my-app-secrets -n ${KUBERNETES_NAMESPACE} \
  -o jsonpath="{.data.my-secret-value}" | base64 -d && echo

# AKVSync style:
kubectl get secret my-secret -n ${KUBERNETES_NAMESPACE}
kubectl get secret my-secret -n ${KUBERNETES_NAMESPACE} \
  -o jsonpath="{.data.v0}" | base64 -d && echo
```

## Stage 5: Update workloads

Replace each AKV SPE CSI volume in your pod manifests with one of:

- **A Secret-backed volume mount** if the workload reads secrets from files at a path.
- **`secretKeyRef`** if the workload reads secrets from environment variables.

You may need both for different workloads (or different containers in the same pod).

### Before (a pod using an AKV SPE CSI volume)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  containers:
    - name: my-app
      image: my-app:latest
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets-store"
          readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "my-akv-provider"
        nodePublishSecretRef:
          name: secrets-store-creds
```

### After (files via a Secret-backed volume)

Use this when your application reads secrets from files at a mount path. The application code doesn't need to change; only the volume source does.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  containers:
    - name: my-app
      image: my-app:latest
      volumeMounts:
        - name: secrets-volume
          mountPath: "/mnt/secrets-store"  # Same path as before
          readOnly: true
  volumes:
    - name: secrets-volume
      secret:
        secretName: my-app-secrets         # SecretSync (or AKVSync) name
```

> [!NOTE]
> File names in a Secret-backed volume come from the Kubernetes Secret's data keys. For `SecretSync`, the keys are the `targetKey` values you chose. For `AKVSync`, the keys are version-indexed (`v0`, `v1`, ...) by default and won't match what an existing application reads; use compound `AKVSync` entries with explicit `dataKey` values, or use the `SecretSync` style, to control filenames.

### After (environment variables via `secretKeyRef`)

Use this when your application reads secrets from environment variables, or when you want to make the secret name explicit in the pod spec.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: ${KUBERNETES_NAMESPACE}
spec:
  containers:
    - name: my-app
      image: my-app:latest
      env:
        - name: MY_SECRET
          valueFrom:
            secretKeyRef:
              name: my-app-secrets        # SecretSync (or AKVSync) name
              key: my-secret-value        # targetKey within the synced Secret
```

> [!NOTE]
> The pod doesn't need its own `serviceAccountName` to read a Kubernetes Secret. Workload identity in this migration is used by SSE (via the `SecretSync` or `AKVSync` resource) to fetch from Azure Key Vault, not by the consuming workload. Add `serviceAccountName` to the pod only if it also needs to authenticate to Azure for some other reason.

If AKV SPE was installed with `syncSecret.enabled=true` and the workload was already consuming the synced Kubernetes Secret via `secretKeyRef`, the `env` block stays as-is, provided the SSE-produced Secret has the same name and the same data keys (`targetKey` values) as the AKV SPE Secret. Only the CSI volume needs to go.

### Restart workloads to apply the changes

Apply the manifests, then restart the workloads so they pick up the new Secret references. The exact command depends on the workload kind:

```bash
envsubst < my-app.yaml | kubectl apply -f -

# Deployment, StatefulSet, or DaemonSet:
kubectl rollout restart deployment/my-app -n ${KUBERNETES_NAMESPACE}

# Bare Pod (as shown in the examples above):
kubectl delete pod my-app -n ${KUBERNETES_NAMESPACE} && envsubst < my-app.yaml | kubectl apply -f -
```

## Stage 6: Verify and clean up

Once secrets are syncing and workloads are healthy:

1. **Revoke the old service principal's access.** Remove its access policy or role assignments on the Azure Key Vault, and disable or delete the service principal in Microsoft Entra ID if it isn't used elsewhere.
2. **Delete the leftover `nodePublishSecretRef` Secrets.** Extract the namespace/name pairs from the Stage 1 inventory and delete each one:

   ```bash
   jq -r '.[] | .metadata.namespace as $ns
          | .spec.volumes[]? | select(.csi?.driver=="secrets-store.csi.k8s.io")
          | "\($ns) \(.csi.nodePublishSecretRef.name)"' \
     akv-spe-csi-consumers.json | sort -u

   kubectl delete secret <name> -n <namespace> --ignore-not-found
   ```
3. **Remove the Stage 1 inventory files** (`akv-spe-spc-backup.yaml`, `akv-spe-csi-consumers.json`, `akv-spe-csi-owners.json`, `akv-spe-extension-config.json`) from your workstation.

The `secrets-store.csi.x-k8s.io` CRDs intentionally stay on the cluster as SSE still uses them.

## Troubleshoot Secret Store migration issues

- **`ProviderError` immediately after applying the SecretSync.** Federated credentials can take a few minutes to propagate after creation. Also double check the kube-apiserver was restarted after `service-account-issuer` was set in Stage 2.
- **Authentication fails on every request.** Confirm the managed identity has both `Key Vault Reader` and `Key Vault Secrets User` on the vault (or equivalent access-policy permissions), and that `clientID` in the `SecretProviderClass` is the managed identity's **client ID**, not its principal/object ID.
- **Workload still tries to mount the CSI volume.** Look for stale references to `driver: secrets-store.csi.k8s.io` or `secretProviderClass:` in pod specs left over from Stage 5.

See the [Secret Store extension troubleshooting guide](secret-store-extension-troubleshooting.md).

## Additional links

- [Secret Store extension overview and getting started](secret-store-extension.md)
- [Secret Store extension configuration reference](secret-store-extension-reference.md)
- [Secret Store extension troubleshooting](secret-store-extension-troubleshooting.md)
- [Workload identity federation in Azure Arc-enabled Kubernetes](conceptual-workload-identity.md)
- [Azure Arc-enabled Kubernetes security book](conceptual-security-book.md)
