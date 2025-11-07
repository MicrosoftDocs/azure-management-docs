---
title: Troubleshooting issues with the Secret Store extension
description: This guide helps you to efficiently diagnose and resolve issues with the Azure Key Vault Secret Store extension.
ms.date: 08/01/2025
ms.topic: troubleshooting
---

# Troubleshooting
The SSE is a Kubernetes deployment that contains a pod with two containers: the controller, which manages storage of secrets in the cluster, and the provider, which accesses secrets from Azure Key Vault (AKV). SSE is configured via `SecretSync` and `SecretProviderClass` resources. In addition to configuration parameters, `SecretSync` classes are updated by the SSE controller with the status of the sync operation, including error messages where appropriate.

## Checking a SecretSync status

The most common issues with the SSE can be investigated by checking the status of the related `SecretSync` object. Use a `kubectl describe ...` command to view the status of a sync object. In addition to the overall configuration, the output includes the secret creation timestamp, the versions of the secret, and detailed status messages for each synchronization event. This output can be used to diagnose connection or configuration errors and to observe when the secret value changes.

```bash
kubectl describe secretsync <secret-sync-name> -n ${KUBERNETES_NAMESPACE}
```

## SecretSync status reasons

The following table lists common status reasons, their meanings, and potential troubleshooting steps to resolve errors.

| SecretSync Status Reason     | Details      | Steps to fix/investigate further    |
|------------|--------------|-------------------------------------|
| `UpdateNoValueChangeSucceeded` | The secret in Kubernetes is fully up-to-date to the AKV version. No change was needed during the latest check. | n/a |
| `UpdateValueChangeOrForceUpdateSucceeded` | The whole secret in Kubernetes was successfully updated to the AKV version during the latest check. | n/a |
| `PartialSync` | Some items in the secret could not be updated. | Investigate further by looking at the `status.conditions.message` field of the `SecretSync` object. This field contains a stringified json summary of the success or failure for each item in the secret. |
| `ProviderError` | Secret creation failed due to some issue with the provider (connection to Azure Key Vault). This failure could be due to internet connectivity, insufficient permissions for the identity syncing secrets, misconfiguration of the `SecretProviderClass`, or other issues. | Investigate first as with `PartialSync`, next look at the logs of the provider using the following commands: <br>```kubectl get pods -n azure-secret-store``` <br>```kubectl logs <secret-sync-controller-pod-name> -n azure-secret-store --container='provider-azure-installer'``` |
| `InvalidClusterSecretLabelError`<br>`InvalidClusterSecretAnnotationError` | A secret already exists with this name that SSE does not manage. | Remove the secret to allow the SSE to recreate the secret: ```kubectl delete secret <secret-name>``` <br>To force the SSE to recreate the secret faster than the configured rotation poll interval, delete the `SecretSync` object (```kubectl delete secretsync <secret-name>```) and reapply the secret sync class (```kubectl apply -f <path_to_secret_sync>```). |
| `UserInputValidationFailed` | Secret update failed because the secret sync class was configured incorrectly (such as an invalid secret type). | Review the secret sync class definition and correct any errors. Then, delete the `SecretSync` object (```kubectl delete secretsync <secret-name>```), delete the secret sync class (```kubectl delete -f <path_to_secret_sync>```), and reapply the secret sync class (```kubectl apply -f <path_to_secret_sync>```). |
| `ControllerSpcError` | Secret update failed because the SSE failed to get the provider class or the provider class is misconfigured. | Review the provider class and correct any errors. Then, delete the `SecretSync` object (```kubectl delete secretsync <secret-name>```), delete the provider class (```kubectl delete -f <path_to_provider>```), and reapply the provider class (```kubectl apply -f <path_to_provider>```). |
| `ControllerInternalError`<br>`ValidatingAdmissionPolicyCheckFailed`<br>`ControllerSyncFailed`  | Secret update failed due to an internal error in the SSE. | Check the SSE logs or the events for more information: <br>```kubectl get pods -n azure-secret-store``` <br>```kubectl logs <secret-sync-controller-pod-name> -n azure-secret-store --container='manager'``` |
| `UnknownError`| Secret update failed during patching the Kubernetes secret value. This failure might occur if someone other than the SSE modifies the secret, or if there were issues during an update of the SSE. | Try deleting the secret and `SecretSync` object, then let the SSE recreate the secret by reapplying the `SecretSync` object: <br>```kubectl delete secret <secret-name>``` <br>```kubectl delete secretsync <secret-name>```  <br>```kubectl apply -f <path_to_secret_sync>```<br>If this does not help, follow the steps to inspect the logs as with a  `ControllerInternalError`. |

## Network access issues

If you see a ```ProviderError``` status reason with a message indicating a timeout or inability to reach your key vault endpoint (```<vault_name>.vault.azure.net```), there is a likely network access issue to resolve. Possible network access issues include:

### Azure Key Vault firewall configuration
By default Azure Key Vault has a permissive firewall configuration and can be accessed via the public internet, but it is good practice to limit network access to Azure Key Vault. Double check that your key vault firewall configuration permits access for your cluster. See [Network security for Azure Key Vault](/azure/key-vault/general/network-security)

#### Azure Arc Gateway (preview)
Azure Arc gateway (preview) does not provide network connectivity to Azure Key Vault. You must independently allow network access to ```<vault_name>.vault.azure.net``` so that SSE can synchronize secrets. See related [Arc gateway](/azure/azure-arc/kubernetes/arc-gateway-simplify-networking) and [Azure Key Vault](/azure/key-vault/general/access-behind-firewall) documentation.


### Azure Private Link
If using Azure Arc Private Link: Ensure your DNS and network allow the Arc-connected cluster to reach the Key Vault’s private endpoint. Verify that DNS resolution for the vault name returns the private IP and that the private link IP is routed correctly from your cluster. See [Diagnose private link configuration issues on Azure Key Vault](/azure/key-vault/general/private-link-diagnostics).

## Slow or missing updates

Secret Store Extension waits a set interval between checking Azure Key Vault for updates. When a secret is updated in AKV, it is only downloaded to the cluster when the interval expires and SSE checks again. The default interval is one hour (3,600 seconds), this is set via the configuration setting `rotationPollIntervalInSeconds`. See [configuration reference](secret-store-extension-reference.md#arc-extension-configuration-settings).

To force the SSE to update a secret immediately, update any part of the `spec` field within the `SecretSync` resource. A special field `forceSynchronization` can be set within a `spec` that does not have any effect on the configuration; SSE updates the secret immediately if the value of `forceSynchronization` is modified. See [SecretSync reference](secret-store-extension-reference.md#secretsync-resources)) for an example.

## Azure Key Vault Rate Limiting

Azure Key Vault has hard limits on the rate of transactions it can service before it throttles requests. See [Azure Key Vault service limits](/azure/key-vault/general/service-limits). All authenticated requests to AKV count towards throttling limits even if they are unsuccessful. This means that AKV can be kept in a throttling state for an indefinite time if clusters continue to make requests. It becomes increasingly likely for AKV to throttle if there are factors that would cause multiple clusters to fetch from AKV simultaneously. For example, if a CI/CD pipeline pushes out an update to clusters' SecretSync resources at once, by default all affected clusters will attempt to refetch immediately. The aggregate demand for secrets must be substantially below AKV's maximum capacity to avoid throttling. 

Some deployments are unlikely to cause AKV to throttle. If the number of clusters multiplied by the number of secrets fetched per cluster is much lower than AKV's ten-second transaction limit (4,000), then throttling is unlikely. For example, A 20 cluster deployment with 10 secrets per cluster is very unlikely to encounter AKV throttling, as 10x20 is much less than 4,000. If throttling is encountered in this situation, double check other loading on the same key vault, and the number of secrets being fetched.

However, with a larger deployments, such as 2,000 clusters each fetching 20 secrets, AKV is very likely to throttle from time to time. In this situation, consider enabling the `jitterSeconds` setting (see [configuration reference](secret-store-extension-reference.md#arc-extension-configuration-settings)). The `jitterSeconds` setting adds a randomized delay before fetching secrets from a SecretSync resource, spreading the deployment's load on AKV over time. When `jitterSeconds` is enabled the worst-case time to attempt a refresh for a secret from AKV is `rotationPollIntervalInSeconds`+`jitterSeconds`. Although `jitterSeconds` cannot _guarantee_ AKV will not be overwhelmed, the probability can be reduced to practically zero.

Choosing an appropriate `jitterSeconds`:

### [Basic](#tab/basic-jitter)

The easiest way to set `jitterSeconds` is to use the longest acceptable time.

1. Decide what is the maximum acceptable time between refreshing secrets for your application. Two hours (7,200 seconds) is a reasonable starting point.
1. Set `rotationPollIntervalInSeconds` to the minimum time between refreshes, or keep the default one hour (3,600 seconds).
1. Subtract `rotationPollIntervalInSeconds` from your maximum acceptable refresh time in seconds to calculate your `jitterSeconds` value.

In this example, `jitterSeconds` would be set to 3,600 seconds. Secrets would then be fetched from AKV at a random time between one and two hours after the last fetch.

For very large deployments you should double check that the chosen jitter is realistic: the number of clusters multiplied by the number of secrets fetched per cluster must be much lower than the total number of secrets AKV can provide in your `jitterSeconds` interval.

### [Lookup from a table](#tab/table-lookup)

The following table provides `jitterSeconds` values that will give a (much) less than 0.01% chance of causing AKV to throttle each time your whole deployment refreshes. Even if AKV does throttle, it is highly likely to recover quickly leaving no visible impact on secret fetching.

To find an appropriate `jitterSeconds` for your deployment, first chose the column with the smallest number of clusters that's larger than your deployment, then chose the row with the smallest number of secrets that's larger than the number of secrets used by each of your clusters. For example, for a 700 cluster deployment with 30 secrets each, lookup the value in the '1000' column and the '50' row, giving the suggested value of 760 seconds for `jitterSeconds`. In this example the real chance of overwhelming AKV is 0.00000000015%; extremely unlikely.

| Secrets needed   | 10 clusters | 20 clusters | 50 clusters | 100 clusters | 200 clusters | 500 clusters | 1,000 clusters | 2,000 clusters  | 5,000 clusters  | 10,000 clusters |
| -- | -- | -- | -- | --- | --- | --- | ---- | ----- | ----- | ----- |
| **5 secrets** | 0  | 0  | 1  | 2   | 4   | 10  | 20   | 39    | 98    | 200   |
| **10 secrets** | 0  | 1  | 2  | 5   | 10  | 24  | 49   | 97    | 240   | 490   |
| **20 secrets** | 1  | 3  | 7  | 14  | 27  | 68  | 140  | 270   | 680   | 1,400 |
| **50 secrets** | 8  | 15 | 38 | 76  | 150 | 380 | 760  | 1,500 | 3,800 | 7,600 |

### [Calculation](#tab/calculation)

Calculating a sensible jitter value requires some statistical work. We want to find the shortest time such that if the clusters' fetches from AKV are uniformly distributed, the chance of overwhelming AKV in any given second is lower than our acceptable threshold.

Three inputs are needed to calculate the jitter for your deployment: Number of clusters, number of secrets required by each cluster, and the acceptable chance of overwhelming AKV.

Using Excel as our calculation tool, follow these steps:
1. Put your number of clusters, secrets for each cluster and acceptable risk overwhelming AKV into cells `A1`, `A2`, `A3` respectively.
1. Calculate the Z-score for your chosen probability. Put this into cell `A4`.
    ```Excel
    =ABS(NORM.S.INV(A3))
    ```
2. Calculate maximum number of clusters that can update in a second before causing AKV to throttle. This is our 'threshold', value. Put this in cell `A5`.
    ```Excel
    =400/A2
    ```
1. Calculate the mean number of clusters we would like to fetch from AKV each second. Put this expression into cell `A6`. (This is the poisson lambda value. We use a Wilson-Hilferty approximation because a normal approximation is not accurate enough as our threshold values can be small.) 
    ```Excel
    =A5 * (1 - (1/(9*A5)) - (A4/(3*SQRT(A5))))^3
    ```
1. Finally, calculate the jitter value in seconds by dividing the number of clusters by the mean number we would like to fetch per second. Put this expression in cell `A7`
    ```Excel
    =A1/A6
    ```

Optionally, you can verify your previous calculations by calculating the chance that your jitter will overwhelm AKV. Add this expression to cell `A8`:
```Excel
=1-POISSON.DIST(A5,A1/A7,TRUE)
```

Example: For a deployment with 700 clusters, 30 secrets per cluster, and a 0.01% acceptable chance to overwhelm AKV, the calculated jitter is 190 seconds. The actual chance to overwhelm AKV is 0.0034%.

---

Other options to reduce likelihood of AKV throttling include:

### Decreasing the poll frequency

Checking AKV for updated secrets counts towards the rate limits in the same way as fetching secrets for the first time. Increasing `rotationPollIntervalInSeconds` (see [configuration reference](secret-store-extension-reference.md#arc-extension-configuration-settings)) can reduce the aggregate demand on AKV. 

### Add additional key vaults

Additional key vault instances can be added to increase the possible number of secret fetches per second, but beware that an Azure subscription has an overall limit of five times a single AKV limit, shared across all key vaults. See [Azure Key Vault service limits](/azure/key-vault/general/service-limits). 