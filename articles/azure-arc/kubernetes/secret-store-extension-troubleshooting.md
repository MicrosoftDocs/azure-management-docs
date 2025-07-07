---
title: Troubleshooting issues with the Secret Store extension
description: This guide will help you to efficiently diagnose and resolve issues with the Azure Key Vault Secret Store extension.
ms.date: 08/01/2025
---

# Troubleshooting Guide
The SSE is a Kubernetes deployment that contains a pod with two containers: the controller, which manages storing secrets in the cluster, and the provider, which manages access to, and pulling secrets from, the Azure Key Vault. Each synchronized secret has a `SecretSync` object that contains the status of the synchronization of that secret from Azure Key Vault to the cluster secret store.

### View last sync status

To view the status of the most recent synchronization for a given secret, use the `kubectl describe` command for the `SecretSync` object. The output includes the secret creation timestamp, the versions of the secret, and detailed status messages for each synchronization event. This output can be used to diagnose connection or configuration errors, and to observe when the secret value changes.

```bash
kubectl describe secretsync secret-sync-name -n ${KUBERNETES_NAMESPACE}
```

## SecretSync Statuses

To troubleshoot an issue, start by looking at the state of the `SecretSync` object, as described in [View last sync status](#view-last-sync-status).  The following table lists common status reasons, their meanings, and potential troubleshooting steps to resolve errors.

| SecretSync Status Reason     | Details      | Steps to fix/investigate further    |
|------------|--------------|-------------------------------------|
| `UpdateNoValueChangeSucceeded` | The secret in Kubernetes is fully up-to-date to the AKV version. No change was needed during the latest check. | n/a |
| `UpdateValueChangeOrForceUpdateSucceeded` | The whole secret in Kubernetes was successfully updated to the AKV version during the latest check. | n/a |
| `PartialSync` | Some items in the secret could not be updated. | Investigate further by looking at the `status.conditions.message` field of the `SecretSync` object. This field will contain a stringified json summary of the success or failure for each item in the secret. |
| `ProviderError` | Secret creation failed due to some issue with the provider (connection to Azure Key Vault). This failure could be due to internet connectivity, insufficient permissions for the identity syncing secrets, misconfiguration of the `SecretProviderClass`, or other issues. | Investigate first as with `PartialSync`, next look at the logs of the provider using the following commands: <br>```kubectl get pods -n azure-secret-store``` <br>```kubectl logs <secret-sync-controller-pod-name> -n azure-secret-store --container='provider-azure-installer'``` |
| `InvalidClusterSecretLabelError`<br>`InvalidClusterSecretAnnotationError` | A secret already exists with this name that is not managed by the SSE. | Remove the secret to allow the SSE to recreate the secret: ```kubectl delete secret <secret-name>``` <br>To force the SSE to recreate the secret faster than the configured rotation poll interval, delete the `SecretSync` object (```kubectl delete secretsync <secret-name>```) and reapply the secret sync class (```kubectl apply -f <path_to_secret_sync>```). |
| `UserInputValidationFailed` | Secret update failed because the secret sync class was configured incorrectly (such as an invalid secret type). | Review the secret sync class definition and correct any errors. Then, delete the `SecretSync` object (```kubectl delete secretsync <secret-name>```), delete the secret sync class (```kubectl delete -f <path_to_secret_sync>```), and reapply the secret sync class (```kubectl apply -f <path_to_secret_sync>```). |
| `ControllerSpcError` | Secret update failed because the SSE failed to get the provider class or the provider class is misconfigured. | Review the provider class and correct any errors. Then, delete the `SecretSync` object (```kubectl delete secretsync <secret-name>```), delete the provider class (```kubectl delete -f <path_to_provider>```), and reapply the provider class (```kubectl apply -f <path_to_provider>```). |
| `ControllerInternalError`<br>`ValidatingAdmissionPolicyCheckFailed`<br>`ControllerSyncFailed`  | Secret update failed due to an internal error in the SSE. | Check the SSE logs or the events for more information: <br>```kubectl get pods -n azure-secret-store``` <br>```kubectl logs <secret-sync-controller-pod-name> -n azure-secret-store --container='manager'``` |
| `UnknownError`| Secret update failed during patching the Kubernetes secret value. This failure might occur if the secret was modified by someone other than the SSE or if there were issues during an update of the SSE. | Try deleting the secret and `SecretSync` object, then let the SSE recreate the secret by reapplying the `SecretSync` object: <br>```kubectl delete secret <secret-name>``` <br>```kubectl delete secretsync <secret-name>```  <br>```kubectl apply -f <path_to_secret_sync>```<br>If this does not help, follow the steps to inspect the logs as with a  `ControllerInternalError`. |