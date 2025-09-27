---
title: Configure Cloud Ingest subvolumes
description: Configure Cloud Ingest subvolumes for Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 09/27/2025

# Customer intent: As a cloud administrator, I want to configure Cloud Ingest subvolumes, so that I can upload files to blob storage and purge them locally.
---

# Configure Cloud Ingest subvolumes

This article describes how to configure Cloud Ingest subvolumes (blob upload with local purge) in Azure Container Storage enabled by Azure Arc. A Cloud Ingest subvolume facilitates limitless data ingestion from edge to blob, including ADLSgen2. Files written to this storage type are seamlessly transferred to blob storage and once confirmed uploaded, are then purged locally. This removal ensures space availability for new data. Moreover, this storage option supports data integrity in disconnected environments, which enables local storage and synchronization upon reconnection to the network.

For example, you can write a file to your cloud ingest Persistent Volume Claim (PVC), and a process runs a scan to check for new files every minute. Once identified, the file is sent for uploading to your designated blob destination. Following confirmation of a successful upload, Cloud Ingest subvolume waits for five minutes, and then deletes the local version of your file.

## Prerequisites

If your final destination is blob storage or ADLSgen2, continue following the prerequisites and instructions in this article. If your final destination is OneLake, follow the instructions in [Configure OneLake Identity for Cloud subvolumes](howto-configure-onelake-identity).

[!INCLUDE [cloud-subvolumes-prerequisites](includes/cloud-subvolumes-prerequisites.md)]

## Configure Extension Identity

Edge Volumes allows the use of a system-assigned extension identity for access to blob storage. This section describes how to use the system-assigned extension identity to grant access to your storage account, allowing you to upload Cloud Ingest subvolumes to these storage systems.

[!INCLUDE [cloud-subvolumes-configure-extension-identity](includes/cloud-subvolumes-configure-extension-identity.md)]

## Create a Cloud Ingest Persistent Volume Claim (PVC)

To create a PVC for your Ingest subvolume, use the following process:

1. Create a file named `cloudIngestPVC.yaml` with the following content:

    [!INCLUDE [create-pvc](includes/create-pvc.md)]

1. To apply the **cloudIngestPVC.yaml**, run:

    ```bash
    kubectl apply -f "cloudIngestPVC.yaml"
    ```

## Attach Ingest subvolume to Edge Volume

To create a subvolume for Ingest, using extension identity to connect to your storage account container, use the following process:

1. Get the name of the Edge Volume you created by running the following command:

    ```bash
    kubectl get edgevolumes
    ```

1. Create a file named `ingestSubvolume.yaml` with the following content:

    ```yaml
    apiVersion: "arccontainerstorage.azure.net/v1"
    kind: IngestSubvolume
    metadata:
      name: <create-a-subvolume-name-here>
    spec:
      edgevolume: <your-edge-volume-name-here>
      path: ingestSubDir # Don't use a preceding slash
      authentication:
        authType: MANAGED_IDENTITY
      storageAccountEndpoint: "https://<STORAGE ACCOUNT NAME>.blob.core.windows.net/"
      containerName: <your-blob-storage-account-container-name>
      ingest:
        order: newest-first
        minDelaySec: 60
      eviction:
        order: unordered
        minDelaySec: 120
      onDelete: trigger-immediate-ingest
    ```
   
    [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

    - `metadata.name`: Create a name for your subvolume.
    - `spec.edgevolume`: This name was retrieved from the previous step.
    - `spec.path`: Create your own subdirectory name under the mount path. The default name is `ingestSubDir`.
    - `spec.authentication.authType`: This should be `MANAGED_IDENTITY` or `WORKLOAD_IDENTITY`, depending on the authentication mechanism chosen.
    - `spec.storageAccountEndpoint`: Navigate to your storage account in the Azure portal. On the Overview page, near the top right of the screen, select JSON View. You can find the link under *properties.primaryEndpoints.blob*. Copy the entire link.
    - `spec.containerName`: The container name in your storage account.
    
    The following variables have reasonable defaults, but can be changed:

    - `spec.ingest.order`: The order in which dirty files are uploaded. This is a best effort, not a guarantee. Options for order are: `oldest-first` or `newest-first`.
    - `spec.ingest.minDelaySec`: The minimum number of seconds before a dirty file is eligible for ingest. This number can range between 0 and 31536000 (a year in seconds).
    - `spec.eviction.order`: How files are evicted once they are uploaded to the cloud. Options for eviction order are: `unordered` or `never`.
    - `spec.eviction.minDelaySec`: The number of seconds before a clean file is eligible for eviction. This number can range between 0 and 31536000 (a year in seconds).
    - `spec.onDelete`: The action to take on this IngestSubVolume if/when it's requested to be deleted. Options are `trigger-immediate-ingest` which will immediately mark all files as eligible for ingest and attempt to ingest them, or `abandon` which will abandon all data in this ingest subvolume and delete the subvolume.

    > [!NOTE]
    > If you choose **abandon** for your `spec.onDelete` value, any dirty data in your subvolume will be lost. Please be careful and mindful before choosing this as an option.

1. To apply the **ingestSubvolume.yaml**, run:

   ```bash
   kubectl apply -f "ingestSubvolume.yaml"
   ```

## Attach your app (Kubernetes native application)

To configure a generic single pod (Kubernetes native application) against the PVC to use the Ingest capabilities, use the following process:

1. Create a file named `deploymentExample.yaml` with the following content:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     ### This must be unique for each deployment you choose to create. ###
     name: cloudingestedgesubvol-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         name: wyvern-testclientdeployment
     template:
       metadata:
         name: wyvern-testclientdeployment
         labels:
           name: wyvern-testclientdeployment
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
             - labelSelector:
                 matchExpressions:
                 - key: app
                   operator: In
                   values:
                   - wyvern-testclientdeployment
               topologyKey: kubernetes.io/hostname
         ### Specify the container in which to launch the busy box. ###
         containers:
           ### This name can be anything; default name shared here ###
           - name: ingest-deployment-container
             image: mcr.microsoft.com/azure-cli:2.57.0@sha256:c7c8a97f2dec87539983f9ded34cd40397986dcbed23ddbb5964a18edae9cd09
             command:
               - "/bin/sh"
               - "-c"
               - "dd if=/dev/urandom of=/data/ingestSubDir/acsaingesttestfile count=16 bs=1M && while true; do ls /data &>/dev/null || break; sleep 1; done"
             volumeMounts:
               ### This name must match the volumes.name attribute below ###
               - name: wyvern-volume
                 ### This mountPath is where the PVC is attached to the pod's filesystem ###
                 mountPath: "/data"
         volumes:
            ### User-defined 'name' that's used to link the volumeMounts ###
           - name: wyvern-volume
             persistentVolumeClaim:
               ### This claimName must refer to your PVC metadata.name (Line 5 in cloudIngestPVC.yaml) ###
               claimName: <your-pvc-metadata-name-from-line-5-of-pvc-yaml>
   ```

   [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

  - Edit the `containers.name` and `volumes.persistentVolumeClaim.claimName` values.
  - If you edited the `spec.path` value in **edgeSubvolume.yaml**, the value `ingestSubDir` on this file must be updated with your new path name.
  - The `spec.replicas` parameter determines the number of replica pods to create. It's 2 in this example, but can be modified to fit your needs.

1. To apply the **deploymentExample.yaml** and create the pod, run:

   ```bash
   kubectl apply -f "deploymentExample.yaml"
   ```

1. Find the name of your pod to use in the next step:

  ```bash
  kubectl get pods
  ```

  > [!NOTE]
  > Because `spec.replicas` from **deploymentExample.yaml** was specified with 2, two pods are created. You can use either pod name for the next step.

1. Run the following command to start exec into the pod. Replace `<name-of-pod>` with your pod name from the previous step:

    ```bash
    kubectl exec -it <name-of-pod> -- sh
    ```

1. Change directories into the `/data` mount path as specified from your **deploymentExample.yaml** file:

    ```bash
    cd /data
    ```

1. You should see a directory that matches the value you set for `spec.path` in **ingestSubvolume.yaml**. If you used the default values, its name is *ingestSubDir*. Change to that subdirectory:

    ```bash
    cd ingestSubDir
    ```

1. As an example, create a file named `file1.txt` and write to it:

    ```bash
    echo "Hello World" > file1.txt
    ```
   
   This file will be uploaded to your blob storage account container, and then purged locally after five minutes.

1. In the Azure portal, navigate to your storage account and find the container that matches the value you set for `spec.containerName` in **ingestSubvolume.yaml**. You should find `file1.txt` populated within the container. If the file is not there yet, wait approximately 1 minute; Edge Volumes waits a minute before uploading.

## Next steps

- To learn how to configure Cloud Mirror subvolumes, see [Configure Cloud Mirror subvolumes](howto-configure-cloud-mirror-subvolumes.md).
- To learn how to use Edge Volumes together, see [Using Edge Volumes together](storage-options.md).