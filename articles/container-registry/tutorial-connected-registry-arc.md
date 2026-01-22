---
title: Secure and Deploy Connected Registry Arc Extension
description: "Learn to secure the connected registry Arc extension deployment with HTTPS, TLS, optional no TLS, BYOC certificate, and trust distribution."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: tutorial  #Don't change.
ms.date: 05/20/2025
ms.custom: sfi-ropc-nochange


# Customer intent: "As a DevOps engineer, I want to securely deploy the connected registry Arc extension with TLS encryption and trust distribution management, so that I can ensure safe and trusted communication between containers and the registry in my Kubernetes cluster."
---

# Tutorial: Secure deployment methods for the connected registry extension

These tutorials cover various deployment scenarios for the connected registry extension in an Arc-enabled Kubernetes cluster. Once the connected registry extension is installed, you can synchronize images from your cloud registry to on-premises or remote locations.  

Before you dive in, take a moment to learn how [Arc-enabled Kubernetes][Arc-enabled Kubernetes] works conceptually.

The connected registry can be securely deployed using various encryption methods. To ensure a successful deployment, follow the quickstart guide to review prerequisites and other pertinent information. By default, the connected registry is configured with HTTPS, ReadOnly mode, Trust Distribution, and the Cert Manager service. You can add more customizations and dependencies as needed, depending on your scenario. 

### What is Cert Manager service? 

The connected registry cert manager is a service that manages TLS certificates for the connected registry extension in an Azure Arc-enabled Kubernetes cluster. It ensures secure communication between the connected registry and other components by handling the creation, renewal, and distribution of certificates. This service can be installed as part of the connected registry deployment, or you can use an existing cert manager if it's already installed on your cluster. 

[Cert-Manager][cert-manager] is an open-source Kubernetes add-on that automates the management and issuance of TLS certificates from various sources. It manages the lifecycle of certificates issued by CA pools created using CA Service, ensuring they are valid and renewed before they expire.  

### What is trust distribution?

Connected registry trust distribution refers to the process of securely distributing trust between the connected registry service and Kubernetes clients within a cluster. This is achieved by using a Certificate Authority (CA), such as cert-manager, to sign TLS certificates, which are then distributed to both the registry service and the clients. This ensures that all entities can securely authenticate each other, maintaining a secure and trusted environment within the Kubernetes cluster.

## Prerequisites

To complete this tutorial, you need:

* Follow the [quickstart][quickstart] to securely deploy the connected registry extension. 

## Deploy connected registry extension using your preinstalled cert-manager

In this tutorial, we demonstrate how to use a preinstalled cert-manager service on the cluster. This setup gives you control over certificate management, enabling you to deploy the connected registry extension with encryption by following the steps provided:

Run the [az-k8s-extension-create][az-k8s-extension-create] command in the [quickstart][quickstart] and set the `cert-manager.enabled=true` and `cert-manager.install=false` parameters to determine the cert-manager service is installed and enabled:

```azurecli
    az k8s-extension create --cluster-name myarck8scluster \ 
    --cluster-type connectedClusters \ 
    --extension-type Microsoft.ContainerRegistry.ConnectedRegistry \ 
    --name myconnectedregistry \ 
    --resource-group myresourcegroup \ 
    --config service.clusterIP=192.100.100.1 \ 
    --config cert-manager.install=false \ 
    --config-protected-file protected-settings-extension.json
```

## Deploy connected registry extension using bring your own certificate (BYOC)

In this tutorial, we demonstrate how to use your own certificate (BYOC) on the cluster. BYOC allows you to use your own public certificate and private key pair, giving you control over certificate management. This setup enables you to deploy the connected registry extension with encryption by following the provided steps:

>[!NOTE] 
>BYOC is applicable for customers who bring their own certificate that is already trusted by their Kubernetes nodes. It is not recommended to manually update the nodes to trust the certificates.

Follow the [quickstart][quickstart] and add the public certificate and private key string variable + value pair. 

1. Create self-signed SSL cert with connected-registry service IP as the SAN

```bash
  mkdir /certs
```

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout /certs/mycert.key -x509 -days 365 -out /certs/mycert.crt -addext "subjectAltName = IP:<service IP>"
```

2. Get base64 encoded strings of these cert files

```bash
export TLS_CRT=$(cat mycert.crt | base64 -w0) 
export TLS_KEY=$(cat mycert.key | base64 -w0) 
```

3. Protected settings file example with secret in JSON format: 

> [!NOTE]
> The public certificate and private key pair must be encoded in base64 format and added to the protected settings file.
    
```json   
    {
    "connectionString": "[connection string here]",
    "tls.crt": $TLS_CRT,
    "tls.key": $TLS_KEY,
    "tls.cacrt": $TLS_CRT
    } 
```

4. Now, you can deploy the Connected registry extension with HTTPS (TLS encryption) using the public certificate and private key pair management by configuring variables set to `cert-manager.enabled=false` and `cert-manager.install=false`. With these parameters, the cert-manager isn't installed or enabled since the public certificate and private key pair is used instead for encryption.  

5. Run the [az-k8s-extension-create][az-k8s-extension-create] command for deployment after protected settings file is edited:

    ```azurecli
    az k8s-extension create --cluster-name myarck8scluster \
    --cluster-type connectedClusters \  
    --extension-type  Microsoft.ContainerRegistry.ConnectedRegistry \ 
    --name myconnectedregistry \ 
    --resource-group myresourcegroup \
    --config service.clusterIP=192.100.100.1 \
    --config cert-manager.enabled=false \
    --config cert-manager.install=false \ 
    --config-protected-file protected-settings-extension.json 
    ```

## Deploy connected registry with Kubernetes secret management

In this tutorial, we demonstrate how to use a [Kubernetes secret][Kubernetes secret] on your cluster. Kubernetes secret allows you to securely manage authorized access between pods within the cluster. This setup enables you to deploy the connected registry extension with encryption by following the provided steps:

Follow the [quickstart][quickstart] and add the Kubernetes TLS secret string variable + value pair. 

1. Create self-signed SSL cert with connected-registry service IP as the SAN

```bash
mkdir /certs
```

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout /certs/mycert.key -x509 -days 365 -out /certs/mycert.crt -addext "subjectAltName = IP:<service IP>"
```

2. Get base64 encoded strings of these cert files

```bash
export TLS_CRT=$(cat mycert.crt | base64 -w0) 
export TLS_KEY=$(cat mycert.key | base64 -w0) 
```

3. Create k8s secret

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: k8secret
  type: kubernetes.io/tls
data:
  ca.crt: $TLS_CRT
  tls.crt: $TLS_CRT
  tls.key: $TLS_KEY
EOF
```

4. Protected settings file example with secret in JSON format: 

    ```json
        { 
        "connectionString": "[connection string here]",
        "tls.secret": “k8secret” 
        } 
    ```

Now, you can deploy the Connected registry extension with HTTPS (TLS encryption) using the Kubernetes secret management by configuring variables set to `cert-manager.enabled=false` and `cert-manager.install=false`. With these parameters, the cert-manager isn't installed or enabled since the Kubernetes secret is used instead for encryption.  

5. Run the [az-k8s-extension-create][az-k8s-extension-create] command for deployment after protected settings file is edited:

    ```azurecli
    az k8s-extension create --cluster-name myarck8scluster \
    --cluster-type connectedClusters \  
    --extension-type  Microsoft.ContainerRegistry.ConnectedRegistry \ 
    --name myconnectedregistry \ 
    --resource-group myresourcegroup \
    --config service.clusterIP=192.100.100.1 \
    --config cert-manager.enabled=false \
    --config cert-manager.install=false \ 
    --config-protected-file protected-settings-extension.json 
    ```

## Deploy the connected registry using your own trust distribution and disable the connected registry's default trust distribution

In this tutorial, we demonstrate how to configure trust distribution on the cluster. While using your own Kubernetes secret or public certificate and private key pairs, you can deploy the connected registry extension with TLS encryption, your inherent trust distribution, and reject the connected registry’s default trust distribution. This setup enables you to deploy the connected registry extension with encryption by following the provided steps:

1. Follow the [quickstart][quickstart] to add either the Kubernetes secret or public certificate, and private key variable + value pairs in the protected settings file in JSON format.

2. Run the [az-k8s-extension-create][az-k8s-extension-create] command in [quickstart][quickstart] and set the `trustDistribution.enabled=false`, `trustDistribution.skipNodeSelector=false` parameters to reject Connected registry trust distribution:
    
    ```azurecli
    az k8s-extension create --cluster-name myarck8scluster \ 
    --cluster-type connectedClusters \ 
    --extension-type Microsoft.ContainerRegistry.ConnectedRegistry \ 
    --name myconnectedregistry \ 
    --resource-group myresourcegroup \ 
    --config service.clusterIP=192.100.100.1 \
    --config trustDistribution.enabled=false \ 
    --config cert-manager.enabled=false \
    --config cert-manager.install=false \ 
    --config-protected-file <JSON file path> 
    ```

With these parameters, cert-manager isn't installed or enabled, additionally, the Connected registry trust distribution isn't enforced. Instead you're using the cluster provided trust distribution for establishing trust between the Connected registry and the client nodes.

## Clean up resources

By deleting the deployed Connected registry extension, you remove the corresponding Connected registry pods and configuration settings.   

1. Run the [az-k8s-extension-delete][az-k8s-extension-delete] command to delete the Connected registry extension:

    ```azurecli
    az k8s-extension delete --name myconnectedregistry 
    --cluster-name myarcakscluster \ 
    --resource-group myresourcegroup \ 
    --cluster-type connectedClusters
    ```   

2. Run the [az acr connected-registry delete][az-acr-connected-registry-delete] command to delete the Connected registry:

    ```azurecli 
    az acr connected-registry delete --registry myacrregistry \ 
    --name myconnectedregistry \ 
    --resource-group myresourcegroup
    ```

By deleting the Connected registry extension and the Connected registry, you remove all the associated resources and configurations.

## Next steps

- [Enable Connected registry with Azure Arc CLI][quickstart]
- [Sync Connected registry with Azure Arc in Scheduled window](tutorial-connected-registry-sync.md)
- [Troubleshoot Connected registry with Azure Arc](troubleshoot-connected-registry-arc.md)

<!-- LINKS - internal -->
[quickstart]: quickstart-connected-registry-arc-cli.md
[Arc-enabled Kubernetes]: /azure/azure-arc/kubernetes/overview
[cert-manager]: https://cert-manager.io/
[Kubernetes secret]: https://kubernetes.io/docs/concepts/configuration/secret/

<!-- LINKS - external -->
[az-k8s-extension-create]: /cli/azure/k8s-extension#az-k8s-extension-create
[az-k8s-extension-delete]: /cli/azure/k8s-extension#az-k8s-extension-delete
[az-acr-connected-registry-delete]: /cli/azure/acr/connected-registry#az-acr-connected-registry-delete
