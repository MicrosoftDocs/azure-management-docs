---
title: Reference for Azure Key Vault Secret Store Extension
description: A reference guide for the Azure Key Vault Secret Store Extension, documenting the possibilities allowed in each of SSE's configuration resources.
ms.date: 08/01/2025
---

# Azure Key Vault Secret Store extension configuration reference

The SSE can be configured in three places: Configuration setting provided to Arc infrastructure when creating or updating the extension, SecretSync kubernetes resources, and SecretProviderClass kubernetes resources.

## Arc extension configuration settings

Configuration settings can be set when the SSE arc extension instance is created, or can be updated later. Use ```--configuration-settings <setting>=<value>``` with ```az k8s-extension create ...``` or ```az k8s-extension update ...``` to create or update an SSE instance respectively.

SSE accepts the following arc extension configuration parameters:

   | Parameter name                    | Description                         | Default value                         |
   |---------------------------------|-------------------------------------------------------------------------------|----------------------------------------------|
   | `rotationPollIntervalInSeconds`          | Specifies how quickly the SSE checks or updates the secret it's managing.       | `3600` (1 hour)                                             |
   | `enablePartialSecretSync` | When set to `false` a secret will only be updated if every contained item could be fetched from AKV successfully. When `true` each item in a secret will be updated it it was fetched successfully, without regard to the success of other items in the secret. | `true` |

## SecretSync resources

SecretSync resources configure how SSE stores secrets and certificates into the Kubernetes secret store. Each SecretSync resource defines one Kubernetes secret, although it may contain more than one secret.

### example

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

### values 

 - **metadata.name** *(required)*: The name of this SecretSync resource. Note: The automatically created Secret resource will have the same name as this resource.

Within .spec:

 - **serviceAccountName** *(required)*: The Kubernetes service account used to access the Kubernetes secret store. This service account should be federated with the managed identity that has read access to the secrets in Azure Key Vault.
 - **secretProviderClassName** *(required)*: The name of the SecretProviderClass resource that defines which secrets to fetch from Azure Key Vault.
 - **secretObject** *(required)*: Defines how the stored secret resource should be structured.
   - **type** *(required)*: The type of the Kubernetes secret object. Set this field to `Opaque` for a general purpose secret with no imposed structure. See [Types of Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types) for guidance on how the special purpose secret types need to be structured.
   -  **data** *(required)*: A list of data items within the secret resource. There must be at least one item. Each data item must contain these two fields:
       - **sourcePath** *(required)*: The path to an item fetched from AKV. When only one version of the named secret is fetched from AKV, the path is simply `<secret name>`.<br> If more than one version of the named secret is fetched from AKV, then the latest version's sourcePath is `<secret name>/0`, second latest is `<secret name>/1`, etc.<br> When a certificate is fetched from AKV, the sourcePaths will depend on the value of `objectType` in the SecretProviderClass. When the `objectType`` in the SPC` is "secret" then both a certificate and a private key will be available at sourcePaths of `<secret name>/tls.crt` and `<secret name>/tls.key` respectively.
       - **targetKey** *(required)*: The key in the Kubernetes secret object to store the data into.
   - **labels** *(optional)*: A list of key-value pairs of additional labels to apply to the secret object.
   - **annotations** *(optional)*: A list of key-value pairs of additional annotations to apply to the secret object.
   - **forceSynchronization** *(optional)*: Changes to this field trigger SSE to recheck AKV for changes. Kubernetes will be updated as usual if SSE finds updated data. The value of this field does not affect how the SSE behaves.

## SecretProviderClass resources

Soon