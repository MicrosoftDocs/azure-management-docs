---
title: "Deploy and configure workload identity federation in Azure Arc-enabled Kubernetes (preview)"
ms.date: 11/19/2024
ms.topic: how-to
ms.custom:
  - ignite-2024
description: "Workload identity federation can be used with Azure Arc-enabled Kubernetes clusters."
# Customer intent: As a Kubernetes administrator, I want to deploy and configure workload identity federation on my Azure Arc-enabled Kubernetes cluster, so that I can enable secure identity management for my workloads and streamline authentication with Azure services.
---

# Deploy and configure workload identity federation in Azure Arc-enabled Kubernetes (preview)

You can enable the [workload identity feature](conceptual-workload-identity.md) on an Azure Arc-enabled Kubernetes cluster by using Azure CLI. The process follows these high-level steps:

1. Enable the workload identity feature on a new or existing Arc-enabled Kubernetes cluster.
1. Create a managed identity (or app registration) and Kubernetes service account.
1. Configure the managed identity for token federation.
1. Configure service account annotations and application pod labels to use workload identity.  
1. Configure workload identity settings on the Kubernetes cluster.
1. Disable workload identity on the cluster.

For an overview of this feature, see [Workload identity federation in Azure Arc-enabled Kubernetes (preview)](conceptual-workload-identity.md).

> [!IMPORTANT]
> The Azure Arc workload identity federation feature is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

> [!TIP]
> This article describes the steps required to deploy and configure workload identity on an Arc-enabled Kubernetes cluster. To learn how to enable workload identity on other types of clusters, see the following articles:
>
> - [Azure Kubernetes Service (AKS)](/azure/aks/workload-identity-deploy-cluster)
> - [AKS enabled by Azure Arc](/azure/aks/hybrid/workload-identity)
> - [AKS Edge Essentials](/azure/aks/hybrid/aks-edge-workload-identity)

## Prerequisites

- Workload identity for Azure Arc-enabled Kubernetes clusters (preview) is supported on the following Kubernetes distributions:
  - Ubuntu Linux cluster running K3s  
  - AKS on Edge Essentials
  - AKS on HCI 23H2

To use the workload identity feature, you must have Azure CLI version 2.64 or higher, and `az connectedk8s` version 1.10.0 or higher. Be sure to update your Azure CLI version before updating your `az connectedk8s` version. If you use Azure Cloud Shell, the latest version of Azure CLI will be installed.

## Enable workload identity on your cluster

Follow the appropriate steps to enable the workload identity feature for either a new Arc-enabled Kubernetes cluster or an existing one. In both cases, make sure to replace the name and resource group with your values, and configure parameters as desired.

|Parameter  |Description  |Required  |
|---------|---------|---------|
|`--enable-oidc-issuer`   |Generates and hosts OIDC issuer URL which is a publicly accessible URL that allows the API server to find public signing keys for verifying tokens.           | Required        |
|`--enable-workload-identity`     | Installs a mutating admission webhook which projects a signed service account token to a well-known path and injects authentication-related environment variables to the application pods based on the settings of the annotated service account. For a new cluster, if this parameter isn't enabled, you must mount a projected volume at a well-known path that exposes the signed service account token to the path.         | Optional       |

### Set environment variables

For convenience, the environment variables defined below are referenced in the examples in this article. Replace these values with your own values:

```azurecli
export RESOURCE_GROUP="myRG"
export LOCATION="eastus"
export CLUSTER_NAME="mycluster"
export SERVICE_ACCOUNT_NAMESPACE="myKubernetesnamespace"
export SERVICE_ACCOUNT_NAME="mysa"
export SUBSCRIPTION="$(az account show --query id --output tsv)"
export USER_ASSIGNED_IDENTITY_NAME="myIdentity"
export FEDERATED_IDENTITY_CREDENTIAL_NAME="myFedIdentity"
```

To create an Azure Arc-enabled cluster with workload identity enabled, use the following command:

```azurecli
az connectedk8s connect --name "${CLUSTER_NAME}" --resource-group "${RESOURCE_GROUP}" --enable-oidc-issuer –-enable-workload-identity
```

To enable workload identity on an existing Arc-enabled Kubernetes cluster, use the `update` command.  

```azurecli
az connectedk8s update --name "${CLUSTER_NAME}" --resource-group "${RESOURCE_GROUP}" --enable-oidc-issuer --enable-workload-identity
```

## Retrieve the OIDC issuer URL

Fetch the OIDC issuer URL and save it to an environmental variable. This issuer URL will be used in the following step.

```azurecli
export OIDC_ISSUER="$(az connectedk8s show --name "${CLUSTER_NAME}" --resource-group "${RESOURCE_GROUP}" \ 
    --query "oidcIssuerProfile.issuerUrl" \  
    --output tsv)"
```

To view the environment variable, enter `echo ${OIDC_ISSUER}`. The environment variable should contain the issuer URL, similar to the following example:

`https://northamerica.oic.prod-arc.azure.com/00000000-0000-0000-0000-000000000000/12345678-1234-1234-1234-123456789123/`

By default, the issuer is set to use the base URL `https://{region}.oic.prod-arc.azure.com/{tenant_id}/{uuid}`, where the value for `{region}` matches the location where the Arc-enabled Kubernetes cluster is created. The value `{uuid}` represents the OpenID Connect (OIDC) key, which is an immutable, randomly generated guid for each cluster.

## Create a managed identity

Use the `az identity create` command to create a user-assigned managed identity. With workload identity, a trust relationship is established between the token of the user-assigned management identity and the Kubernetes cluster's service account token.  

```azurecli
az identity create \ 
    --name "${USER_ASSIGNED_IDENTITY_NAME}" \
    --resource-group "${RESOURCE_GROUP}" \
    --location "${LOCATION}" \
    --subscription "${SUBSCRIPTION}"
```

Fetch the managed identity's client ID and store in an environment variable.

```azurecli
export USER_ASSIGNED_CLIENT_ID="$(az identity show \ 
    --resource-group "${RESOURCE_GROUP}" \
    --name "${USER_ASSIGNED_IDENTITY_NAME}" \
    --query 'clientId' \
    --output tsv)"
```

## Create a Kubernetes service account

Create a Kubernetes service account and annotate it with the client ID of the managed identity created in the previous step. The signed tokens associated with the Kubernetes service account will be exchanged for a Microsoft Entra ID token after the trust relationship is established between the two.

Apply the following YAML snippet to create a service account with workload identity annotation added.  

```yml
apiVersion: v1 
kind: ServiceAccount 
metadata: 
  annotations: 
    azure.workload.identity/client-id: "${USER_ASSIGNED_CLIENT_ID}" 
  name: "${SERVICE_ACCOUNT_NAME}" 
  namespace: "${SERVICE_ACCOUNT_NAMESPACE}" 
```

## Create the federated identity credential

Use the [`az identity federated-credential create` command](/cli/azure/identity/federated-credential#az-identity-federated-credential-create) to create the federated identity credential between the managed identity, the service account issuer, and the subject. This step establishes the trust relationship between the Kubernetes cluster and Microsoft Entra for exchanging tokens. For more information about federated identity credentials in Microsoft Entra, see [Overview of federated identity credentials in Microsoft Entra ID](/graph/api/resources/federatedidentitycredentials-overview).

```azurecli
az identity federated-credential create \ 
    --name ${FEDERATED_IDENTITY_CREDENTIAL_NAME} \ 
    --identity-name "${USER_ASSIGNED_IDENTITY_NAME}" \ 
    --resource-group "${RESOURCE_GROUP}" \ 
    --issuer "${OIDC_ISSUER}" \ 
    --subject system:serviceaccount:"${SERVICE_ACCOUNT_NAMESPACE}":"${SERVICE_ACCOUNT_NAME}" \ 
    --audience api://AzureADTokenExchange 
```

> [!NOTE]
> After the federal identity credential is added, it takes a few seconds to propagate. If a token request is made immediately after adding the federated identity credential, the request might fail until the cache is refreshed. To avoid this issue, add a slight delay in your scripts after adding the federated identity credential.

## Configure service account annotations and pod labels

The following service account and pod annotations are available for configuring workload identity based on application requirements. Pod label specified below is mandatory if `–-enable-workload-identity` is set to `true`.

### Service account annotations

All service account annotations are optional. If an annotation isn't specified, the default value will be used.

|Annotation  |Description  |Default  |
|---------|---------|---------|
|`azure.workload.identity/client-id`     |Microsoft Entra application client ID to be used with the pod.          |         |
|`azure.workload.identity/tenant-id`     |Azure tenant ID where the Microsoft Entra application is registered.         |`AZURE_TENANT_ID` environment variable extracted from `azure-wi-webhook-config` ConfigMap.          |
|`azure.workload.identity/service-account-token-expiration`     | `expirationSeconds` field for the projected service account token. Configure to prevent downtime caused by errors during service account token refresh. Kubernetes service account token expiry isn't correlated with Microsoft Entra tokens. Microsoft Entra tokens expire 24 hours after they're issued.         |3600 (supported range is 3600-86400)          |

### Pod labels

|Annotation  |Description  |Recommended value  |Required |
|---------|---------|---------|---------|
|`azure.workload.identity/use` |Required in the pod template spec. If `–-enable-workload-identity` is set to `true`, only pods with this label are mutated by the mutating admission webhook to inject the Azure-specific environment variables and the projected service account token volume. | `true` | Yes |

### Pod annotations

All pod annotations are optional. If an annotation isn't specified, the default value will be used.

|Annotation  |Description  |Default  |
|---------|---------|---------|
|`azure.workload.identity/service-account-token-expiration`     | `expirationSeconds` field for the projected service account token. Configure to prevent downtime caused by errors during service account token refresh. Kubernetes service account token expiry isn't correlated with Microsoft Entra tokens. Microsoft Entra tokens expire 24 hours after they're issued.         |3600 (supported range is 3600-86400)          |
|`azure.workload.identity/skip-containers` |Represents a semi-colon-separated list of containers to skip adding projected service account token volume. For example: `container1;container2`. |By default, the projected service account token volume is added to all containers if the pod is labeled with `azure.workload.identity/use: true`.|

## Configure workload identity settings on the Kubernetes cluster

 The API server on the Kubernetes cluster needs to be configured to issue service account tokens that include the publicly accessible OIDC issuer URL (so that Entra knows where to find the public keys to validate the token).

To configure workload identity settings on Ubuntu Linux with K3s, follow the below steps to complete the configuration:  

1. Create k3s config file.
1. Edit `/etc/rancher/k3s/config.yaml` to add these settings:

   ```yml
      `kube-apiserver-arg:  
        - 'service-account-issuer=${OIDC_ISSUER}'
        - 'service-account-max-token-expiration=24h'`
   ```

1. Save the config.yaml.
1. Restart the k3s API server using the command `systemctl restart k3s`.

   We recommend rotating service account keys frequently. For more information, see [Service-Account Issuer Key Rotation](https://docs.k3s.io/cli/certificate#service-account-issuer-key-rotation).

## Disable workload identity

To disable workload identity feature on an Azure Arc-enabled Kubernetes cluster, run the following command:

```azurecli
az connectedk8s update
    --resource-group "${RESOURCE_GROUP}"
    --name "${CLUSTER_NAME}"
    --disable-workload-identity
```

## Next steps

- Explore a [sample for configuring an application to use workload identity](https://azure.github.io/azure-workload-identity/docs/quick-start.html).
- Help to protect your cluster in other ways by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).