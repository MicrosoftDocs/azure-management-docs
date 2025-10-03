---
title: Validate Container Image Signatures with Ratify and Azure Policy
description: Learn how to set up Ratify and Azure policies on Azure Kubernetes Service (AKS) clusters to validate container image signatures during deployment.
ms.topic: how-to
author: yizha1
ms.author: yizha1
ms.date: 09/24/2025
ms.service: security
---

# Validating container image signatures by using Ratify and Azure Policy

Container security is crucial in the cloud-native landscape to help protect workloads. To enhance security throughout the lifecycle of container images, Microsoft introduced the [Containers Secure Supply Chain (CSSC) framework](/azure/security/container-secure-supply-chain/articles/container-secure-supply-chain-implementation/containers-secure-supply-chain-overview). In the Deploy stage of the framework, container images are deployed to production environments, such as Azure Kubernetes Service (AKS) clusters.

Ensuring a secure production environment involves maintaining the integrity and authenticity of container images. Signing container images at the Build stage and then verifying them at the Deploy stage helps to ensure that only trusted and unaltered images are deployed.

[Ratify](https://ratify.dev/) is a [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/) sandbox project that Microsoft supports. It's a robust verification engine that verifies the security metadata of container images, such as signatures. It allows only the deployment of images that meet your specified policies.

## Scenarios

This article covers two primary scenarios for implementing container image signature verification by using Ratify on AKS. The scenarios differ based on how you manage certificates for signing and verification: using Azure Key Vault for traditional certificate management or using the Microsoft Trusted Signing service for zero-touch certificate lifecycle management. Choose the scenario that aligns with your current certificate management approach and security requirements.

### Use Key Vault for certificate management

An image producer builds and pushes container images to Azure Container Registry within continuous integration and continuous delivery (CI/CD) pipelines. These images are intended for image consumers to deploy and run cloud-native workloads on AKS clusters.

The image producer signs container images in Container Registry by using [Notary Project](https://notaryproject.dev) tooling, specifically Notation, within the CI/CD pipelines. The keys and certificates for signing are securely stored in Key Vault.

Notary Project signatures are created and stored in Container Registry, where they reference the corresponding images. An image consumer sets up Ratify and policies on the AKS cluster to validate the Notary Project signatures of images during deployment. Images that fail signature verification are denied from deployment if the policy effect is set to `Deny`. This system ensures that only trusted and unaltered images are deployed to the AKS cluster.

As the image producer, you can follow these articles to sign container images in Container Registry by using Key Vault:

- For signing via self-signed certificates, see [Sign container images by using Notation, Azure Key Vault, and a self-signed certificate](container-registry-tutorial-sign-build-push.md).
- For signing via certificates issued by a certificate authority (CA), see [Sign container images by using Notation, Azure Key Vault, and a CA-issued certificate](container-registry-tutorial-sign-trusted-ca.md).
- For signing in Azure Pipelines, see [Sign container images in Azure Pipelines](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign).
- For signing in GitHub workflows, see [Sign container images in GitHub workflows](/azure/security/container-secure-supply-chain/articles/notation-sign-gha).

### Use Trusted Signing for certificate management

In this scenario, an image producer signs container images in Container Registry by using certificates managed by Trusted Signing rather than Key Vault. Because Trusted Signing provides zero-touch certificate lifecycle management, producers no longer need to handle certificate issuance, rotation, or expiration.

On the consumer side, Trusted Signing produces short-lived certificates. Image consumers configure timestamping during verification to maintain trust after certificates expire.

Additionally, Ratify and cluster policies are configured on AKS to validate signatures at deployment time. Any image that fails validation is blocked if the policy effect is set to `Deny`. The blocking ensures that only trusted and unaltered images are deployed.

As the image producer, you can follow these articles for signing container images by using Trusted Signing:

- [Sign container images by using Notation and Trusted Signing (preview)](container-registry-tutorial-sign-verify-notation-trusted-signing.md)
- [Sign container images in GitHub workflows by using Notation and Trusted Signing](container-registry-tutorial-github-sign-notation-trusted-signing.md)

This article will guide you, as the image consumer, through the process of verifying container image signatures by using Ratify and Azure Policy on AKS clusters.

> [!IMPORTANT]
> If you prefer using a managed experience over using open-source Ratify directly, you can instead opt for the [AKS image integrity policy (preview)](/azure/aks/image-integrity) to ensure image integrity on your AKS clusters.

## Signature verification overview

Here are the high-level steps for signature verification:

1. **Set up identity and access controls for Container Registry**: Configure the identity that Ratify uses to access Container Registry with the necessary roles.

2. **Set up identity and access controls for Key Vault**: Configure the identity that Ratify usues to access Key Vault with the necessary roles. Skip this step if images are signed via Trusted Signing.

3. **Set up Ratify on your AKS cluster**: Set up Ratify by using Helm chart installation as a standard Kubernetes service.

4. **Set up a custom Azure policy**: Create and assign a custom Azure policy with the desired policy effect: `Deny` or `Audit`.

After you follow these steps, you can start deploying your workloads to observe the results:

- With the `Deny` policy effect, only images that have passed signature verification are allowed for deployment. Images that are unsigned or signed by untrusted identities are denied.
- With the `Audit` policy effect, images can be deployed, but your component are marked as noncompliant for auditing purposes.

## Prerequisites

- Install and configure the latest [Azure CLI](/cli/azure/install-azure-cli) version, or run commands in [Azure Cloud Shell](https://portal.azure.com/#cloudshell/).
- Install [Helm](https://helm.sh/docs/intro/install/) for Ratify installation and [kubectl](https://kubernetes.io/docs/reference/kubectl/) for troubleshooting and status checking.
- Create or use an AKS cluster enabled with an OpenID Connect (OIDC) issuer by following the steps in [Create an OpenID Connect provider on Azure Kubernetes Service](/azure/aks/use-oidc-issuer). This AKS cluster is where your container images are deployed, Ratify is installed, and custom Azure policies are applied.
- Connect Container Registry to the AKS cluster (if it's not already connected) by following the steps in [Authenticate with Azure Container Registry from Azure Kubernetes Service](/azure/aks/cluster-container-registry-integration). Container Registry is where your container images are stored for deployment to your AKS cluster.
- Enable the Azure Policy add-on. To verify that the add-on is installed, or to install it if it isn't already, follow the steps in [Azure Policy add-on for AKS](/azure/governance/policy/concepts/policy-for-kubernetes#install-azure-policy-add-on-for-aks).

## Set up identity and access controls

Before you install Ratify on your AKS cluster, you need to establish the proper identity and access controls. Ratify requires access to your container registry to pull container images and signatures. When you use Key Vault for certificate management, Ratify also needs access to retrieve certificates for signature verification.

The identity configuration involves:

- Creating a user-assigned managed identity, or using an existing one.
- Setting up federated identity credentials to enable workload identity authentication.
- Granting appropriate role assignments for Container Registry access.
- Configuring Key Vault access permissions (if you're using Key Vault for certificate management).

### Create or use a user-assigned managed identity

If you don't already have a user-assigned managed identity, see [Create a user-assigned managed identity](/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities?pivots=identity-mi-methods-azcli#create-a-user-assigned-managed-identity) to create one. Ratify uses this identity to access Azure resources, such as Container Registry and (when applicable) Key Vault for certificate management.

### Create a federated identity credential for your identity

Set up environment variables by using the following code. Update the values of the variables `RATIFY_NAMESPACE` and `RATIFY_SA_NAME` if you're not using the default values. Be sure to use the same values during installation of the Ratify Helm chart.

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

The following command creates a federated credential for your managed identity. The credential allows the managed identity to authenticate by using tokens issued by an OIDC issuer, specifically for a Kubernetes service account `RATIFY_SA_NAME` in the namespace `RATIFY_NAMESPACE`.

```shell
az identity federated-credential create \
--name ratify-federated-credential \
--identity-name "$IDENTITY_NAME" \
--resource-group "$IDENTITY_RG" \
--issuer "$AKS_OIDC_ISSUER" \
--subject system:serviceaccount:"$RATIFY_NAMESPACE":"$RATIFY_SA_NAME"
```

### Configure access to Container Registry

The `AcrPull` role is required for your identity to pull signatures and other metadata for container images. Use the following code to assign the role:

```shell
export ACR_SUB=<acr-subscription-id>
export ACR_RG=<acr-resource-group>
export ACR_NAME=<acr-name>

az role assignment create \
--role acrpull \
--assignee-object-id ${IDENTITY_OBJECT_ID} \
--scope subscriptions/${ACR_SUB}/resourceGroups/${ACR_RG}/providers/Microsoft.ContainerRegistry/registries/${ACR_NAME}
```

### Configure access to Key Vault

Skip this step if you use Trusted Signing for certificate management.

The `Key Vault Secrets User` role is required for your identity to fetch the entire certificate chain from your key vault. Use the following code to assign the role:

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

With the identity and access controls properly configured, you can now install Ratify on your AKS cluster. Ratify integrates with Azure Policy to enforce signature verification policies. The installation process involves deploying Ratify by using Helm charts with specific configuration parameters that define how it should verify container image signatures.

The following sections cover two key aspects of the Ratify setup:

- Understanding the Helm chart parameters required for your certificate management approach (Key Vault or Trusted Signing)
- Installing Ratify with the appropriate configuration to enable signature verification

The configuration parameters vary depending on whether you're using Key Vault or Trusted Signing for certificate management. Be sure to follow the instructions that match your chosen scenario.

### Know your Helm chart parameters

When you're installing the Helm chart for Ratify, you need to pass values to parameters by using the `--set` flag or by providing a custom values file. Those values are used to configure Ratify for signature verification. For a comprehensive list of parameters, refer to the [Ratify Helm chart documentation](https://github.com/notaryproject/ratify/tree/v1.4.0/charts/ratify).

#### [Key Vault](#tab/akv)

You need to configure:

- The identity that you set up previously for accessing Container Registry and Key Vault.
- The certificate stored in Key Vault for signature verification.
- A Notary Project trust policy for signature verification, including `registryScopes`, `trustStores`, and `trustedIdentities`.

This table provides details about the parameters:

| Parameter                                       | Description                                                                                        | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------- |
| `azureWorkloadIdentity.clientId`                  | Client ID of the Azure workload identity                                             | `"$IDENTITY_CLIENT_ID"`               |
| `oras.authProviders.azureWorkloadIdentityEnabled` | Azure workload identity for Container Registry authentication (enable or disable)                                     | `true`                                |
| `azurekeyvault.enabled`                           | Fetching certificates from Key Vault (enable or disable)                                                     | `true`                                |
| `azurekeyvault.vaultURI`                          | URI of the Key Vault resource                                                                        | `"https://$AKV_NAME.vault.azure.net"` |
| `azurekeyvault.tenantId`                          | Tenant ID of the key vault resource                                                                  | `"$AKV_TENANT_ID"`                    |
| `azurekeyvault.certificates[0].name`              | Name of the certificate                                                                            | `"$CERT_NAME"`                        |
| `notation.trustPolicies[0].registryScopes[0]`     | Repository URI that the policy applies to                                                        | `"$REPO_URI"`                         |
| `notation.trustPolicies[0].trustStores[0]`        | Trust stores where certificates of type `ca` or `tsa` are stored                                   | `ca:azurekeyvault`                    |
| `notation.trustPolicies[0].trustedIdentities[0]`  | Subject field of the signing certificate, with prefix `x509.subject:` indicating what you trust  | `"x509.subject: $SUBJECT"`            |

By using timestamping for your images, you can ensure that images signed before the certificate expires can still be verified successfully. Add the following parameters for Time Stamping Authority (TSA) configuration:

| Parameter                                       | Description                                                                | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- |
| `notationCerts[0]`                                | File path to the PEM-formatted TSA root certificate file                | `"$TSA_ROOT_CERT_FILEPATH"`           |
| `notation.trustPolicies[0].trustStores[1]`        | Another trust store where the TSA root certificate is stored               | `tsa:notationCerts[0]`                |

If you have multiple certificates for signature verification, specify extra parameters:

| Parameter                                       | Description                                                                | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- |
| `azurekeyvault.certificates[1].name`              | Name of the certificate                                                    | `"$CERT_NAME_2"`                      |
| `notation.trustPolicies[0].trustedIdentities[1]`  | Another subject field of the signing certificate, indicating what you trust  | `"x509.subject: $SUBJECT_2"`          |

#### [Trusted Signing](#tab/trusted-signing)

You need to configure:

- The identity that you set up previously for accessing Container Registry.
- A Notary Project trust policy for signature verification, including `registryScopes`, `trustStores`, and `trustedIdentities`.
- A timestamping configuration, because Trusted Signing issues short-lived certificates.

This table provides details about the parameters:

| Parameter                                       | Description                                                                                        | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------- |
| `azureWorkloadIdentity.clientId`                  | Client ID of the Azure workload identity                                             | `"$IDENTITY_CLIENT_ID"`               |
| `oras.authProviders.azureWorkloadIdentityEnabled` | Azure workload identity for Container Registry authentication (enable or disable)                                      | `true`                                |
| `notationCerts[0]`                                | File path to the PEM-formatted Trusted Signing root certificate file                            | `"$TS_ROOT_CERT_FILEPATH"`               |
| `notationCerts[1]`                                | File path to the PEM-formatted TSA root certificate file                                        | `"$TSA_ROOT_CERT_FILEPATH"`           |
| `notation.trustPolicies[0].registryScopes[0]`     | Repository URI that the policy applies to                                                        | `"$REPO_URI"`                         |
| `notation.trustPolicies[0].trustStores[0]`        | Trust stores where the Trusted Signing root certificate is stored                                  | `ca:notationCerts`[0]                 |
| `notation.trustPolicies[0].trustStores[1]`        | Trust stores where the TSA root certificate is stored                                              | `tsa:notationCerts[1]`                |
| `notation.trustPolicies[0].trustedIdentities[0]`  | Subject field of the Trusted Signing certificate, with prefix `x509.subject:` indicating what you trust  | `"x509.subject: $SUBJECT"`            |

If you have multiple Trusted Signing certificate profiles, you can add additional trusted identities:

| Parameter                                       | Description                                                                | Value                               |
| ----------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- |
| `notation.trustPolicies[0].trustedIdentities[1]`  | Another subject field of the certificate profile, indicating what you trust  | `"x509.subject: $SUBJECT_2"`          |

---

### Install a Ratify Helm chart with desired parameters and values

Ensure that the Ratify Helm chart version is at least `1.15.0`, which will install Ratify version `1.4.0` or later. The following example uses Helm chart version `1.15.0`.

Set up additional environment variables for installation:

#### [Key Vault](#tab/akv)

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

By default, the Trusted Signing root certificate and TSA root certificate are provided in `.crt` format. Before passing them to the Helm installation, you must convert them to **PEM** format.

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
> For images that are not linked to a trust policy, signature verification fails. For instance, if the images are not within the repository `$REPO_URI`, the signature verification for those images fails. You can add multiple repositories by specifying additional parameters. For example, to add another repository for the trust policy `notation.trustPolicies[0]`, include the parameter `--set notation.trustPolicies[0].registryScopes[1]="$REPO_URI_1"`.

## Set up a custom Azure policy

With Ratify successfully installed and configured on your AKS cluster, the final step is to create and assign an Azure Policy that will enforce signature verification during container deployments. This policy acts as the enforcement mechanism that instructs the cluster to use Ratify for verifying container image signatures before allowing deployments.

Azure Policy offers two enforcement modes:

- `Deny`: Blocks deployment of images that fail signature verification, ensuring only trusted images run in your cluster
- `Audit`: Allows all deployments but marks compliant resources for monitoring and reporting purposes

The `Audit` effect is useful during initial setup or testing phases, allowing you to validate your configuration without risking service disruptions in production environments.

### Assign a new policy to your AKS cluster

Create a custom Azure policy for signature verification. By default, the policy effect is set to `Deny`, meaning images that fail signature verification is denied deployment. Alternatively, you can configure the policy effect to `Audit`, allowing images that fail signature verification to be deployed while marking the AKS cluster and related workloads as compliant. The `Audit` effect is useful for verifying your signature verification configuration without risking outages due to incorrect settings for your production environment.

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

To change the policy effect to `Audit`, you can pass another parameter to `az policy assignment create` command. For example:

```shell
az policy assignment create --policy "$DEFINITION_ID" --name "$DEFINITION_NAME" --scope "$POLICY_SCOPE" -p "{\"effect\": {\"value\":\"Audit\"}}"
```

>[!NOTE]
> It takes around 15 minutes to complete the assignment.

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

Now that you have successfully configured Ratify and assigned the Azure Policy to your AKS cluster, it's time to test the signature verification functionality. This section demonstrates how the policy enforcement works in practice by deploying different types of container images and observing the results.

You test three scenarios to validate your setup:

- **Signed images with trusted certificates**: Should deploy successfully
- **Unsigned images**: Should be blocked (with `Deny` effect) or marked compliant (with `Audit` effect)
- **Images signed with untrusted certificates**: Should be blocked (with `Deny` effect) or marked compliant (with `Audit` effect)

The behavior you observe will depend on the policy effect you chose during the Azure Policy assignment step. This testing process helps ensure your signature verification is working correctly and provides confidence that only trusted images will be allowed in your production environment.

### Use Deny policy effect

With the `Deny` policy effect, only images signed with trusted identities are allowed for deployment. You can begin deploying your workloads to observe the effects. In this document, we use the `kubectl` command to deploy a pod. Similarly, you can deploy your workloads using a Helm chart or any templates that trigger Helm installation.

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

`$IMAGE_UNSIGNED` references an image that is not signed. `$IMAGE_SIGNED_UNTRUSTED` references an image that is signed using a different certificate that you won't trust. So, these two images are denied for deployment. For example, run the following command:

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

With Audit policy effect, unsigned images or images signed with untrusted identities are allowed for deployment. However, the AKS cluster and related components are marked as `noncompliant`. For more details on how to view noncompliant resources and understand the reasons, see [Get the Azure policy compliance-data](/azure/governance/policy/how-to/get-compliance-data).

## Cleaning up

Use the following commands to uninstall Ratify and clean-up Ratify CRDs:

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

### How can I set up certificates for signature verification if I don't have access to Key Vault?

In some cases, image consumers may not have access to the certificates used for signature verification. To verify signatures, you'll need to download the root CA certificate file in PEM format and specify the related parameters for the Ratify Helm chart installation. Below is an example command similar to the previous installation command, but without any parameters related to Key Vault certificates. The Notary Project trust store refers to the certificate file that passed in parameter `notationCerts[0]`:

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
> Since `notationCerts[0]` is used for the root CA certificate, if you have an extra certificate file for timestamping purpose, make sure you use the correct index. For example,
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

- To update Key Vault related configurations, use the Ratify `KeyManagementProvider` custom resource with the type `azurekeyvault`. To update Trusted Signing related configurations, use the Ratify `KeyManagementProvider` custom resource with the type `inline`. Follow the [documentation](https://ratify.dev/docs/reference/custom%20resources/key-management-providers).
- To update Notary Project trust policies and stores, use the Ratify `Verifier` custom resource. Follow the [documentation](https://ratify.dev/docs/reference/custom%20resources/verifiers).
- To authenticate and interact with Container Registry (or other OCI-compliant registries), use the Ratify Store custom resource. Follow the [documentation](https://ratify.dev/docs/reference/custom%20resources/stores).

### What should I do if my container images aren't signed using the Notation tool?

This document is applicable for verifying Notary Project signatures independently on any tools that can produce Notary Project-compliant signatures. Ratify also supports verifying other types of signatures. For more information, see the [Ratify user guide](https://ratify.dev/docs/1.2/quickstarts/ratify-on-azure#use-cosign-with-keys-stored-in-akv).
