---
title: Cloud Ingest Edge Volumes configuration
description: Learn about Cloud Ingest Edge Volumes configuration for Edge Volumes.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.custom:
  - linux-related-content
  - build-2025
ms.date: 03/12/2025
# Customer intent: "As a Kubernetes administrator, I want to configure Cloud Ingest Edge Volumes for my applications, so that I can efficiently manage data ingestion and ensure local purging of files while maintaining data integrity in disconnected environments."
---


# Cloud Ingest Edge Volumes configuration

This article describes the configuration for *Cloud Ingest Edge Volumes* (blob upload with local purge).

## What is Cloud Ingest Edge Volumes?

*Cloud Ingest Edge Volumes* facilitates limitless data ingestion from edge to blob, including ADLSgen2. Files written to this storage type are seamlessly transferred to blob storage and once confirmed uploaded, are then purged locally. This removal ensures space availability for new data. Moreover, this storage option supports data integrity in disconnected environments, which enables local storage and synchronization upon reconnection to the network.

For example, you can write a file to your cloud ingest PVC, and a process runs a scan to check for new files every minute. Once identified, the file is sent for uploading to your designated blob destination. Following confirmation of a successful upload, Cloud Ingest Edge Volume waits for five minutes, and then deletes the local version of your file.

## Prerequisites

If your final destination is blob storage or ADLSgen2, continue following the prerequisites and instructions below. If your final destination is OneLake, follow the instructions in [Alternate: OneLake configuration for Cloud Ingest Edge Volumes](howto-configure-cloud-ingest-onelake.md).

[!INCLUDE [cloud-subvolumes-prerequisites](includes/cloud-subvolumes-prerequisites.md)]

## Configure Extension Identity

Edge Volumes allows the use of a system-assigned extension identity for access to blob storage. This section describes how to use the system-assigned extension identity to grant access to your storage account, allowing you to upload Cloud Ingest subvolumes to these storage systems.

[!INCLUDE [cloud-subvolumes-configure-extension-identity](includes/cloud-subvolumes-configure-extension-identity.md)]

## Create a Cloud Ingest Persistent Volume Claim (PVC)

To create a PVC for your Ingest subvolume, use the following process:

1. Create a file named `cloudIngestPVC.yaml` with the following content:

    [!INCLUDE [create-pvc](includes/create-pvc.md)]

1. To apply `cloudIngestPVC.yaml`, run:

    ```bash
    kubectl apply -f "cloudIngestPVC.yaml"
    ```

## Attach Ingest subvolume to Edge Volume

To create a subvolume for Ingest, using extension identity to connect to your storage account container, use the following process:

1. Get the name of the Edge Volume you created by running the following command. This will go into `spec.edgevolume` in the following step:

    ```bash
    kubectl get edgevolumes
    ```

1. Create a file named `edgeSubvolume.yaml` and copy the following contents. These variables must be updated with your information:

    ```yaml
    apiVersion: "arccontainerstorage.azure.net/v1"
    kind: EdgeSubvolume
    metadata:
      name: <create-a-subvolume-name-here>
    spec:
      edgevolume: <your-edge-volume-name-here>
      path: exampleSubDir # If you change this path, line 33 in deploymentExample.yaml must be updated. Don't use a preceding slash.
      subvolumeType: INGEST 
      auth:
        authType: MANAGED_IDENTITY
      storageaccountendpoint: "https://<STORAGE ACCOUNT NAME>.blob.core.windows.net/"
      container: <your-blob-storage-account-container-name>
      ingestPolicy: edgeingestpolicy-default # Optional: See the following instructions if you want to update the ingestPolicy with your own configuration
    ```
   
    [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

   - `metadata.name`: Create a name for your subvolume.
   - `spec.edgevolume`: This name was retrieved from the previous step using `kubectl get edgevolumes`.
   - `spec.path`: Create your own subdirectory name under the mount path. The following example already contains an example name (`exampleSubDir`). If you change this path name, line 33 in `deploymentExample.yaml` must be updated with the new path name. If you choose to rename the path, don't use a preceding slash.
   - `spec.container`: The container name in your storage account.
   - `spec.storageaccountendpoint`: Navigate to your storage account in the Azure portal. On the **Overview** page, near the top right of the screen, select **JSON View**. You can find the `storageaccountendpoint` link under **properties.primaryEndpoints.blob**. Copy the entire link; for example, `https://mytest.blob.core.windows.net/`.

2. To apply `edgeSubvolume.yaml`, run:

   ```bash
   kubectl apply -f "edgeSubvolume.yaml"
   ```

### Optional: Modify the `ingestPolicy` from the default

1. If you want to change the `ingestPolicy` from the default `edgeingestpolicy-default`, create a file named `myedgeingest-policy.yaml` with the following contents. The following variables must be updated with your preferences:

   [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

   - `metadata.name`: Create a name for your **ingestPolicy**. This name must be updated and referenced in the `spec.ingestPolicy` section of your `edgeSubvolume.yaml`.
   - `spec.ingest.order`: The order in which dirty files are uploaded. This is best effort, not a guarantee (defaults to **oldest-first**). Options for order are: **oldest-first** or **newest-first**.
   - `spec.ingest.minDelaySec`: The minimum number of seconds before a dirty file is eligible for ingest (defaults to 60). This number can range between 0 and 31536000.
   - `spec.eviction.order`: How files are evicted (defaults to **unordered**). Options for eviction order are: **unordered** or **never**.
   - `spec.eviction.minDelaySec`: The number of seconds before a clean file is eligible for eviction (defaults to 300). This number can range between 0 and 31536000.

   ```yaml
   apiVersion: arccontainerstorage.azure.net/v1
   kind: EdgeIngestPolicy
   metadata:
     name: <create-a-policy-name-here> # This must be updated and referenced in the spec.ingestPolicy section of the edgeSubvolume.yaml
   spec:
     ingest:
       order: <your-ingest-order>
       minDelaySec: <your-min-delay-sec>
     eviction:
       order: <your-eviction-order>
       minDelaySec: <your-min-delay-sec>
   ```

1. To apply `myedgeingest-policy.yaml`, run:

   ```bash
   kubectl apply -f "myedgeingest-policy.yaml"
   ```

## Attach your app (Kubernetes native application)

1. To configure a generic single pod (Kubernetes native application) against the Persistent Volume Claim (PVC), create a file named `deploymentExample.yaml` with the following contents. Modify the `containers.name` and `volumes.persistentVolumeClaim.claimName` values. If you updated the path name from `edgeSubvolume.yaml`, `exampleSubDir` on line 33 must be updated with your new path name. The `spec.replicas` parameter determines the number of replica pods to create. It's 2 in this example, but can be modified to fit your needs:

   [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

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
               - "dd if=/dev/urandom of=/data/exampleSubDir/acsaingesttestfile count=16 bs=1M && while true; do ls /data &>/dev/null || break; sleep 1; done"
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

1. To apply `deploymentExample.yaml`, run:

   ```bash
   kubectl apply -f "deploymentExample.yaml"
   ```

1. Use `kubectl get pods` to find the name of your pod. Copy this name to use in the next step.

   > [!NOTE]
   > Because `spec.replicas` from `deploymentExample.yaml` was specified as `2`, two pods appear using `kubectl get pods`. You can choose either pod name to use for the next step.

1. Run the following command and replace `POD_NAME_HERE` with your copied value from the last step:

   ```bash
   kubectl exec -it POD_NAME_HERE -- sh
   ```

1. Change directories into the `/data` mount path as specified from your `deploymentExample.yaml`.

1. You should see a directory with the name you specified as your `path` in Step 2 of the [Attach subvolume to Edge Volume](#attach-subvolume-to-edge-volume) section. Change directories into `/YOUR_PATH_NAME_HERE`, replacing the `YOUR_PATH_NAME_HERE` value with your details.

1. As an example, create a file named `file1.txt` and write to it using `echo "Hello World" > file1.txt`.

1. In the Azure portal, navigate to your storage account and find the container specified from Step 2 of [Attach subvolume to Edge Volume](#attach-subvolume-to-edge-volume). When you select your container, you should find `file1.txt` populated within the container. If the file hasn't appeared yet, wait approximately 1 minute; Edge Volumes waits a minute before uploading.

## Next steps

After you complete these steps, you can begin monitoring your deployment using Azure Monitor and Kubernetes Monitoring or 3rd-party monitoring with Prometheus and Grafana.

[Monitor your deployment](howto-azure-monitor-kubernetes.md)
