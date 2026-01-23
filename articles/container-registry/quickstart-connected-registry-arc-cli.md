---
title: "Deploy the Connected Registry Arc Extension"
description: "Learn how to deploy the connected registry extension to an Arc-enabled Kubernetes cluster using Azure CLI with secure-by-default settings for efficient and secure container workload operations."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to
ms.date: 01/23/2026
ai-usage: ai-assisted
ms.custom: sfi-ropc-blocked

# Customer intent: As a DevOps engineer, I want to deploy the Connected Registry Arc extension using CLI with secure settings, so that I can manage container workloads efficiently and securely in my Kubernetes cluster.
---

# Deploy the connected registry Arc extension

In this article, you learn how to deploy the connected registry extension to an Arc-enabled Kubernetes cluster using Azure CLI.

The [connected registry feature of Azure Container Registry](intro-connected-registry.md) enables efficient management and access to containerized workloads, whether on-premises or at remote sites. By integrating with Azure Arc, the service ensures a seamless and unified lifecycle management experience for Kubernetes-based containerized workloads. Deploying the connected registry Arc extension on Arc-enabled Kubernetes clusters simplifies the management and access of these workloads.

## Prerequisites

* An [Azure Container Registry (ACR)][create-acr] with the **Premium** service tier (SKU), with at least one repository and a connected registry. If you don't already have a connected registry resource, follow the steps in [Create a connected registry](quickstart-create-connected-registry.md).

* An [Azure Arc-enabled Kubernetes cluster][quickstart-connect-cluster].

* The latest version of [Azure CLI][Install Azure CLI].

* The latest versions of the `connectedk8s` and `k8s-extension` Azure CLI extensions. To install these extensions, run the following commands:

   ```azurecli
   az extension add --name k8s-extension
   az extension add --name connectedk8s
   ```

## Deploy the connected registry extension to the Arc-enabled Kubernetes cluster

By deploying the connected registry Arc extension, you can synchronize container images and other Open Container Initiative (OCI) artifacts with your ACR registry. The deployment helps speed-up access to registry artifacts and enables the building of advanced scenarios. The extension deployment ensures secure trust distribution between the connected registry and all client nodes within the cluster, and installs the cert-manager service for Transport Layer Security (TLS) encryption.

To learn more about deploying extensions to Azure Arc-enabled Kubernetes clusters, see [Deploy and manage an Azure Arc-enabled Kubernetes cluster extension](/azure/azure-arc/kubernetes/extensions).

### Generate the connection string and protected settings JSON File

For secure deployment of the connected registry extension, generate the connection string, including a new password and the specification of the transport protocol, and store the details in the `protected-settings-extension.json` file.

### [Bash](#tab/bash)

```bash
cat << EOF > protected-settings-extension.json
{
  "connectionString": "$(az acr connected-registry get-settings \
  --name myconnectedregistry \
  --registry myacrregistry \
  --parent-protocol https \
  --generate-password 1 \
  --query ACR_REGISTRY_CONNECTION_STRING --output tsv --yes)"
}
EOF
```

### [PowerShell](#tab/powershell)

```azurepowershell
echo "{\"connectionString\":\"$(az acr connected-registry get-settings \
--name myconnectedregistry \
--registry myacrregistry \
--parent-protocol https \
--generate-password 1 \
--query ACR_REGISTRY_CONNECTION_STRING \
--output tsv \
--yes | tr -d '\r')\" }" > settings.json
```

---

### Deploy the connected registry extension

Next, deploy the connected registry extension to your Arc-enabled Kubernetes cluster, referencing the JSON file you created. Use the [az k8s-extension create][az-k8s-extension-create] command:

  ```azurecli
    az k8s-extension create --cluster-name myarck8scluster \ 
    --cluster-type connectedClusters \
    --extension-type Microsoft.ContainerRegistry.ConnectedRegistry \
    --name myconnectedregistry \
    --resource-group myresourcegroup \ 
    --config service.clusterIP=192.100.100.1 \ 
    --config-protected-file protected-settings-extension.json  
    --auto-upgrade-minor-version true
  ```

The `clusterIP` must be from the cluster subnet IP range. The `service.clusterIP` parameter specifies the IP address of the connected registry service within the cluster. Ensure that the IP address you specify for `service.clusterIP` falls within the designated service IP range defined during the cluster's initial configuration, typically found in the cluster's networking settings. If the `service.clusterIP` isn't within this range, update it to an IP address that is both within the valid range and not currently in use by another service.

> [!TIP]
> The example here enables automatic upgrades for the connected registry extension whenever a new version becomes available. If you prefer not to automatically upgrade, use `--auto-upgrade-minor-version false`.
>
> To deploy a specific version of the connected registry extension, include the `--version <version number>` parameter in your [az-k8s-extension-create][az-k8s-extension-create] command.

## Verify the connected registry extension deployment

To verify the deployment of the connected registry extension in the Arc-enabled Kubernetes cluster, run the [az k8s-extension show][az-k8s-extension-show] command to check the deployment status of the connected registry extension:

  ```azurecli
    az k8s-extension show 
    --name myconnectedregistry \ 
    --cluster-name myarck8scluster \
    --resource-group myresourcegroup \
    --cluster-type connectedClusters
  ```

For an example of the output, see [Show extension details](/azure/azure-arc/kubernetes/extensions#show-extension-details).

Next, for each connected registry, view the status and state of the connected registry using the [az acr connected-registry list][az-acr-connected-registry-list] command:

```azurecli
az acr connected-registry list --registry myacrregistry \
--output table
```

You should see output similar to the following example:

```console
    | NAME | MODE | CONNECTION STATE | PARENT | LOGIN SERVER | LAST SYNC(UTC) |
    |------|------|------------------|--------|--------------|----------------|
    | myconnectedregistry | ReadWrite | online | myacrregistry | myacrregistry.azurecr.io | 2026-01-09 12:00:00 |
    | myreadonlyacr | ReadOnly | offline | myacrregistry | myacrregistry.azurecr.io | 2026-01-09 12:00:00 |
```

For details on a specific connected registry, use [az acr connected-registry show][az-acr-connected-registry-show]:

```azurecli
  az acr connected-registry show --registry myacrregistry \
  --name myreadonlyacr \ 
  --output table
```

This command shows output similar to the following example:

```console
   | NAME                | MODE      | CONNECTION STATE | PARENT        | LOGIN SERVER             | LAST SYNC(UTC)      | SYNC SCHEDULE | SYNC WINDOW       |
   | ------------------- | --------- | ---------------- | ------------- | ------------------------ | ------------------- | ------------- | ----------------- |
   | myconnectedregistry | ReadWrite | online           | myacrregistry | myacrregistry.azurecr.io | 2026-01-09 12:00:00 | 0 0 * * *     | 00:00:00-23:59:59 |
```

## Deploy a pod that uses an image from connected registry

To deploy a pod that uses an image from connected registry within the cluster, run the operation from the cluster node itself.

1. Run the [kubectl create secret docker-registry][kubectl-create-secret-docker-registry] command to create a secret in the cluster to authenticate with the connected registry:

   ```bash
   kubectl create secret docker-registry regcred --docker-server=192.100.100.1 --docker-username=mytoken --docker-password=mypassword
   ```

1. Deploy the pod that uses the desired image from the connected registry. This example uses service.clusterIP address `192.100.100.1` and image name `hello-world` with tag `latest`:

    ```bash
    kubectl apply -f - <<EOF
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: hello-world-deployment
      labels:
        app: hello-world
    spec:
      selector:
        matchLabels:
          app: hello-world
      replicas: 1
      template:
        metadata:
          labels:
            app: hello-world
        spec:
          imagePullSecrets:
            - name: regcred
          containers:
            - name: hello-world
              image: 192.100.100.1/hello-world:latest
    EOF
    ```

## Clean up resources

If you no longer want to use your connected registry, delete the extension from your Arc-enabled Kubernetes cluster and then delete the connected registry resource.

1. Run the [az k8s-extension delete][az-k8s-extension-delete] command to delete the connected registry extension:

    ```azurecli
    az k8s-extension delete --name myconnectedregistry 
    --cluster-name myarcakscluster \ 
    --resource-group myresourcegroup \ 
    --cluster-type connectedClusters
    ```

1. Run the [az acr connected-registry delete][az-acr-connected-registry-delete] command to delete the connected registry resource:

    ```azurecli
    az acr connected-registry delete --registry myacrregistry \
    --name myconnectedregistry \
    --resource-group myresourcegroup 
    ```

<!-- LINKS - internal -->
[create-acr]: container-registry-get-started-azure-cli.md
[Install Azure CLI]: /cli/azure/install-azure-cli
[quickstart-connect-cluster]: /azure/azure-arc/kubernetes/quickstart-connect-cluster
[az-k8s-extension-create]: /cli/azure/k8s-extension#az-k8s-extension-create
[az-k8s-extension-show]: /cli/azure/k8s-extension#az-k8s-extension-show
[az-acr-connected-registry-list]: /cli/azure/acr/connected-registry#az-acr-connected-registry-list
[az-acr-connected-registry-show]: /cli/azure/acr/connected-registry#az-acr-connected-registry-show
[az-k8s-extension-delete]: /cli/azure/k8s-extension#az-k8s-extension-delete
[az-acr-connected-registry-delete]: /cli/azure/acr/connected-registry#az-acr-connected-registry-delete
[kubectl-create-secret-docker-registry]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_docker-registry/