---
title: "Tutorial: Deploy applications using GitOps"
description: "This tutorial shows how to use GitOps with ArgoCD in Azure Arc and AKS clusters."
ms.date: 02/02/2026
ms.topic: tutorial
ms.custom:
  - template-tutorial
  - devx-track-azurecli
  - references_regions
  - build-2025
# Customer intent: As a DevOps engineer, I want to deploy applications using GitOps with ArgoCD on Azure Arc or AKS clusters, so that I can manage configurations and automate application deployments efficiently.
---

# Tutorial: Deploy applications using GitOps with ArgoCD

This tutorial describes how to use GitOps in a Kubernetes cluster. GitOps with ArgoCD is enabled as a [cluster extension](conceptual-extensions.md) in Azure Arc-enabled Kubernetes clusters or Azure Kubernetes Service (AKS) clusters. With GitOps, you can use your Git repository as the source of truth for cluster configuration and application deployment. ArgoCD also supports other common file sources, such as Helm and Open Container Initiative (OCI) repositories.

> [!NOTE]
> Starting with version 1.0.0-preview, the ArgoCD extension uses the [community Helm chart](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd). **This change is a breaking change as the configuration keys have changed**. If you installed a previous version (0.0.x) of the extension, uninstall the extension and reinstall the latest with updated configuration keys.

> [!IMPORTANT]
> GitOps with ArgoCD is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.
> For production GitOps extension support, try the [GitOps extension using Flux](tutorial-use-gitops-flux2.md).

## Prerequisites

To deploy applications using GitOps, you need either an Azure Arc-enabled Kubernetes cluster or an AKS cluster.

#### Azure Arc-enabled Kubernetes clusters

* An Azure Arc-enabled Kubernetes connected cluster that's up and running.

  [Learn how to connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md). If you need to connect through an outbound proxy, then assure you [install the Arc agents with proxy settings](./quickstart-connect-cluster.md?tabs=azure-cli#connect-using-an-outbound-proxy-server).

* Read and write permissions on the `Microsoft.Kubernetes/connectedClusters` resource type.

#### Azure Kubernetes Service clusters

* An MSI-based AKS cluster that's up and running.

  > [!IMPORTANT]
  > The AKS cluster needs to be created with Managed Service Identity (MSI), not Service Principal Name (SPN), for this extension to work.
  > For new AKS clusters created with `az aks create`, the cluster is MSI-based by default. To convert SPN-based clusters to MSI, run `az aks update -g $RESOURCE_GROUP -n $CLUSTER_NAME --enable-managed-identity`. For more information, see [Use a managed identity in AKS](/azure/aks/use-managed-identity).

* Read and write permissions on the `Microsoft.ContainerService/managedClusters` resource type.

#### Common to both cluster types

* Read and write permissions on these resource types:

  * `Microsoft.KubernetesConfiguration/extensions`

* Azure CLI version 2.15 or later. [Install the Azure CLI](/cli/azure/install-azure-cli) or use the following commands to update to the latest version:

  ```azurecli
  az version
  az upgrade
  ```

* The Kubernetes command-line client, [kubectl](https://kubernetes.io/docs/reference/kubectl/). `kubectl` is already installed if you use Azure Cloud Shell.

  Install `kubectl` locally using the [`az aks install-cli`](/cli/azure/aks#az-aks-install-cli) command:

  ```azurecli
  az aks install-cli
  ```

* Registration of the following Azure resource providers:

  ```azurecli
  az provider register --namespace Microsoft.Kubernetes
  az provider register --namespace Microsoft.ContainerService
  az provider register --namespace Microsoft.KubernetesConfiguration
  ```

  Registration is an asynchronous process and should finish within 10 minutes. To monitor the registration process, use the following command:

  ```azurecli
  az provider show -n Microsoft.KubernetesConfiguration -o table

  Namespace                          RegistrationPolicy    RegistrationState
  ---------------------------------  --------------------  -------------------
  Microsoft.KubernetesConfiguration  RegistrationRequired  Registered
  ```

> [!TIP]
> While the source in this tutorial is a Git repository, ArgoCD supports other common file sources such as Helm and Open Container Initiative (OCI) repositories.

#### Version and region support

GitOps is currently supported in [all regions that Azure Arc-enabled Kubernetes supports](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=kubernetes-service,azure-arc). GitOps is currently supported in a subset of the regions that AKS supports. The GitOps service is adding new supported regions on a regular cadence.

#### Network requirements

The GitOps agents require outbound (egress) TCP to the repo source on either port 22 (SSH) or port 443 (HTTPS) to function. The agents also require access to the following outbound URLs:

| Endpoint (DNS) | Description |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `https://management.azure.com` | Required for the agent to communicate with the Kubernetes Configuration service. |
| `https://<region>.dp.kubernetesconfiguration.azure.com` | Data plane endpoint for the agent to push status and fetch configuration information. Depends on `<region>` (the supported regions mentioned earlier). |
| `https://login.microsoftonline.com` | Required to fetch and update Azure Resource Manager tokens. |
| `https://mcr.microsoft.com` | Required to pull container images for controllers. |

### Enable CLI extensions

Install the latest `k8s-configuration` and `k8s-extension` CLI extension packages:

```azurecli
az extension add -n k8s-configuration
az extension add -n k8s-extension
```

To update these packages to the latest versions:

```azurecli
az extension update -n k8s-configuration
az extension update -n k8s-extension
```

To see a list of all installed Azure CLI extensions and their versions, use the following command:

```azurecli
az extension list -o table

Experimental   ExtensionType   Name                   Path                                                       Preview   Version
-------------  --------------  -----------------      -----------------------------------------------------      --------  --------
False          whl             connectedk8s           C:\Users\somename\.azure\cliextensions\connectedk8s         False     1.10.7
False          whl             k8s-configuration      C:\Users\somename\.azure\cliextensions\k8s-configuration    False     2.2.0
False          whl             k8s-extension          C:\Users\somename\.azure\cliextensions\k8s-extension        False     1.6.4
```

## Create GitOps (ArgoCD) extension (simple installation)

The GitOps [ArgoCD installation](https://argo-cd.readthedocs.io/en/stable/operator-manual/installation/) supports multi-tenancy in high availability (HA) mode and supports workload identity.

> [!IMPORTANT]
> The HA mode is the default configuration and requires three nodes in the cluster to be able to install. The command below adds `--config "redis-ha\.enabled=false` to install the extension on a single node.

 This command creates the simplest configuration installing the ArgoCD components to a new `argocd` namespace with cluster-wide access. Cluster-wide access enables ArgoCD app definitions to be detected in any namespace listed in the ArgoCD configmap configuration in the cluster. For example: `namespace1,namespace2`

```azurecli
az k8s-extension create --resource-group <resource-group> --cluster-name <cluster-name> \
--cluster-type managedClusters \
--name argocd \
--extension-type Microsoft.ArgoCD \
--release-train preview \
--config "redis-ha\.enabled=false" \
--config "config-maps.argocd-cmd-params-cm.data.application\.namespaces=namespace1,namespace2"
```

This installation command creates a new `<namespace>` namespace and installs the ArgoCD components in the `<namespace>`.  ArgoCD application definitions in this configuration only function in the `<namespace>` namespace.

> [!NOTE]
> For addition configuration options, such as resource limits, see [values.yaml](https://github.com/argoproj/argo-helm/blob/main/charts/argo-cd/values.yaml). Use these configurations in your Azure CLI command when configuring the extension.

## Create GitOps (ArgoCD) extension with workload identity

An alternative installation method recommended for production usage is [workload identity](/azure/aks/workload-identity-deploy-cluster). This method allows you to use Microsoft Entra ID identities to authenticate to Azure resources without needing to manage secrets or credentials in your Git repository. This installation utilizes workload identity authentication enabled in the 3.0.0-rc2 or later OSS version of ArgoCD.

> [!IMPORTANT]
> The HA mode is the default configuration and requires three nodes in the cluster to be able to install. Use `'redis-ha\\.enabled': false` to install the extension on a single node.

To create the extension with workload identity, first replace the following variables with your own values in this Bicep template:

```bicep
var clusterName = '<aks-or-arc-cluster-name>'

var workloadIdentityClientId = 'replace-me##-##-###-###'
var ssoWorkloadIdentityClientId = 'replace-me##-##-###-###'

var url = 'https://<public-ip-for-argocd-ui>/'
var oidcConfig = '''
name: Azure
issuer: https://login.microsoftonline.com/<your-tenant-id>/v2.0
clientID: <same-value-as-ssoWorkloadIdentityClientId-above>
azure:
  useWorkloadIdentity: true
requestedIDTokenClaims:
  groups:
    essential: true
requestedScopes:
  - openid
  - profile
  - email
'''

var defaultPolicy = 'role:readonly'
var policy = '''
p, role:org-admin, applications, *, */*, allow
p, role:org-admin, clusters, get, *, allow
p, role:org-admin, repositories, get, *, allow
p, role:org-admin, repositories, create, *, allow
p, role:org-admin, repositories, update, *, allow
p, role:org-admin, repositories, delete, *, allow
g, replace-me##-argocd-ui-Microsoft Entra-group-admin-id, role:org-admin
'''

resource cluster 'Microsoft.ContainerService/managedClusters@2024-10-01' existing = {
  name: clusterName
}

resource extension 'Microsoft.KubernetesConfiguration/extensions@2023-05-01' = {
  name: 'argocd'
  scope: cluster
  properties: {
    extensionType: 'Microsoft.ArgoCD'
    releaseTrain: 'preview'
    configurationSettings: {
      'redis-ha.enabled': 'true'
      'azure.workloadIdentity.enabled': 'true'
      'workloadIdentity.clientId': workloadIdentityClientId
      'workloadIdentity.entraSSOClientId': ssoWorkloadIdentityClientId
      'config-maps.argocd-cm.data.oidc\\.config': oidcConfig
      'config-maps.argocd-cm.data.url': url
      'config-maps.argocd-rbac-cm.data.policy\\.default': defaultPolicy
      'config-maps.argocd-rbac-cm.data.policy\\.csv': policy
      'config-maps.argocd-cmd-params-cm.data.application\\.namespaces': 'default, argocd'
      'azure.workloadIdentity.clientId': workloadIdentityClientId
      'azure.workloadIdentity.entraSSOClientId': ssoWorkloadIdentityClientId
      'configs.cm.oidc\\.config': oidcConfig
      'configs.cm.url': url
      'configs.rbac.policy\\.default': defaultPolicy
      'configs.rbac.policy\\.csv': policy
      'configs.params.application\\.namespaces': 'default, argocd'
   }
  }
}
```

The Bicep template can be created using this command:

`az deployment group create --resource-group <resource-group> --template-file <bicep-file>`

> [!NOTE]
> For addition configuration options, such as resource limits, see [values.yaml](https://github.com/argoproj/argo-helm/blob/main/charts/argo-cd/values.yaml). Use these configurations in the Bicep template when configuring the extension.

### Parameters

`clusterName` is the name of the AKS or Arc-enabled Kubernetes cluster.

`workloadIdentityClientId` and `ssoWorkloadIdentityClientId` are the client IDs of the managed identity desired to be used for workload identity. The `ssoWorkloadIdentityClientId` is used for the authentication for the ArgoCD UI and the `workloadIdentityClientId` is used for the workload identity for the ArgoCD components. Visit [Microsoft Entra ID App Registration Auth using OIDC](https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/user-management/microsoft.md) for additional information on general setup and configuration of the ssoWorkloadIdentityClientId.

`url` is the public IP of the ArgoCD UI. There's no public IP or domain name unless the cluster already has a customer provided ingress controller. If so, the ingress rule needs to be added to the ArgoCD UI after deployment.

`oidcConfig` - replace `<your-tenant-id>` with the tenant ID of your Microsoft Entra ID. Replace `<same-value-as-ssoWorkloadIdentityClientId-above>` with the same value as `ssoWorkloadIdentityClientId`.

`policy` variable is the `argocd-rbac-cm configmap` settings of ArgoCD. `g, replace-me##-argocd-ui-entra-group-admin-id` is the Microsoft Entra group ID that gives admin access to the ArgoCD UI. The Microsoft Entra group ID can be found in the Azure portal under **Microsoft Entra ID > Groups > _your-group-name_ > Properties**. You can use the Microsoft Entra user ID instead of a Microsoft Entra group ID. The Microsoft Entra user ID can be found in the Azure portal under **Microsoft Entra ID > Users > _your-user-name_ > Properties.**

### Create workload identity credentials

To set up new workload identity credentials, follow these steps:

1. Retrieve the OIDC issuer URL for your [AKS cluster](/azure/aks/workload-identity-deploy-cluster#retrieve-the-oidc-issuer-url) or [Arc-enabled Kubernetes cluster](workload-identity.md#retrieve-the-oidc-issuer-url).
1. Create a [managed identity](/azure/aks/workload-identity-deploy-cluster#create-a-managed-identity) and note its client ID and tenant ID.
1. Establish a federated identity credential for your [AKS cluster](/azure/aks/workload-identity-deploy-cluster#establish-federated-identity-credential) or [Arc-enabled Kubernetes cluster](workload-identity.md#create-the-federated-identity-credential). For example:

   ```azurecli
   # For source-controller
   az identity federated-credential create --name ${FEDERATED_IDENTITY_CREDENTIAL_NAME} --identity-name "${USER_ASSIGNED_IDENTITY_NAME}" --resource-group "${RESOURCE_GROUP}" --issuer "${OIDC_ISSUER}" --subject system:serviceaccount:"argocd":"source-controller" --audience api://AzureADTokenExchange
   ```

1. Be sure to provide proper permissions for workload identity for the resource that you want source-controller or image-reflector controller to pull. For example, if using Azure Container Registry, ensure either `Container Registry Repository Reader` (for [ABAC-enabled registries](../../container-registry/container-registry-rbac-abac-repository-permissions.md)) or `AcrPull` (for non-ABAC registries) has been applied.

## Connect to private ACR registries or ACR repositories using workload identity

To utilize the private ACR registry or ACR repositories, follow the instructions in the official ArgoCD documentation for [connecting to private ACR registries](https://github.com/argoproj/argo-cd/blob/master/docs/user-guide/private-repositories.md#azure-container-registryazure-repos-using-azure-workload-identity). The **Label the Pods**, **Create Federated Identity Credential**, and **Add annotation to Service Account** steps in that guide were completed by the extension with the Bicep deployment and can be skipped.

## Access the ArgoCD UI

If there's no existing ingress controller for the AKS cluster, then the ArgoCD UI can be exposed directly using a LoadBalancer service. The following command will expose the ArgoCD UI on port 80 and 443.

```bash
kubectl -n argocd expose service argocd-server --type LoadBalancer --name argocd-server-lb --port 80 --target-port 8080
```

## Deploy ArgoCD application

Now that the ArgoCD extension is installed, you can deploy an application using the ArgoCD UI or CLI. The following example simply uses `kubectl apply` to deploy AKS store inside an ArgoCD application to the default ArgoCD project in the `argocd` namespace.

```bash
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: aks-store-demo
  namespace: argocd
spec:
  project: default
  source:    
      repoURL: https://github.com/Azure-Samples/aks-store-demo.git
      targetRevision: HEAD
      path: kustomize/overlays/dev
  syncPolicy:
      automated: {}
  destination:
      namespace: argocd
      server: https://kubernetes.default.svc
EOF
```

The AKS store demo application was installed into the `pets` namespace. See the application webpage by [following these instructions](/azure/aks/learn/quick-kubernetes-deploy-cli#test-the-application). Be sure to visit the IP address using http and not https.

## Update extension configuration

ArgoCD configmaps can be updated after installation and other extension configuration settings using the following command:

```azurecli
az k8s-extension update --resource-group <resource-group> --cluster-name <cluster-name> --cluster-type <cluster-type> --name Microsoft.ArgoCD –-config "configs.cm.url='https://<public-ip-for-argocd-ui>/auth/callback'"
```

Update the ArgoCD configmap through the extension, so the settings don't get overwritten.  [Applying the Bicep template](#create-gitops-argocd-extension-with-workload-identity) is an alternate method to using Azure CLI to update the configuration.

## Delete the extension

Use the following commands to delete the extension.

```azurecli
az k8s-extension delete -g <resource-group> -c <cluster-name> -n argocd -t managedClusters --yes
```

---

## Next steps

* File issues and feature requests on the [Azure/AKS repository](https://github.com/Azure/AKS/labels/extension%2Fargocd) and be sure to include the word "ArgoCD" in the description or title.
* Explore [AKS-Platform engineering code sample](https://github.com/Azure-Samples/aks-platform-engineering) which deploys OSS ArgoCD with Backstage and Cluster API Provider for Azure (CAPZ) or Crossplane.
