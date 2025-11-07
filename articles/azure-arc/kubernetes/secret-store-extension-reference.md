---
title: Reference for Azure Key Vault Secret Store Extension
ms.date: 08/01/2025
ms.topic: reference
description: A reference guide for the Azure Key Vault Secret Store Extension, documenting the possibilities allowed in each of SSE's configuration resources.
---

# Azure Key Vault Secret Store extension configuration reference

The SSE can be configured in three places: Configuration setting provided to Arc infrastructure when creating or updating the extension, SecretSync kubernetes resources, and SecretProviderClass kubernetes resources.

## Arc extension configuration settings

Configuration settings can be set when the SSE Arc extension instance is created, or they can be updated later. Use ```--configuration-settings <setting>=<value>``` with ```az k8s-extension create ...``` or ```az k8s-extension update ...``` to create or update an SSE instance respectively.


SSE accepts the following Arc extension configuration parameters:


   | Parameter name                    | Description                         | Default value                         |
   |---------------------------------|-------------------------------------------------------------------------------|----------------------------------------------|
   | `rotationPollIntervalInSeconds`          | Specifies how quickly the SSE checks or updates the secret it's managing.       | `3600` (1 hour)                                             |
   | `enablePartialSecretSync` | When set to `false` a secret is only updated if every contained item could be fetched from Azure Key Vault (AKV) successfully. When `true` each item in a secret is updated if it was fetched successfully, without regard to the success of other items in the secret. | `true` |
   | `jitterSeconds` | Specifies the maximum additional SecretSync jitter. SSE will wait a random time between 0 and `jitterSeconds` each time it considers a SecretSync resource, this will happen every time a SecretSync is updated or after `rotationPollIntervalInSeconds` has elapsed. See  | `0` (no jitter) |


## SecretSync resources

SecretSync resources configure how SSE stores secrets and certificates into the Kubernetes secret store. Each SecretSync resource defines one Kubernetes secret, although it may contain more than one secret.

### Example

```yaml
apiVersion: secret-sync.x-k8s.io/v1alpha1
kind: SecretSync
metadata:
  name: secret-sync-name
  namespace: workload-namespace
spec:
  serviceAccountName: workload-serviceaccountname
  secretProviderClassName: secret-provider-class-name
  secretObject:
    type: Opaque
    data:
    - sourcePath: aSecret/0
      targetKey: aSecret-data-key0
    - sourcePath: aSecret/1
      targetKey: aSecret-data-key1
    labels:
      - fromExample: absolutelyYes
    annotations:
      - exampleAnnotation: annotationValue 
  forceSynchronization: ArbitraryValue12354
```

### Values 

 - **metadata.name** *(required)*: The name of this SecretSync resource. Note: The automatically created Secret resource has the same name as this resource.

Within .spec:

 - **serviceAccountName** *(required)*: The Kubernetes service account used to access the Kubernetes secret store. This service account should be federated with the managed identity with access to the secrets in Azure Key Vault.
 - **secretProviderClassName** *(required)*: The name of the SecretProviderClass resource that defines which secrets to fetch from Azure Key Vault.
 - **secretObject** *(required)*: Defines how the stored secret resource should be structured.
   - **type** *(required)*: The type of the Kubernetes secret object. Set this field to `Opaque` for a general purpose secret with no imposed structure. See [Types of Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types) for guidance on how the special purpose secret types need to be structured.
   -  **data** *(required)*: A list of data items within the secret resource. There must be at least one item. Each data item must contain these two fields:
       - **sourcePath** *(required)*: The path to an item fetched from AKV. When only one version of the named secret is fetched from AKV, the path is simply `<secret name>`.<br> If more than one version of the named secret is fetched from AKV, then the latest version's sourcePath is `<secret name>/0`, second latest is `<secret name>/1`, etc.<br> When a certificate is fetched from AKV, the sourcePaths depend on the value of `objectType` in the SecretProviderClass. When the `objectType` in the SPC is "secret" then both a certificate and private key are available at sourcePaths of `<secret name>/tls.crt` and `<secret name>/tls.key` respectively.
       - **targetKey** *(required)*: The key in the Kubernetes secret object to store the data into.
   - **labels** *(optional)*: A list of key-value pairs of additional labels to apply to the secret object.
   - **annotations** *(optional)*: A list of key-value pairs of additional annotations to apply to the secret object.
   - **forceSynchronization** *(optional)*: Changes to this field trigger SSE to recheck AKV for changes. Kubernetes is updated as usual if SSE finds updated data. The value of this field does not affect how the SSE behaves.

## SecretProviderClass resources

SecretProviderClass resources configure what and how to fetch from an Azure Key Vault. This reference only covers the fields necessary for the SSE use cases of the SecretProviderClass.

### Example

``` yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: secret-provider-class-name
  namespace: workload-namespace
spec:
  provider: azure
  parameters:
    clientID: "xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx"
    tenantID: "xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx"
    keyvaultName: exampleKeyvault
    objects: |
      array:
        - |
          objectName: aSecret
          objectType: secret
          objectVersionHistory: 2
        - |
          objectName: aCertificate
          objectType: secret
```

### Values

Within .spec:

 - **provider** *(required)*: Set this field to `azure` when using SSE to fetch secrets from Azure Key Vault.
 - **parameters** *(required)*: Defines how and where to fetch AKV secrets from.
     - **clientID** *(required)*: The clientID for the managed identity with access to the required secrets. This managed identity must have a federated credential associated with the service account named in the SecretSync resource that uses this SecretProviderClass.
     - **tenantID** *(required)*: The ID of the Azure tenant containing the AKV instance.
     - **keyvaultName** *(required)*: The name of the keyvault.
     - **objects** *(required)*: A **string** containing a YAML fragment representing the items to be fetched from this AKV. Pay close attention to the example to see how the objects field is constructed. Additional examples of SecretProviderClass resources for use with Azure Key Vault may be found in the [AKV provider docs](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/keyvault-secrets/v1alpha1_secretproviderclass_secrets.yaml). **objects** must contain at least one item within the "array" sub-object.
         - **objectName** *(required)*: The name of the secret or certificate to fetch from AKV.
         - **objectType** *(required)*: Set this field to `secret` when fetching a secret from AKV.<br> When fetching a certificate, set this field to:
             - `cert` to fetch only the certificate.
             - `key` to fetch only the public key of the certificate.
             - `secret` to fetch the certificate *and* the private key.
         - **objectVersionHistory** *(optional)*: If present and greater than one, this many versions are fetched from AKV, starting with the latest.
