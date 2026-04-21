---
title: "Deploy cert-manager for Arc-enabled Kubernetes (preview)"
ms.date: 4/13/2026
ms.topic: how-to
description: "Learn how to deploy the cert-manager for Azure Arc-enabled Kubernetes (preview) extension or upgrade from open source cert-manager and trust-manager."
# Customer intent: As a customer using Azure Arc-enabled Kubernetes, I want to understand how to deploy and configure the cert-manager for Arc-enabled Kubernetes extension, so that I can ensure secure communication and compliance across my hybrid Kubernetes environments.
---

# Deploy cert-manager for Arc-enabled Kubernetes (preview)

This article shows how to deploy the cert-manager for Arc-enabled Kubernetes (preview) extension in your Arc-connected clusters. Steps for a new deployment are included, along with requirements for migrating from a cluster that has open source cert-manager and trust-manager.

> [!IMPORTANT]
> Cert-manager for Azure Arc-enabled Kubernetes clusters is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites

Before installing CME, ensure you meet the following prerequisites:

- An Azure account. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- An Arc-enabled Kubernetes cluster deployed in a [supported region](cert-manager-overview.md#regional-support). For best results, use a [Kubernetes distribution that has been validated for use with cert-manager for Arc-enabled Kubernetes](cert-manager-overview.md#validated-arc-enabled-kubernetes-distributions). If you haven't already connected your cluster to Azure Arc, [follow our quickstart](quickstart-connect-cluster.md). Take note of the cluster name and Azure resource group, as you'll need these to install the extension.
- The **Azure Connected Machine Resource Administrator** or an equivalent role on the cluster resource that allows you to deploy extensions.
- The latest version of Azure CLI.
- The latest version of the `connectedk8s` and `k8s-extension` Azure CLI extensions. Run these commands to install or upgrade the extensions:

  ```azurecli
  az extension add --upgrade -n connectedk8s
  az extension add --upgrade -n k8s-extension
  ```

- An installed version of kubectl for your cluster (admin privileges). While not strictly required to install the extension, kubectl is helpful to verify installation and manage certificate custom resources in later steps.
- If you previously installed open source [cert-manager](https://cert-manager.io/) or [trust-manager](https://cert-manager.io/docs/trust/trust-manager/) manually, uninstall them before deploying the Arc extension to avoid conflicts. Existing custom resources (CRs) will be retained and recognized by the extension, as long as they adhere to the conventions of the open-source projects. For more information, see [Migrate from open source cert-manager and trust-manager](#migrate-from-open-source-cert-manager-and-trust-manager).

General knowledge about [Arc-enabled Kubernetes](../overview.md), [cert-manager](https://cert-manager.io/), and [trust-manager](https://cert-manager.io/docs/trust/trust-manager/) can be helpful, but isn't required to deploy the extension.

> [!IMPORTANT]
> To protect sensitive data, encrypt your Kubernetes secret store in all environments. For Azure Local, encryption of the secret store is enabled by default. Follow the guidance to enable and manage secret encryption for [AKS Edge Essentials](/azure/aks/aksarc/aks-edge-howto-secret-encryption), [AKS enabled by Azure Arc](/azure/aks/aksarc/encrypt-etcd-secrets), and third-party Kubernetes distributions to ensure your cluster meets enterprise and compliance requirements for data protection.

## Deploy cert-manager for Arc-enabled Kubernetes

To deploy cert-manager for Arc-enabled Kubernetes (preview) to a cluster that doesn't already have cert-manager and trust-manager, and that meets the prerequisites, use the following command:

```azurecli
az k8s-extension create \
  --resource-group ${RESOURCE_GROUP} \
  --cluster-name ${CLUSTER_NAME} \
  --cluster-type connectedClusters \
  --name "azure-cert-management" \
  --extension-type "microsoft.certmanagement" \
```

You can add `--auto-upgrade-minor-version true` to automatically receive minor version updates, which include new features and non-breaking changes. Otherwise, you can [manually update the extension](extensions.md#upgrade-an-extension-instance) when needed to get the latest features and fixes.

After the Azure CLI confirms that the installation was successful, [verify that the components are running in your cluster](cert-manager-monitor-troubleshoot.md#confirm-that-pods-and-components-are-running).

### Restricted PSA environments

Cert-manager for Azure Arc-enabled Kubernetes requires privileged access for log collection into Azure for support and troubleshooting purposes. For environments with restricted Pod Security Standards (PSA) where privileged access isn't permitted, log collection into Azure can be disabled to comply with security requirements.

To disable log collection, use `--set global.telemetry.logs.enabled=false`. This prevents log volumes from being set up and removes the need for privileged access, while metrics collection continues as normal.

To disable metrics collection, use `--set global.telemetry.metrics.enabled=false`.

## Migrate from open source cert-manager and trust-manager

The cert-manager for Azure Arc-enabled Kubernetes extension serves as a replacement for both open source [cert-manager](https://cert-manager.io/) and [trust-manager](https://cert-manager.io/docs/trust/trust-manager/). Before you install cert-manager for Arc-enabled Kubernetes, you must uninstall the open source resources.

> [!WARNING]
> During the time between uninstalling the OS version and installing the Arc extension, certificate rotation doesn't occur and trust bundles aren't distributed to new namespaces. To minimize potential security risks, ensure that this transition period is as short as possible.
> 
> Uninstalling the open source cert-manager and trust-manager doesn't remove any existing certificates or related resources you created. These resources remain accessible and usable after the cert-manager for Arc-enabled Kubernetes extension is installed.

Specific steps for uninstallation depend on your installation method. For instructions, see the documentation for [uninstalling cert-manager](https://cert-manager.io/docs/installation/uninstall/) and [uninstalling trust-manager](https://cert-manager.io/docs/trust/trust-manager/installation/#uninstalling). If you used Helm for installation, use this command to confirm which namespaces cert-manager and trust-manager are installed in:

```bash
helm list -A | grep -E 'trust-manager|cert-manager'
```

After that, use the following commands to uninstall:

```bash
helm uninstall cert-manager -n "your namespace" --ignore-not-found
helm uninstall trust-manager -n "your namespace" --ignore-not-found
```

## Configure cert-manager for Arc-enabled Kubernetes

After the cert-manager for Arc-enabled Kubernetes extension is deployed, configure how certificates should be issued, then request a certificate for a workload. At a high level, the steps are:

1. Create an Issuer or ClusterIssuer resource to specify a Certificate Authority (CA) that will generate signed certificates. This could be a self-signed certificate authority (CA), an account registered with an Automated Certificate Management Environment (ACME) Certificate Authority server such as Let's Encrypt (or others), or a Certificate Authority whose certificate and private key are stored inside the cluster as a Kubernetes Secret.
1. Create a Certificate resource that requests a TLS certificate request and includes the fields that are used to generate Certificate Signing Requests (CSRs), which are then fulfilled by the issuer type referenced on the resource for a specific domain or use.
1. Let cert-manager fulfill the request, which results in a signed certificate and private key being stored in a Kubernetes secret that your application can use.
1. Optionally, if your scenario requires custom trust roots across namespaces, use trust-manager to distribute the custom CA certificates cluster-wide.

The following sections walk through an example scenario to issue a self-signed certificate for an in-cluster service.

> [!IMPORTANT]
> The following example is for demonstration purposes only. In production scenarios, use a secure CA such as an ACME provider or your enterprise PKI, rather than a self-signed issuer.

### Create a self-signed ClusterIssuer

> [!IMPORTANT]
> Self-signed certificates aren't recommended for production use. For production environments with ingress and egress scenarios, use an existing organization CA that you control, or use type ACME or Vault depending on your needs. This example is intended only for evaluation purposes.

#### Create a ClusterIssuer (self-signed CA)

First, set up a ClusterIssuer that uses a self-signed certificate as its CA. This ClusterIssuer will act as an internal certificate authority for the cluster, signing certificates for your workloads. The CA should be backed by RSA-generated private keys with a minimum length of 4096.

This example uses a self-signed certificate to demonstrate the process, which is suitable for inter-cluster communication or testing, but is not recommended for production workloads.

Apply the following YAML manifest to create a self-signed ClusterIssuer:

```yml

cat <<EOF | kubectl apply -f -
  apiVersion: cert-manager.io/v1
  kind: ClusterIssuer
  metadata:
    name: selfsigned-cluster-ca
  spec:
    selfSigned: {}
EOF
```

This defines a ClusterIssuer named `selfsigned-cluster-ca` that issues self-signed certificates (meaning it effectively generates a root CA certificate for signing). After you define the ClusterIssuer, cert-manager automatically generates a root key and certificate for this issuer behind the scenes. It stores the generated CA certificate and key in a secret (by default named `selfsigned-cluster-ca-cert` in the same namespace as cert-manager, typically called `cert-manager`)

#### Create a certificate signed by the ClusterIssuer

Now request a certificate for a specific purpose (for example, workload authentication). Create a certificate resource that asks for a certificate for an internal name such as `domain my-service.mydomainlocal`, signed by the new self-signed cluster CA.

```yaml
cat <<EOF | kubectl apply -f -
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: my-service-cert
    namespace: default  # or the namespace where your application resides
  spec:
    secretName: my-service-tls 
    duration: 2160h            # 90 days (default) 
    renewBefore: 360h          # renew 15 days before expiration
    commonName: my-service.mydomain.local
    dnsNames:
    - my-service.mydomain.local
    issuerRef:
      name: selfsigned-cluster-ca     
      kind: ClusterIssuer
    privateKey:
      rotationPolicy: Always       
EOF
```

This spec includes the following fields:

- `secretName: my-service-tls` is where the resulting TLS certificate and key will be stored (as a secret in the same namespace). Applications will use this secret.
- `commonName` and `dnsNames` specify the subject name(s) for the certificate. This example uses a fictitious service DNS name.
- `issuerRef` points to the issuer that you created. Set `kind: ClusterIssuer` and `name` to reference your self-signed CA ClusterIssuer.

Within a few seconds, cert-manager generates a private key and create a `CertificateSigningRequest` to the referenced issuer. Because this issuer is self-signed, cert-manager itself will act as the signer by using the CA key it generated earlier.

#### Distribute trust bundle (optional)

If you only use self-signed certificates within your cluster, you might want all workloads to trust the cluster's self-signed root CA across namespaces. Trust Manager can distribute the CA as a ConfigMap where needed.

Create a `Certificate` resource to store the self-signed CA in the secret selfsigned-cluster-ca-cert:

```yaml
cat <<EOF | kubectl apply -f -
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: selfsigned-cluster-ca-cert
    namespace: cert-manager
  spec:
    isCA: true
    commonName: selfsigned-cluster-ca-cert
    secretName: selfsigned-cluster-ca-cert
    privateKey:
      algorithm: ECDSA
      size: 256
      rotationPolicy: Always      
    issuerRef:
      name: selfsigned-cluster-ca
      kind: ClusterIssuer
      group: cert-manager.io
EOF
```

Create a `Bundle` resource to configure trust-manager to distribute the CA certificate:

```yaml
cat <<EOF | kubectl apply -f -
  apiVersion: trust.cert-manager.io/v1alpha1
  kind: Bundle
  metadata:
    name: cluster-ca-bundle
    namespace: cert-manager
  spec:
    sources:
    - secret:
        name: selfsigned-cluster-ca-cert
        key: tls.crt
    target:
      configMap:
        key: ca.crt
EOF
```

This configuration instructs trust-manager to take the certificate data from the specified secret (`selfsigned-cluster-ca-cert`) and publish it into a ConfigMap called `cluster-ca-bundle` in all namespaces. The ConfigMap has a data entry `ca.crt` containing the CA certificate. By applying this Bundle, trust-manager will create and update the ConfigMaps accordingly.

Any pods can then mount the `cluster-ca-bundle` ConfigMap to trust the custom CA, or the ConfigMap can be referenced by other extensions. Using a trust bundle is especially useful if using a corporate CA; you would similarly distribute that CA to all pods needing to trust it.

## Remove cert-manager for Arc-enabled Kubernetes (preview)

To uninstall the cert-manager for Arc-enabled Kubernetes (preview) extension, use the following command:

```azurecli
az k8s-extension delete \
  --resource-group ${RESOURCE_GROUP} \
  --cluster-name ${CLUSTER_NAME} \
  --cluster-type connectedClusters \
  --name "azure-cert-management"
```

This command removes the cert-manager and trust-manager deployments. It doesn't remove secrets and configmaps for trust bundles, but it does remove the controllers that keep them updated. Any certificates or issuers that you created will remain, and any users of those certificates can continue to use them until they expire. To fully remove cert-manager from the cluster, remove the certificate and issuer resources that you created, and any secrets that were generated.
