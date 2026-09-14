---
title: "GitOps (Flux v2) supported parameters"
description: "Understand the supported parameters for GitOps (Flux v2) in Azure for use in Azure Arc-enabled Kubernetes and Azure Kubernetes Service (AKS) clusters."
ms.date: 09/14/2026
ms.topic: concept-article
ms.custom: sfi-ropc-nochange
# Customer intent: As a Kubernetes administrator, I want to understand the supported parameters for Flux v2 in Azure so that I can effectively configure GitOps for my Azure Kubernetes Service and Azure Arc-enabled Kubernetes clusters.
---

# GitOps (Flux v2) supported parameters

Azure provides an automated application deployments capability using GitOps that works with Azure Kubernetes Service (AKS) and Azure Arc-enabled Kubernetes clusters. GitOps with Flux v2 lets you use your Git repository as the source of truth for cluster configuration and application deployment. For more information, see [Application deployments with GitOps (Flux v2)](conceptual-gitops-flux2.md) and [Tutorial: Deploy applications using GitOps with Flux v2](tutorial-use-gitops-flux2.md).

GitOps on Azure Arc-enabled Kubernetes or Azure Kubernetes Service uses [Flux](https://fluxcd.io/docs/), a popular open-source tool set that supports many parameters to enable various scenarios. For a description of all parameters that Flux supports, see the [official Flux documentation](https://fluxcd.io/docs/).

To see all the parameters supported by Flux in Azure, see the [`az k8s-configuration flux` documentation](/cli/azure/k8s-configuration/flux). This implementation doesn't currently support every parameter that Flux supports. Let us know if a parameter you need is missing from the Azure implementation.

This article describes some of the parameters and arguments available for the `az k8s-configuration flux create` command. You can also see the full list of parameters for the `az k8s-configuration flux` by using the `-h` parameter in Azure CLI (for example, `az k8s-configuration flux -h` or `az k8s-configuration flux create -h`).

> [!TIP]
> A workaround to deploy Flux resources with unsupported parameters is to define the required Flux custom resources (such as [GitRepository](https://fluxcd.io/flux/components/source/gitrepositories/) or [Kustomization](https://fluxcd.io/flux/components/kustomize/kustomization/)) inside your Git repository. Deploy these resources with the `az k8s-configuration flux create` command. You can still access your Flux resources through the Azure Arc UI.

## Configuration general arguments

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--cluster-name` `-c` | String | Name of the cluster resource in Azure. |
| `--cluster-type` `-t` | Allowed values: `connectedClusters`, `managedClusters`, `provisionedClusters` | Use `connectedClusters` for Azure Arc-enabled Kubernetes clusters or `managedClusters` for AKS clusters. |
| `--resource-group` `-g` | String | Name of the Azure resource group that holds the cluster resource. |
| `--name` `-n`| String | Name of the Flux configuration in Azure. |
| `--namespace` `--ns` | String | Name of the namespace to deploy the configuration.  Default: `default`. |
| `--scope` `-s` | String | Permission scope for the operators. Possible values are `cluster` (full access) or `namespace` (restricted access). Default: `cluster`. |
| `--suspend` | flag | Suspends all source and kustomize reconciliations defined in this Flux configuration. Reconciliations active at the time of suspension continue.  |

## Source general arguments

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--kind` | String | Source kind to reconcile. Allowed values: `bucket`, `git`, `azblob`, `oci`. Default: `git`. |
| `--timeout` | [golang duration format](https://pkg.go.dev/time#Duration.String) | Maximum time to attempt to reconcile the source before timing out. Default: `10m`. |
| `--sync-interval` `--interval` | [golang duration format](https://pkg.go.dev/time#Duration.String) | Time between reconciliations of the source on the cluster. Default: `10m`. |

## Git repository source reference arguments

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--branch` | String | Branch within the Git source to sync to the cluster. Default: `master`. Newer repositories might have a root branch named `main`, in which case you need to set `--branch=main`.  |
| `--tag` | String | Tag within the Git source to sync to the cluster. Example: `--tag=3.2.0`. |
| `--semver` | String | Git tag `semver` range within the Git source to sync to the cluster. Example: `--semver=">=3.1.0-rc.1 <3.2.0"`. |
| `--commit` | String | Git commit SHA within the Git source to sync to the cluster. Example: `--commit=363a6a8fe6a7f13e05d34c163b0ef02a777da20a`. |

For more information, see the [Flux documentation on Git repository checkout strategies](https://fluxcd.io/docs/components/source/gitrepositories/#checkout-strategies).

### Public Git repository

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | `http[s]://server/repo[.git]` | URL of the Git repository source to reconcile with the cluster. |

### Private Git repository with SSH

> [!IMPORTANT]
> Azure DevOps [announced the deprecation of SSH-RSA](https://aka.ms/ado-ssh-rsa-deprecation) as a supported encryption method for connecting to Azure repositories using SSH. If you use SSH keys to connect to Azure repositories in Flux configurations, we recommend you move to more secure RSA-SHA2-256 or RSA-SHA2-512 keys. For more information, see [Azure DevOps SSH-RSA deprecation](tutorial-use-gitops-flux2.md#azure-devops-ssh-rsa-deprecation).

#### Private Git repository with SSH and Flux-created keys

Add the public key generated by Flux to the user account in your Git service provider.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | `ssh://user@server/repo[.git]` | `git@` should replace `user@` if the public key is associated with the repository instead of the user account. |

#### Private Git repository with SSH and user-provided keys

Use your own private key directly or from a file. The key must be in [PEM format](https://aka.ms/PEMformat) and end with a newline (`\n`).

Add the associated public key to the user account in your Git service provider.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | ssh://user@server/repo[.git] | `git@` should replace `user@` if the public key is associated with the repository instead of the user account. |
| `--ssh-private-key` | Base64 key in [PEM format](https://aka.ms/PEMformat) | Provide the key directly. |
| `--ssh-private-key-file` | Full path to local file | Provide the full path to the local file that contains the PEM-format key.

#### Private Git host with SSH and user-provided known hosts

The Flux operator maintains a list of common Git hosts in its `known_hosts` file. Flux uses this information to authenticate the Git repository before establishing the SSH connection. If you're using an uncommon Git repository or your own Git host, you can supply the host key so that Flux can identify your repository.

Just like private keys, you can provide your `known_hosts` content directly or in a file. When you're providing your own content, use the [known_hosts content format specifications](https://aka.ms/KnownHostsFormat), along with either of the preceding SSH key scenarios.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | ssh://user@server/repo[.git] | `git@` can replace `user@`. |
| `--known-hosts` | Base64 string | Provide `known_hosts` content directly. |
| `--known-hosts-file` | Full path to local file | Provide `known_hosts` content in a local file. |

### Private Git repository with an HTTPS user and key

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | `https://server/repo[.git]` | HTTPS with Basic Authentication. |
| `--https-user` | Raw string | HTTPS username. |
| `--https-key` | Raw string | HTTPS personal access token or password. |

### Private Git repository with an HTTPS CA certificate

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | `https://server/repo[.git]` | HTTPS with Basic Authentication. |
| `--https-ca-cert` | Base64 string | CA certificate for TLS communication. |
| `--https-ca-cert-file` | Full path to local file | Provide CA certificate content in a local file. |

## Bucket source arguments

If you use the `bucket` source, use these bucket-specific command arguments.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | URL String | The URL for the `bucket`. Supported formats: `http://`, `https://`. |
| `--bucket-name` | String | Name of the `bucket` to sync. |
| `--bucket-access-key` | String | Access Key ID used to authenticate with the `bucket`. |
| `--bucket-secret-key` | String | Secret Key used to authenticate with the `bucket`. |
| `--bucket-insecure` | Boolean | Communicate with a `bucket` without TLS.  If not provided, assumed false; if provided, assumed true. |

## Azure Blob Storage Account source arguments

If you use the `azblob` source, here are the blob-specific command arguments.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--url` `-u` | URL String | The URL for the `azblob`. |
| `--container-name` | String | Name of the Azure Blob Storage container to sync. |
| `--sp-client-id` | String | The client ID for authenticating a service principal with Azure Blob. This authentication method requires this parameter. |
| `--sp-tenant-id` | String | The tenant ID for authenticating a service principal with Azure Blob. This authentication method requires this parameter. |
| `--sp-client-secret` | String | The client secret for authenticating a service principal with Azure Blob. |
| `--sp-client-cert` | String | The Base64-encoded client certificate for authenticating a service principal with Azure Blob. |
| `--sp-cert-password` | String | The password for the client certificate used to authenticate a service principal with Azure Blob. |
| `--sp-cert-send-chain` | String | Specifies whether to include the x5c header in client claims when acquiring a token to enable subject name or issuer-based authentication for the client certificate. |
| `--account-key` | String | The Azure Blob Shared Key for authentication. |
| `--sas-token` | String | The Azure Blob SAS token for authentication. |
| `--managed-identity-client-id` | String | The client ID of the managed identity for authentication with Azure Blob. |

> [!IMPORTANT]
> When you use managed identity authentication for AKS clusters and `azblob` source, assign the managed identity at minimum the [Storage Blob Data Reader](/azure/role-based-access-control/built-in-roles#storage-blob-data-reader) role. Authentication by using a managed identity isn't available yet for Azure Arc-enabled Kubernetes clusters.

## Local secret for authentication with source

Use a local Kubernetes secret for authentication with a `git`, `bucket`, or `azblob` source. The local secret must contain all of the authentication parameters needed for the source and must be created in the same namespace as the Flux configuration.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--local-auth-ref` `--local-ref`  | String | Local reference to a Kubernetes secret in the Flux configuration namespace to use for authentication with the source. |

For HTTPS authentication, create a secret with the `username` and `password`:

```azurecli
kubectl create ns flux-config
kubectl create secret generic -n flux-config my-custom-secret --from-literal=username=<my-username> --from-literal=password=<my-password-or-key>
```

For SSH authentication, create a secret with the `identity` and `known_hosts` fields:

```azurecli
kubectl create ns flux-config
kubectl create secret generic -n flux-config my-custom-secret --from-file=identity=./id_rsa --from-file=known_hosts=./known_hosts
```

> [!IMPORTANT]
> Azure DevOps [announced the deprecation of SSH-RSA](https://aka.ms/ado-ssh-rsa-deprecation) as a supported encryption method for connecting to Azure repositories using SSH. If you use SSH keys to connect to Azure repositories in Flux configurations, we recommend you move to more secure RSA-SHA2-256 or RSA-SHA2-512 keys. For more information, see [Azure DevOps SSH-RSA deprecation](tutorial-use-gitops-flux2.md#azure-devops-ssh-rsa-deprecation).

For both cases, when you create the Flux configuration, use `--local-auth-ref my-custom-secret` in place of the other authentication parameters:

```azurecli
az k8s-configuration flux create -g <cluster_resource_group> -c <cluster_name> -n <config_name> -t connectedClusters --scope cluster --namespace flux-config -u <git-repo-url> --kustomization name=kustomization1 --local-auth-ref my-custom-secret
```

Learn more about using a local Kubernetes secret with these authentication methods:

- [Git repository HTTPS authentication](https://fluxcd.io/docs/components/source/gitrepositories/#https-authentication)
- [Git repository HTTPS self-signed certificates](https://fluxcd.io/docs/components/source/gitrepositories/#https-self-signed-certificates)
- [Git repository SSH authentication](https://fluxcd.io/docs/components/source/gitrepositories/#ssh-authentication)
- [Bucket static authentication](https://fluxcd.io/docs/components/source/buckets/#static-authentication)

> [!NOTE]
> If you need Flux to access the source through your proxy, update the Azure Arc agents with the proxy settings. For more information, see [Connect using an outbound proxy server](./quickstart-connect-cluster.md#connect-using-an-outbound-proxy-server).

## Git implementation

To support various repository providers that implement Git, Flux can be configured to use one of two Git libraries: `go-git` or `libgit2`. For details, see the [Flux documentation](https://fluxcd.io/docs/components/source/gitrepositories/#git-implementation).

The GitOps implementation of Flux v2 automatically determines which library to use for public cloud repositories:

- For GitHub, GitLab, and Bitbucket repositories, Flux uses `go-git`.
- For Azure DevOps and all other repositories, Flux uses `libgit2`.

For on-premises repositories, Flux uses `libgit2`.

## Kustomization

Kustomization is a setting created for Flux configurations that you use to choose a specific path in the source repo that reconciles into the cluster. You don't need to create a `kustomization.yaml` file on this specified path. By default, the cluster reconciles all of the manifests in this path. However, if you want to have a Kustomize overlay for applications available on this repo path, create [Kustomize files](https://kustomize.io/) in Git for the Flux configuration to use.

By using [`az k8s-configuration flux kustomization create`](/cli/azure/k8s-configuration/flux/kustomization#az-k8s-configuration-flux-kustomization-create), you can create one or more kustomizations during the configuration.

| Parameter | Format | Notes |
| ------------- | ------------- | ------------- |
| `--kustomization` | No value | Start of a string of parameters that configure a kustomization. Use it multiple times to create multiple kustomizations. |
| `name` | String | Unique name for this kustomization. |
| `path` | String | Path within the Git repository to reconcile with the cluster. Default is the top level of the branch. |
| `prune` | Boolean | Default is `false`. Set `prune=true` to ensure that the objects Flux deployed to the cluster are cleaned up if they're removed from the repository or if the Flux configuration or kustomizations are deleted. Using `prune=true` is important for environments where users don't have access to the clusters and can make changes only through the Git repository. |
| `depends_on` | String | Name of one or more kustomizations (within this configuration) that must reconcile before this kustomization can reconcile. For example: `depends_on=["kustomization1","kustomization2"]`. If you remove a kustomization that has dependent kustomizations, the state of dependent kustomizations becomes `DependencyNotReady`, and reconciliation halts.|
| `timeout` | [golang duration format](https://pkg.go.dev/time#Duration.String) | Default: `10m`.  |
| `sync_interval` | [golang duration format](https://pkg.go.dev/time#Duration.String) | Default: `10m`.  |
| `retry_interval` | [golang duration format](https://pkg.go.dev/time#Duration.String) | Default: `10m`.  |
| `validation` | String | Values: `none`, `client`, `server`. Default: `none`.  See [Flux documentation](https://fluxcd.io/docs/) for details.|
| `force` | Boolean | Default: `false`. Set `force=true` to instruct the kustomize controller to re-create resources when patching fails because of an immutable field change. |

You can also use [`az k8s-configuration flux kustomization`](/cli/azure/k8s-configuration/flux/kustomization) to update, list, show, and delete kustomizations in a Flux configuration.

## Next steps

- Learn more about [Application deployments with GitOps (Flux v2) for AKS and Azure Arc-enabled Kubernetes](conceptual-gitops-flux2.md).
- Use our tutorial to learn how to [enable GitOps on your AKS or Azure Arc-enabled Kubernetes clusters](tutorial-use-gitops-flux2.md).
- Learn about [CI/CD workflow using GitOps](conceptual-gitops-flux2-ci-cd.md).
