---
title: "Securing AKS workloads: Validating Container Image Signatures with Ratify and Azure Policy"
description: Learn how to set up Ratify and Azure policies on Azure Kubernetes Service (AKS) clusters to validate container image signatures during deployment, ensuring the integrity and authenticity of your workloads. 
ms.topic: how-to
author: yizha1
ms.author: yizha1
ms.date: 09/24/2025
ms.service: security
---

# Securing AKS Workloads: Validating Container Image Signatures with Ratify and Azure Policy

## Introduction

Container security is crucial in the cloud-native landscape to protect workloads. To address this, Microsoft introduced the [Containers Secure Supply Chain (CSSC) framework](/azure/security/container-secure-supply-chain/articles/containers-secure-supply-chain-overview.md), enhancing security throughout the lifecycle of container images. One of the stages defined in the CSSC framework is the `Deploy` stage, where container images are deployed to production environments, such as Azure Kubernetes Service (AKS) clusters. Ensuring a secure production environment involves maintaining the integrity and authenticity of container images. This is achieved by signing container images at the Build stage and then verifying them at the Deploy stage, ensuring that only trusted and unaltered images are deployed.

[Ratify](https://ratify.dev/), a [CNCF](https://www.cncf.io/) sandbox project supported by Microsoft, is a robust verification engine that verifies container images security metadata, such as signatures, and only allows the deployment of images that meet your specified policies.

## Scenarios

This section covers two primary scenarios for implementing container image signature validation with Ratify on AKS. The scenarios differ based on how you manage certificates for signing and verification: using Azure Key Vault (AKV) for traditional certificate management or using Microsoft's Trusted Signing service for zero-touch certificate lifecycle management. Choose the scenario that aligns with your current certificate management approach and security requirements.

### Using AKV for certificate management

An image producer builds and pushes container images to the Azure Container Registry (ACR) within CI/CD pipelines. These images are intended for deploying and running cloud-native workloads on AKS clusters by image consumers. The image producer signs container images in ACR using [Notary Project](https://notaryproject.dev) tooling, specifically Notation, within the CI/CD pipelines. The keys and certificates for signing are securely stored in Azure Key Vault (AKV). Once signed, Notary Project signatures are created and stored in ACR, referencing the corresponding images. An image consumer sets up Ratify and policies on the AKS cluster to validate the Notary Project signatures of images during deployment. Images that fail signature validation will be denied from deployment if the policy effect is set to deny effect. This ensures that only trusted and unaltered images are deployed to the AKS cluster.

As the image producer, follow these documents to sign container images in ACR with AKV:

- For signing using self-signed certificates, see [Sign container images with Notation CLI and AKV using self-signed certificates](container-registry-tutorial-sign-build-push.md)
- For signing using CA issued certificates, see [Sign container images with Notation CLI and AKV using CA issued certificates](container-registry-tutorial-sign-trusted-ca.md)
- For signing in Azure DevOps (ADO) pipelines, see [Sign container images in Azure DevOps (ADO) pipelines](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign.md)
- For signing in GitHub workflows, see [Sign container images in GitHub workflows](/azure/security/container-secure-supply-chain/articles/notation-sign-gha.md)

### Using Trusted Signing for certificate management

In this scenario, an image producer signs container images in ACR using certificates managed by Trusted Signing rather than AKV. Because Trusted Signing provides zero-touch certificate lifecycle management, producers no longer need to handle certificate issuance, rotation, or expiration. On the consumer side, while Trusted Signing produces short-lived certificates, image consumers configure timestamping during verification to maintain trust after certificates expire. Additionally, Ratify and cluster policies are configured on AKS to validate signatures at deployment time, and any image that fails validation is blocked if the policy effect is set to deny, ensuring that only trusted and unaltered images are deployed.

As the image producer, see the following guides for signing with Trusted Signing:

- [Sign and verify contaienr images with Notation and Trusted Signing](container-registry-tutorial-sign-verify-notation-trusted-signing.md)
- [Sign container images in GitHub workflows with Notation and Trusted Signing](container-registry-tutorial-github-sign-notation-trusted-signing.md)

This document will guide you, as the image consumer, through the process of verifying container image signatures with Ratify and Azure policy on AKS clusters.

> [!IMPORTANT]
> If you prefer using a managed experience over using open-source Ratify directly, you can opt for the [AKS image integrity policy (public preview)](/azure/aks/image-integrity) to ensure image integrity on your AKS clusters instead.

## Signature validation overview

Here are the high-level steps for signature verification:

1. **Set up identity and access controls for ACR**: Configure the identity used by Ratify to access ACR with the necessary roles.

2. **Set up identity and access controls for AKV**: Configure the identity used by Ratify to access AKV with the necessary roles. Skip this step if images are signed with Trusted Signing.

3. **Set up Ratify on your AKS cluster**: Set up Ratify using Helm chart installation as a standard Kubernetes service.

4. **Set up a custom Azure policy**: Create and assign a custom Azure policy with the desired policy effect: `Deny` or `Audit`.

After following these steps, you can start deploying your workloads to observe the results. With the `Deny` effect policy, only images that have passed signature verification are allowed for deployment, while images that are unsigned or signed by untrusted identities are denied. With the `Audit` effect policy, images can be deployed, but your component will be marked as non-compliant for auditing purposes.

## Prerequisites

* Install and configure the latest [Azure CLI](/cli/azure/install-azure-cli), or run commands in the [Azure Cloud Shell](https://portal.azure.com/#cloudshell/).
* Install [helm](https://helm.sh/docs/intro/install/) for Ratify installation and [kubectl](https://kubernetes.io/docs/reference/kubectl/) for troubleshooting and status checking.
* Create or use an AKS cluster enabled with an OIDC Issuer by following the steps in [Configure an AKS cluster with an OpenID Connect (OIDC) issuer](/azure/aks/use-oidc-issuer). This AKS cluster is where your container images will be deployed, Ratify will be installed, and custom Azure policies will be applied.
* Connect the ACR to the AKS cluster if not already connected by following the steps in [Authenticate with ACR from AKS](/azure/aks/cluster-container-registry-integration). The ACR is where your container images are stored for deployment to your AKS cluster.
* Enable the Azure Policy add-on. To verify that the add-on is installed, or to install it if it is not already, follow the steps in [Azure Policy add-on for AKS](/azure/governance/policy/concepts/policy-for-kubernetes#install-azure-policy-add-on-for-aks).

## Set up identity and access controls

Before installing Ratify on your AKS cluster, you need to establish the proper identity and access controls. Ratify requires access to your ACR to pull container images and signatures, and when using Azure Key Vault for certificate management, it also needs access to retrieve certificates for signature verification. This section guides you through creating a user-assigned managed identity and configuring the necessary permissions for Ratify to operate securely within your Azure environment.

The identity configuration involves:
- Creating or using an existing user-assigned managed identity
- Setting up federated identity credentials to enable workload identity authentication
- Granting appropriate role assignments for ACR access
- Configuring AKV access permissions (when using AKV for certificate management)

### Create or use a user-assigned managed identity

If you don't already have a user-assigned managed identity, follow this [document](/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities?pivots=identity-mi-methods-azcli#create-a-user-assigned-managed-identity-1) to create one. This identity will be used by Ratify to access Azure resources, such as ACR and when applicable, AKV for certificate management.

### Create a federated identity credential for your identity

Set up environment variables:

```shell
export AKS_RG=<aks-resource-group-name>
export AKS_NAME=<aks-name>
export AKS_OIDC_ISSUER=$(az aks show -n $AKS_NAME -g $AKS_RG --query "oidcIssuerProfile.issuerUrl" -otsv)

export IDENTITY_RG=<identity-resource-group-name>
export IDENTITY_NAME=<identity-name>
export IDENTITY_CLIENT_ID=$(az identity show --name  $IDENTITY_NAME --resource-group $IDENTITY_RG --query 'clientId' -o tsv)
export IDENTITY_OBJECT_ID=$(az identity show --name $IDENTITY_NAME --resource-group $IDENTITY_RG --query 'principalId' -otsv)

export RATIFY_NAMESPACE="gatekeeper-system"
export RATIFY_SA_NAME="ratify-admin"
```

> [!NOTE]
> Update the values of the variables `RATIFY_NAMESPACE` and `RATIFY_SA_NAME` if you are not using the default values. Make sure you use the same values during Ratify helm chart installation.

The following command creates a federated credential for your managed identity, allowing it to authenticate using tokens issued by an OIDC issuer, specifically for a Kubernetes service account `RATIFY_SA_NAME` in the namespace `RATIFY_NAMESPACE`.

```shell
az identity federated-credential create \
--name ratify-federated-credential \
--identity-name "$IDENTITY_NAME" \
--resource-group "$IDENTITY_RG" \
--issuer "$AKS_OIDC_ISSUER" \
--subject system:serviceaccount:"$RATIFY_NAMESPACE":"$RATIFY_SA_NAME"
```

### Configure access to ACR

The `AcrPull` role is required for your identity to pull signatures and other container image metadata. Use the following instructions to assign the role:

```shell
export ACR_SUB=<acr-subscription-id>
export ACR_RG=<acr-resource-group>
export ACR_NAME=<acr-name>

az role assignment create \
--role acrpull \
--assignee-object-id ${IDENTITY_OBJECT_ID} \
--scope subscriptions/${ACR_SUB}/resourceGroups/${ACR_RG}/providers/Microsoft.ContainerRegistry/registries/${ACR_NAME}
```

### Configure access to AKV

Skip this step if you use Trusted Signing for certificate management.

The `Key Vault Secrets User` role is required for your identity to fetch the entire certificate chain from your AKV. Use the following instructions to assign the role:

Set up additional environment variables for the AKV resource:

```shell
export AKV_SUB=<acr-subscription-id>
export AKV_RG=<acr-resource-group>
export AKV_NAME=<acr-name>

az role assignment create \
--role "Key Vault Secrets User" \
--assignee ${IDENTITY_OBJECT_ID} \
--scope "/subscriptions/${AKV_SUB}/resourceGroups/${AKV_RG}/providers/Microsoft.KeyVault/vaults/${AKV_NAME}"
```

## Set up Ratify on your AKS cluster with Azure Policy enabled

With the identity and access controls properly configured, you can now install Ratify on your AKS cluster. Ratify operates as a verification engine that integrates with Azure Policy to enforce signature validation policies. The installation process involves deploying Ratify using Helm charts with specific configuration parameters that define how it should verify container image signatures.

This section covers two key aspects of the Ratify setup:
- Understanding the Helm chart parameters required for your certificate management approach (AKV or Trusted Signing)
- Installing Ratify with the appropriate configuration to enable signature verification

The configuration parameters will vary depending on whether you're using AKV or Trusted Signing for certificate management, so ensure you follow the instructions that match your chosen scenario.

### Know your helm chart parameters

When installing the Helm chart for Ratify, you need to pass values to parameters using the `--set` flag or by providing a custom values file. Those values will be used to configure Ratify for signature verification. For a comprehensive list of parameters, refer to the [Ratify Helm chart documentation](https://github.com/notaryproject/ratify/tree/v1.4.0/charts/ratify). 

Configuration differs depending on whether you use **AKV** or **Trusted Signing** for certificate management.

#### [Azure Key Vault (AKV)](#tab/akv)

You need to configure:

- The identity set up previously for accessing ACR and AKV.  
- The certificate stored in AKV for signature verification.  
- A Notary Project trust policy for signature verification, including `registryScopes`, `trustStores`, and `trustedIdentities`.  

See the parameter table below for details:

| Parameter                                       | Description                                                                                        | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------- |
| azureWorkloadIdentity.clientId                  | Specifies the client ID of the Azure Workload Identity                                             | "$IDENTITY_CLIENT_ID"               |
| oras.authProviders.azureWorkloadIdentityEnabled | Enable/disable Azure Workload Identity for ACR authentication                                      | true                                |
| azurekeyvault.enabled                           | Enable/disable fetching certificates from AKV                                                      | true                                |
| azurekeyvault.vaultURI                          | The URI of the AKV resource                                                                        | "https://$AKV_NAME.vault.azure.net" |
| azurekeyvault.tenantId                          | The tenant ID of the AKV resource                                                                  | "$AKV_TENANT_ID"                    |
| azurekeyvault.certificates[0].name              | Name of the certificate                                                                            | "$CERT_NAME"                        |
| notation.trustPolicies[0].registryScopes[0]     | A repository URI that the policy applies to                                                        | "$REPO_URI"                         |
| notation.trustPolicies[0].trustStores[0]        | Trust stores where certificates of type `ca` or `tsa` are stored                                   | ca:azurekeyvault                    |                    
| notation.trustPolicies[0].trustedIdentities[0]  | The subject field of the signing certificate with prefix `x509.subject:` indicating who you trust  | "x509.subject: $SUBJECT"            |

By using timestamping for your images, you can ensure that images signed before the certificate expires can still be verified successfully. Add the following parameters for TSA configuration:

| Parameter                                       | Description                                                                | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- |
| notationCerts[0]                                | The filepath to the PEM formatted TSA root certificate file                | "$TSA_ROOT_CERT_FILEPATH"           |
| notation.trustPolicies[0].trustStores[1]        | Another trust store where the TSA root certificate is stored               | tsa:notationCerts[0]                |

If you have multiple certificates for signature verification, specify additional parameters:

| Parameter                                       | Description                                                                | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- |
| azurekeyvault.certificates[1].name              | Name of the certificate                                                    | "$CERT_NAME_2"                      |
| notation.trustPolicies[0].trustedIdentities[1]  | Another subject field of the signing certificate indicating who you trust  | "x509.subject: $SUBJECT_2"          |

#### [Trusted Signing](#tab/trusted-signing)

You need to configure:

- The identity set up previously for accessing ACR.  
- A Notary Project trust policy for signature verification, including `registryScopes`, `trustStores`, and `trustedIdentities`.  
- A timestamping configuration, since Trusted Signing issues short-lived certificates.  

See the parameter table below for details:

| Parameter                                       | Description                                                                                        | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------- |
| azureWorkloadIdentity.clientId                  | Specifies the client ID of the Azure Workload Identity                                             | "$IDENTITY_CLIENT_ID"               |
| oras.authProviders.azureWorkloadIdentityEnabled | Enable/disable Azure Workload Identity for ACR authentication                                      | true                                |
| notationCerts[0]                                | The filepath to the PEM formatted Trusted Signing root certificate file                            | "$TS_ROOT_CERT_FILEPATH"               |
| notationCerts[1]                                | The filepath to the PEM formatted TSA root certificate file                                        | "$TSA_ROOT_CERT_FILEPATH"           |
| notation.trustPolicies[0].registryScopes[0]     | A repository URI that the policy applies to                                                        | "$REPO_URI"                         |
| notation.trustPolicies[0].trustStores[0]        | Trust stores where the Trusted Signing root certificate is stored                                  | ca:notationCerts[0]                 |
| notation.trustPolicies[0].trustStores[1]        | Trust stores where the TSA root certificate is stored                                              | tsa:notationCerts[1]                |
| notation.trustPolicies[0].trustedIdentities[0]  | The subject field of the Trusted Signing certificate with prefix `x509.subject:` indicating trust  | "x509.subject: $SUBJECT"            |

If you have multiple Trusted Signing certificate profiles, you can add additional trusted identities:

| Parameter                                       | Description                                                                | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- |
| notation.trustPolicies[0].trustedIdentities[1]  | Another subject field of the certificate profile indicating who you trust  | "x509.subject: $SUBJECT_2"          |

---

### Install Ratify helm chart with desired parameters and values

Ensure that the Ratify Helm chart version is at least `1.15.0`, which will install Ratify version `1.4.0` or higher. In this example, helm chart version `1.15.0` is used.

Set up additional environment variables for installation:

#### [Azure Key Vault (AKV)](#tab/akv)

```shell
export CHART_VER="1.15.0"
export REPO_URI="$ACR_NAME.azurecr.io/<namespace>/<repo>"
export SUBJECT="<Subject-of-signing-certificate>"
export AKV_TENANT_ID="$(az account show --query tenantId --output tsv)"

helm repo add ratify https://notaryproject.github.io/ratify
helm repo update

helm install ratify ratify/ratify --atomic --namespace $RATIFY_NAMESPACE --create-namespace --version $CHART_VER --set provider.enableMutation=false --set featureFlags.RATIFY_CERT_ROTATION=true \
--set azureWorkloadIdentity.clientId=$IDENTITY_CLIENT_ID \
--set oras.authProviders.azureWorkloadIdentityEnabled=true \
--set azurekeyvault.enabled=true \
--set azurekeyvault.vaultURI="https://$AKV_NAME.vault.azure.net" \
--set azurekeyvault.certificates[0].name="$CERT_NAME" \
--set azurekeyvault.tenantId="$AKV_TENANT_ID" \  
--set notation.trustPolicies[0].registryScopes[0]="$REPO_URI" \
--set notation.trustPolicies[0].trustStores[0]="ca:azurekeyvault" \
--set notation.trustPolicies[0].trustedIdentities[0]="x509.subject: $SUBJECT"
```

> [!NOTE]
> For timestamping support, you need to specify additional parameters: `--set-file notationCerts[0]="$TSA_ROOT_CERT_FILE"` and `--set notation.trustPolicies[0].trustStores[1]="ca:azurekeyvault"`.

#### [Trusted Signing](#tab/trusted-signing)

By default, the Trusted Signing root certificate and TSA root certificate are provided in `.crt` format.
Before passing them to the Helm installation, you must convert them to **PEM** format.  

On Linux, you can use the following commands to download and convert the certificates:

```shell
export TS_ROOT_CERT_URL="https://www.microsoft.com/pkiops/certs/Microsoft%20Enterprise%20Identity%20Verification%20Root%20Certificate%20Authority%202020.crt"
export TSA_ROOT_CERT_URL="http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt"

# Download and convert Trusted Signing root certificate
curl -o msft-identity-verification-root-cert-2020.crt $TS_ROOT_CERT_URL
openssl x509 -in msft-identity-verification-root-cert-2020.crt -out msft-identity-verification-root-cert-2020.pem -outform PEM
    
# Download and convert TSA root certificate
curl -o msft-tsa-root-certificate-authority-2020.crt $TSA_ROOT_CERT_URL
openssl x509 -in msft-tsa-root-certificate-authority-2020.crt -out msft-tsa-root-certificate-authority-2020.pem -outform PEM
```

Next, set the environment variables and install Ratify with Helm:

```shell
export TS_ROOT_CERT_FILEPATH="msft-identity-verification-root-cert-2020.pem"
export TSA_ROOT_CERT_FILEPATH="msft-tsa-root-certificate-authority-2020.pem"
export CHART_VER="1.15.0"
export REPO_URI="$ACR_NAME.azurecr.io/<namespace>/<repo>"
export SUBJECT="<Subject-of-certificate-profile>"

helm repo add ratify https://notaryproject.github.io/ratify
helm repo update

helm install ratify ratify/ratify --atomic --namespace $RATIFY_NAMESPACE --create-namespace --version $CHART_VER --set provider.enableMutation=false --set featureFlags.RATIFY_CERT_ROTATION=true \
--set azureWorkloadIdentity.clientId=$IDENTITY_CLIENT_ID \
--set oras.authProviders.azureWorkloadIdentityEnabled=true \
--set-file notationCerts[0]=$TS_ROOT_CERT_FILEPATH \
--set-file notationCerts[1]=$TSA_ROOT_CERT_FILEPATH \
--set notation.trustPolicies[0].registryScopes[0]="$REPO_URI" \
--set notation.trustPolicies[0].trustStores[0]="ca:notationCerts[0]" \
--set notation.trustPolicies[0].trustStores[1]="tsa:notationCerts[1]" \
--set notation.trustPolicies[0].trustedIdentities[0]="x509.subject: $SUBJECT"
```

---

> [!IMPORTANT]
> For images that are not linked to a trust policy, signature validation will fail. For instance, if the images are not within the repository `$REPO_URI`, the signature validation for those images will fail. You can add multiple repositories by specifying additional parameters. For example, to add another repository for the trust policy `notation.trustPolicies[0]`, include the parameter `--set notation.trustPolicies[0].registryScopes[1]="$REPO_URI_1"`.

## Set up a custom Azure policy

With Ratify successfully installed and configured on your AKS cluster, the final step is to create and assign an Azure Policy that will enforce signature validation during container deployments. This policy acts as the enforcement mechanism that instructs the cluster to use Ratify for verifying container image signatures before allowing deployments.

Azure Policy offers two enforcement modes:
- **Deny effect**: Blocks deployment of images that fail signature verification, ensuring only trusted images run in your cluster
- **Audit effect**: Allows all deployments but marks non-compliant resources for monitoring and reporting purposes

The Audit effect is particularly useful during initial setup or testing phases, allowing you to validate your configuration without risking service disruptions in production environments.

### Assign a new policy to your AKS cluster

Create a custom Azure policy for signature verification. By default, the policy effect is set to `Deny`, meaning images that fail signature validation will be denied deployment. Alternatively, you can configure the policy effect to `Audit`, allowing images that fail signature verification to be deployed while marking the AKS cluster and related workloads as non-compliant. The `Audit` effect is useful for verifying your signature verification configuration without risking outages due to incorrect settings for your production environment.

```shell
export CUSTOM_POLICY=$(curl -L https://raw.githubusercontent.com/notaryproject/ratify/refs/tags/v1.4.0/library/default/customazurepolicy.json)
export DEFINITION_NAME="ratify-default-custom-policy"
export DEFINITION_ID=$(az policy definition create --name "$DEFINITION_NAME" --rules "$(echo "$CUSTOM_POLICY" | jq .policyRule)" --params "$(echo "$CUSTOM_POLICY" | jq .parameters)" --mode "Microsoft.Kubernetes.Data" --query id -o tsv)
```

Assign the policy to your AKS cluster with the default effect `Deny`.

```shell
export POLICY_SCOPE=$(az aks show -g "$AKS_RG" -n "$AKS_NAME" --query id -o tsv)
az policy assignment create --policy "$DEFINITION_ID" --name "$DEFINITION_NAME" --scope "$POLICY_SCOPE"
```

To change the policy effect to `Audit`, you can pass additional parameter to `az policy assignment create` command. For example:

```shell
az policy assignment create --policy "$DEFINITION_ID" --name "$DEFINITION_NAME" --scope "$POLICY_SCOPE" -p "{\"effect\": {\"value\":\"Audit\"}}"
```

>[!NOTE]
> It will take around 15 minutes to complete the assignment.

Use the following command to check the custom policy status.

```shell
kubectl get constraintTemplate ratifyverification
```

Below is an example of the output for a successful policy assignment:

```text
NAME                 AGE
ratifyverification   11m
```

To make a change on an existing assignment, you need to delete the existing assignment first, make changes, and finally create a new assignment.

## Deploy your images and check the policy effects

Now that you have successfully configured Ratify and assigned the Azure Policy to your AKS cluster, it's time to test the signature validation functionality. This section demonstrates how the policy enforcement works in practice by deploying different types of container images and observing the results.

You'll test three scenarios to validate your setup:
- **Signed images with trusted certificates**: Should deploy successfully
- **Unsigned images**: Should be blocked (with Deny effect) or marked non-compliant (with Audit effect)
- **Images signed with untrusted certificates**: Should be blocked (with Deny effect) or marked non-compliant (with Audit effect)

The behavior you observe will depend on the policy effect you chose during the Azure Policy assignment step. This testing process helps ensure your signature validation is working correctly and provides confidence that only trusted images will be allowed in your production environment.

### Use Deny policy effect

With the `Deny` policy effect, only images signed with trusted identities are allowed for deployment. You can begin deploying your workloads to observe the effects. In this document, we will use the `kubectl` command to deploy a simple pod. Similarly, you can deploy your workloads using a Helm chart or any templates that trigger Helm installation.

Set up environment variables:

```shell
export IMAGE_SIGNED=<signed-image-reference>
export IMAGE_UNSIGNED=<unsigned-image-reference>
export IMAGE_SIGNED_UNTRUSTED=<signed-untrusted-image-reference>
```

Run the following command. Since `$IMAGE_SIGNED` references an image that is signed by a trusted identity and configured in Ratify, it is allowed for deployment.

```shell
kubectl run demo-signed --image=$IMAGE_SIGNED
```

Below is an example of the output for a successful deployment:

```text
pod/demo-signed created
```

`$IMAGE_UNSIGNED` references an image that is not signed. `$IMAGE_SIGNED_UNTRUSTED` references an image that is signed using a different certificate that you will not trust. So, these two images will be denied for deployment. For example, run the following command:

```shell
kubectl run demo-unsigned --image=$IMAGE_UNSIGNED
```

Below is an example of the output for a deployment that is denied:

```text
Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [azurepolicy-ratifyverification-077bac5b63d37da0bc4a] Subject failed verification: $IMAGE_UNSIGNED
```

You can use the following command to output Ratify logs and search the log with text `verification response for subject $IMAGE_UNSIGNED`, check the `errorReason` field to understand the reason for any denied deployment.

```shell
kubectl logs <ratify-pod> -n $RATIFY_NAMESPACE
```

### Use Audit policy effect

With Audit policy effect, unsigned images or images signed with untrusted identities are allowed for deployment. However, the AKS cluster and related components will be marked as `non-compliant`. For more details on how to view non-compliant resources and understand the reasons, see [Get the Azure policy compliance-data](/azure/governance/policy/how-to/get-compliance-data).

## Cleaning Up

Use the following commands to uninstall Ratify and clean up Ratify CRDs:

```shell
helm delete ratify --namespace $RATIFY_NAMESPACE
kubectl delete crd stores.config.ratify.deislabs.io verifiers.config.ratify.deislabs.io certificatestores.config.ratify.deislabs.io policies.config.ratify.deislabs.io keymanagementproviders.config.ratify.deislabs.io namespacedkeymanagementproviders.config.ratify.deislabs.io namespacedpolicies.config.ratify.deislabs.io namespacedstores.config.ratify.deislabs.io namespacedverifiers.config.ratify.deislabs.io
```

Delete the policy assignment and definition using the following commands:

```shell
az policy assignment delete --name "$DEFINITION_NAME" --scope "$POLICY_SCOPE"
az policy definition delete --name "$DEFINITION_NAME"
```

## FAQ

### How can I set up certificates for signature verification if I don't have access to AKV?

In some cases, image consumers may not have access to the certificates used for signature verification. To verify signatures, you will need to download the root CA certificate file in PEM format and specify the related parameters for the Ratify Helm chart installation. Below is an example command similar to the previous installation command, but without any parameters related to AKV certificates. The Notary Project trust store refers to the certificate file that passed in parameter `notationCerts[0]`:

```shell
helm install ratify ratify/ratify --atomic --namespace $RATIFY_NAMESPACE --create-namespace --version $CHART_VER --set provider.enableMutation=false --set featureFlags.RATIFY_CERT_ROTATION=true \
--set azureWorkloadIdentity.clientId=$IDENTITY_CLIENT_ID \
--set oras.authProviders.azureWorkloadIdentityEnabled=true \
--set-file notationCerts[0]="<root-ca-certifice-filepath>"
--set notation.trustPolicies[0].registryScopes[0]="$REPO_URI" \
--set notation.trustPolicies[0].trustStores[0]="ca:notationCerts[0]" \
--set notation.trustPolicies[0].trustedIdentities[0]="x509.subject: $SUBJECT"
```

> [!NOTE]
> Since `notationCerts[0]` is used for the root CA certificate, if you have an additional certificate file for timestamping purpose, make sue you use the correct index. For example,
> `notationCerts[1]` is used for the TSA root certificate file, then use another trust store `notation.trustPolicies[0].trustStores[1]"` with the value `"tsa:notationCerts[1]"`.

### What steps should I take if Azure Policy is disabled in my AKS cluster?

If Azure Policy is disabled on your AKS cluster, you must install [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/) as the policy controller before installing Ratify. 

> [!NOTE]
> Azure Policy should remain disabled, as Gatekeeper conflicts with the Azure Policy add-on on AKS clusters. If you want to enable Azure Policy later on, you need to uninstall Gatekeeper and Ratify, and then follow this document to set up Ratify with Azure Policy enabled.

```shell
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts

helm install gatekeeper/gatekeeper  \
--name-template=gatekeeper \
--namespace gatekeeper-system --create-namespace \
--set enableExternalData=true \
--set validatingWebhookTimeoutSeconds=5 \
--set mutatingWebhookTimeoutSeconds=2 \
--set externaldataProviderResponseCacheTTL=10s
```

Then, install Ratify as described in the previous steps. After installation, enforce policies using the following commands. By default, the policy effect is set to `Deny`. You can refer to the [Gatekeeper violations document](https://open-policy-agent.github.io/gatekeeper/website/docs/violations) to update the `constraint.yaml` for different policy effects.

```shell
kubectl apply -f https://notaryproject.github.io/ratify/library/default/template.yaml
kubectl apply -f https://notaryproject.github.io/ratify/library/default/samples/constraint.yaml
```

### How can I update Ratify configurations after it has been installed?

Ratify configurations are [Kubernetes custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/), allowing you to update these resources without reinstalling Ratify.

- To update AKV related configurations, use the Ratify `KeyManagementProvider` custom resource with the type `azurekeyvault`. To update Trusted Signing related configurations, use the Ratify `KeyManagementProvider` custom resource with the type `inline`. Follow the [documentation](https://ratify.dev/docs/reference/custom%20resources/key-management-providers).
- To update Notary Project trust policies and stores, use the Ratify `Verifier` custom resource. Follow the [documentation](https://ratify.dev/docs/reference/custom%20resources/verifiers).
- To authenticate and interact with ACR (or other OCI-compliant registries), use the Ratify Store custom resource. Follow the [documentation](https://ratify.dev/docs/reference/custom%20resources/stores).

### What should I do if my container images are not signed using the Notation tool?

This document is applicable for verifying Notary Project signatures independently on any tools that can produce Notary Project-compliant signatures. Ratify also supports verifying other types of signatures. For more information, see the [Ratify user guide](https://ratify.dev/docs/1.2/quickstarts/ratify-on-azure#use-cosign-with-keys-stored-in-akv).