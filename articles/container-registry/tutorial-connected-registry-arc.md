---
title: Secure Deployment Options for the Connected Registry Extension
description: "Learn to secure the connected registry Arc extension deployment with HTTPS, TLS, optional no TLS, BYOC certificate, and trust distribution."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to  #Don't change.
ms.date: 05/20/2025
ms.custom: sfi-ropc-nochange


# Customer intent: "As a DevOps engineer, I want to securely deploy the connected registry Arc extension with TLS encryption and trust distribution management, so that I can ensure safe and trusted communication between containers and the registry in my Kubernetes cluster."
---

# Secure deployment options for the connected registry extension with Azure Container Registry

This article describes various deployment scenarios for the connected registry extension in an Arc-enabled Kubernetes cluster. After you install the connected registry extension, you can synchronize images from your cloud registry to on-premises or remote locations.  

Before you dive in, take a moment to learn how [Arc-enabled Kubernetes][Arc-enabled Kubernetes] works conceptually.

You can securely deploy the connected registry by using various encryption methods. To ensure a successful deployment, follow the quickstart guide to review prerequisites and other pertinent information. By default, the connected registry is configured with HTTPS, ReadOnly mode, Trust Distribution, and the Cert Manager service. You can add more customizations and dependencies as needed, depending on your scenario.

### What is Cert Manager service? 

The connected registry cert manager is a service that manages TLS certificates for the connected registry extension in an Azure Arc-enabled Kubernetes cluster. It ensures secure communication between the connected registry and other components by handling the creation, renewal, and distribution of certificates. You can install this service as part of the connected registry deployment, or you can use an existing cert manager if it's already installed on your cluster. 

[Cert-Manager][cert-manager] is an open-source Kubernetes add-on that automates the management and issuance of TLS certificates from various sources. It manages the lifecycle of certificates issued by CA pools created using CA Service, ensuring they are valid and renewed before they expire.  

### What is trust distribution?

Connected registry trust distribution refers to the process of securely distributing trust between the connected registry service and Kubernetes clients within a cluster. By using a Certificate Authority (CA), such as cert-manager, to sign TLS certificates, you can distribute these certificates to both the registry service and the clients. This process ensures that all entities can securely authenticate each other, maintaining a secure and trusted environment within the Kubernetes cluster.

## Deploy connected registry extension by using your preinstalled cert-manager

When you use a preinstalled cert-manager service on the cluster, you have control over certificate management. Deploy the connected registry extension with encryption by following the steps provided:

Run the [az-k8s-extension-create][az-k8s-extension-create] command in the [quickstart][quickstart] and set the `cert-manager.enabled=true` and `cert-manager.install=false` parameters to indicate the cert-manager service is installed and enabled:

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

Bring your own certificate (BYOC) allows you to use your own public certificate and private key pair, giving you control over certificate management. This setup enables you to deploy the connected registry extension with encryption.

> [!NOTE]
> BYOC is useful for customers who bring their own certificate that is already trusted by their Kubernetes nodes. Don't manually update nodes to trust certificates.

Add the public certificate and private key string variable and value pair when deploying the extension by following these steps:

1. Create self-signed SSL cert with connected-registry service IP as the SAN:

   ```bash
   mkdir /certs
   ```

   ```bash
   openssl req -newkey rsa:4096 -nodes -sha256 -keyout /certs/mycert.key -x509 -days 365 -out /certs/mycert.crt -addext   "subjectAltName = IP:<service IP>"
   ```

1. Get base64 encoded strings of these cert files:

   ```bash
   export TLS_CRT=$(cat mycert.crt | base64 -w0) 
   export TLS_KEY=$(cat mycert.key | base64 -w0) 
   ```

1. Create a protected settings file with secret in JSON format, similar to the following example:

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

1. Deploy the Connected registry extension with HTTPS (TLS encryption) using the public certificate and private key pair management by configuring variables set to `cert-manager.enabled=false` and `cert-manager.install=false`. By using these parameters, the cert-manager isn't installed or enabled, since the public certificate and private key pair is used instead for encryption.  

1. Run the [az-k8s-extension-create][az-k8s-extension-create] command for deployment after editing the protected settings file:

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

By using a [Kubernetes secret][Kubernetes secret] on your cluster, you can securely manage authorized access between pods within the cluster. This setup enables you to deploy the connected registry extension with encryption.

Add the Kubernetes TLS secret string variable and value pair by following these steps:

1. Create self-signed SSL cert with connected-registry service IP as the SAN

   ```bash
   mkdir /certs
   ```

   ```bash
   openssl req -newkey rsa:4096 -nodes -sha256 -keyout /certs/mycert.key -x509 -days 365 -out /certs/mycert.crt -addext    "subjectAltName = IP:<service IP>"
   ```

1. Get base64-encoded strings of these cert files:

   ```bash
   export TLS_CRT=$(cat mycert.crt | base64 -w0) 
   export TLS_KEY=$(cat mycert.key | base64 -w0) 
   ```

1. Create the Kubernetes secret:

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

1. Create a protected settings file with secret in JSON format, similar to the following example:

   ```json
       { 
       "connectionString": "[connection string here]",
       "tls.secret": “k8secret” 
       } 
   ```

1. Deploy the Connected registry extension with HTTPS (TLS encryption) by using the Kubernetes secret management. Configure variables set to `cert-manager.enabled=false` and `cert-manager.install=false`. By using these parameters, the cert-manager isn't installed or enabled since the Kubernetes secret is used instead for encryption.  

1. Run the [az-k8s-extension-create][az-k8s-extension-create] command for deployment after editing the protected settings file:

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

## Deploy the connected registry using your own trust distribution 

By using your own Kubernetes secret or public certificate and private key pairs, you can deploy the connected registry extension with TLS encryption, your inherent trust distribution, and reject the connected registry's default trust distribution. This setup enables you to deploy the connected registry extension with encryption. Be sure to add either the Kubernetes secret or public certificate, and private key variable and value pairs in the protected settings file in JSON format.

To use this option, set the `trustDistribution.enabled=false` and `trustDistribution.skipNodeSelector=false` parameters to reject connected registry trust distribution when you deploy the extension:

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

By using these parameters, cert-manager isn't installed or enabled, and the connected registry trust distribution isn't enforced. Instead, you use the cluster-provided trust distribution for establishing trust between the connected registry and the client nodes.

## Clean up resources

Follow these steps to remove the connected registry extension, the corresponding Connected registry pods, and the configuration settings.

1. Run [az-k8s-extension-delete][az-k8s-extension-delete] to delete the connected registry extension:

   ```azurecli
   az k8s-extension delete --name myconnectedregistry 
   --cluster-name myarcakscluster \ 
   --resource-group myresourcegroup \ 
   --cluster-type connectedClusters
   ```

1. Run  [az acr connected-registry delete][az-acr-connected-registry-delete] to delete the connected registry resource:

   ```azurecli 
   az acr connected-registry delete --registry myacrregistry \ 
   --name myconnectedregistry \ 
   --resource-group myresourcegroup
   ```

## Next steps

- [Enable connected registry with Azure CLI][quickstart]
- [Sync connected registry with Azure Arc in a scheduled window](tutorial-connected-registry-sync.md)
- [Troubleshoot connected registry issues](troubleshoot-connected-registry-arc.md)

<!-- LINKS - internal -->
[quickstart]: quickstart-connected-registry-arc-cli.md
[Arc-enabled Kubernetes]: /azure/azure-arc/kubernetes/overview
[cert-manager]: https://cert-manager.io/
[Kubernetes secret]: https://kubernetes.io/docs/concepts/configuration/secret/

<!-- LINKS - external -->
[az-k8s-extension-create]: /cli/azure/k8s-extension#az-k8s-extension-create
[az-k8s-extension-delete]: /cli/azure/k8s-extension#az-k8s-extension-delete
[az-acr-connected-registry-delete]: /cli/azure/acr/connected-registry#az-acr-connected-registry-delete
