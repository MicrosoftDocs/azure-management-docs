---
title: Cloud Ingest Edge Volumes with Workload Identity  
description: Learn how to configure Cloud Ingest Edge Volumes with Workload Identity.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 06/25/2025

#CustomerIntent: As a Kubernetes administrator, I want to configure Cloud Ingest Edge Volumes with Workload Identity.
---

# Cloud Ingest Edge Volumes with Workload Identity

This article describes how to configure *Cloud Ingest Edge Volumes* with Workload Identity.

## Prerequisites

Before you begin, ensure you have followed the prerequisites in the [Cloud Ingest Edge Volumes configuration](cloud-ingest-edge-volume-configuration.md).

## Configure Kubernetes cluster for Workload Identity

If you wish to use Workload Identity with Azure Container Storage Enabled by Azure Arc you must first enable the features for your Azure Arc Connected Kubernetes Cluster. Enable the OIDC issuer and workload identity by running the following command:

```azurecli
az connectedk8s update -n <CLUSTER_NAME> -g <RESOURCE_GROUP> --enable-oidc-issuer --enable-workload-identity
```

Next you need to configure your Kubernetes cluster to utilize workload identity:

1. Obtain the Issuer URL:

    ```azurecli
    az connectedk8s show --resource-group <RESOURCE_GROUP> --name <CLUSTER_NAME> --query oidcIssuerProfile.issuerUrl --output tsv
    ```

1. Use this issuer in */etc/rancher/k3s/config.yaml*:

    ```yaml
    kube-apiserver-arg:
    - service-account-issuer=<SERVICE_ACCOUNT_ISSUER>
    - service-account-max-token-expiration=24h
    ```
    
1. Once you update the config.yaml, you need to restart K3s.

    ```bash
    systemctl restart k3s
    ```

## Create a User Assigned Managed Identity (UAMI)

You need to create a User Assigned Managed Identity (UAMI) to federate with your Kubernetes cluster. This is the identity in Azure Public Cloud that you grant role based access to your Storage Account or OneLake Lakehouse for use with Azure Container Storage Enabled by Azure Arc.

You can create one UAMI using Azure CLI with the following command:

```azurecli
az identity create --name <USER_ASSIGNED_IDENTITY_NAME> --resource-group <RESOURCE_GROUP> --location <LOCATION> --subscription <SUBSCRIPTION>
```

This will return a client id that you will use later to federate the credential and configure Azure Container Storage.

## Federate the User Assigned Managed Identity (UAMI)

1. Add the UAMI to your EdgeStorageConfiguration Custom Resource Definition (CRD) for Azure Container Storage:

    ```bash
    kubectl edit edgestorageconfiguration edge-storage-configuration
    ```

1. Add the following YAML in the `spec` section after the `serviceMesh`:

    ```yaml
    workloadIdentity:
      clientID: <CLIENT_ID_FROM_UAMI>
      tenantID: <TENANT_ID_FROM_UAMI>
    ```

1. Next, federate your UAMI with the three required service accounts in the `azure-arc-containerstorage` namespace by editing and running the following Azure CLI commands:

    ```azurecli
    az identity federated-credential create --name acsa-csi-fed --identity-name <USER_ASSIGNED_IDENTITY_NAME> --resource-group <RESOURCE_GROUP> --issuer <OIDC_ISSUER> --subject system:serviceaccount:azure-arc-containerstorage:csi-wyvern-sa --audience api://AzureADTokenExchange
    ```
    
    ```azurecli
    az identity federated-credential create --name acsa-pv-fed --identity-name <USER_ASSIGNED_IDENTITY_NAME> --resource-group <RESOURCE_GROUP> --issuer <OIDC_ISSUER> --subject system:serviceaccount:azure-arc-containerstorage:wyvern-pv-sa --audience api://AzureADTokenExchange
    ```
    
    ```azurecli
    az identity federated-credential create --name acsa-operator-fed --identity-name <USER_ASSIGNED_IDENTITY_NAME> --resource-group <RESOURCE_GROUP> --issuer <OIDC_ISSUER> --subject system:serviceaccount:azure-arc-containerstorage:wyvern-operator-sa --audience api://AzureADTokenExchange
    ```

1. You also need to add a role binding to the Storage Account you wish to use for the UAMI you created. In the portal, you may add a role for *Storage Blob Data Owner* for the UAMI created above:

    ```azurecli
    az role assignment create --assignee <USER_ASSIGNED_IDENTITY_ID> --role "Storage Blob Data Owner" --scope "/subscriptions/<SUBSCRIPTION>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGEACCOUNT>"
    ```

## Create a Cloud Ingest Persistent Volume Claim (PVC)

To create a Cloud Ingest Persistent Volume Claim (PVC) with Workload Identity, you can use the following example YAML file:

```yaml
apiVersion: "arccontainerstorage.azure.net/v1"
kind: EdgeSubvolume
metadata:
  name: acsa-pvc
spec:
  edgevolume: acsa-pvc
  path: exampleSubDir # If you change this path, line 33 in deploymentExample.yaml must be updated. Don't use a preceding slash.
  subvolumeType: INGEST
  auth:
    authType: WORKLOAD_IDENTITY
  storageaccountendpoint: ""
```

Then continue following the instructions in the [Create a Cloud Ingest Persistent Volume Claim (PVC)](cloud-ingest-edge-volume-configuration.md#create-a-cloud-ingest-persistent-volume-claim-pvc) section.

## Next step

[Monitor your deployment](monitor-deployment-edge-volumes.md)

