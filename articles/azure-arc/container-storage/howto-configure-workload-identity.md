---
title: Configure Workload Identity for Cloud subvolumes
description: Learn how to configure Workload Identity for Cloud subvolumes in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 09/27/2025

#CustomerIntent: As a Kubernetes administrator, I want to configure Cloud Ingest Edge Volumes with Workload Identity.
---

# Configure Workload Identity for Cloud subvolumes

This article describes how to configure Workload Identity to be used for a subvolume configuration. This applies to either Cloud Ingest subvolumes or Cloud Mirror subvolumes.

## Configure Kubernetes cluster for Workload Identity

In order to use Workload Identity with Azure Container Storage enabled by Azure Arc, you must first enable the features for your Azure Arc Connected Kubernetes Cluster. Enable the OpenID Connect (OIDC) issuer and workload identity by running the following command:

```azurecli
az connectedk8s update --resource-group <RESOURCE_GROUP> --name <CLUSTER_NAME> --enable-oidc-issuer --enable-workload-identity
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

You need to create a UAMI to federate with your Kubernetes cluster and grant role based access to your Storage Account or OneLake Lakehouse, for use with Azure Container Storage enabled by Azure Arc.

You can create a UAMI using Azure CLI with the following command:

```azurecli
az identity create --resource-group <RESOURCE_GROUP> --name <USER_ASSIGNED_IDENTITY_NAME> --location <LOCATION> --subscription <SUBSCRIPTION>
```

This command returns a client ID that you use later to federate the credential and configure Azure Container Storage.

## Federate the UAMI

1. Add the UAMI to your *EdgeStorageConfiguration* Custom Resource Definition (CRD) for Azure Container Storage:

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
    az identity federated-credential create --resource-group <RESOURCE_GROUP> --identity-name <USER_ASSIGNED_IDENTITY_NAME> --name acsa-csi-fed --issuer <OIDC_ISSUER> --subject system:serviceaccount:azure-arc-containerstorage:csi-wyvern-sa --audience api://AzureADTokenExchange
    ```
    
    ```azurecli
    az identity federated-credential create --resource-group <RESOURCE_GROUP> --identity-name <USER_ASSIGNED_IDENTITY_NAME> --name acsa-pv-fed --issuer <OIDC_ISSUER> --subject system:serviceaccount:azure-arc-containerstorage:wyvern-pv-sa --audience api://AzureADTokenExchange
    ```
    
    ```azurecli
    az identity federated-credential create --resource-group <RESOURCE_GROUP> --identity-name <USER_ASSIGNED_IDENTITY_NAME> --name acsa-operator-fed --issuer <OIDC_ISSUER> --subject system:serviceaccount:azure-arc-containerstorage:wyvern-operator-sa --audience api://AzureADTokenExchange
    ```

1. You also need to add a role binding to the Storage Account you wish to use for the UAMI you created. Add the role *Storage Blob Data Owner* to the UAMI you created:

    ```azurecli
    az role assignment create --assignee <USER_ASSIGNED_IDENTITY_ID> --role "Storage Blob Data Owner" --scope "/subscriptions/<SUBSCRIPTION>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGEACCOUNT>"
    ```

## Create a Persistent Volume Claim (PVC) with Workload Identity

When creating the CRD for Ingest or Mirror subvolumes, set the `spec.authentication.authType` parameter to be `WORKLOAD_IDENTITY`.

## Next step

Continue the steps in either [Create a Cloud Ingest Persistent Volume Claim (PVC)](howto-configure-cloud-ingest-subvolumes.md#create-a-cloud-ingest-persistent-volume-claim-pvc) or [Create a Cloud Mirror Persistent Volume Claim (PVC)](howto-configure-cloud-mirror-subvolumes.md#create-a-cloud-mirror-persistent-volume-claim-pvc) to complete the configuration.

