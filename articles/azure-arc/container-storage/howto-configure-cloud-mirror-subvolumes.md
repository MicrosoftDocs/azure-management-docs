---
title: Configure Cloud Mirror subvolumes (preview)
description: Configure Cloud Mirror subvolumes for Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.service: azure-arc
ms.topic: how-to
ms.date: 09/24/2025

#CustomerIntent: As a cloud administrator, I want to configure Cloud Mirror subvolumes so that I can enable syncing data from the cloud to the edge.
---

# Configure Cloud Mirror subvolumes (preview)

This article describes how to configure Cloud Mirror subvolumes (syncing data from the cloud to the edge) in Azure Container Storage enabled by Azure Arc. Conceptually, a Cloud Mirror subvolume is a location that will *mirror* data from a cloud destination to the edge as a read only copy. The frequency controls when a mirror sync occurs without direct user intervention, for instance, every hour, or once a day at a certain time. The OneShot functionality allows a user to perform a sync *right now* at a time of their choosing but having the system do a *one off* sync operation, after which it will return to its normal schedule per the frequency.

> [!IMPORTANT]
> Cloud Mirror subvolumes are currently in preview. This functionality is not recommended for production workloads. If you have any issues or need help with configuration, or to give feedback, please contact: ACSA@microsoft.com


## Prerequisites

If your mirror root location is blob storage or ADLSgen2, continue following the prerequisites and instructions below. If your mirror root location is OneLake, follow the instructions in [Alternate: OneLake configuration for Cloud Ingest Edge Volumes](howto-configure-cloud-ingest-onelake.md) first.

[!INCLUDE [cloud-subvolumes-prerequisites](includes/cloud-subvolumes-prerequisites.md)]

## Configure Extension Identity

Edge Volumes allows the use of a system-assigned extension identity for access to blob storage. This section describes how to use the system-assigned extension identity to grant access to your storage account, allowing data to be mirrored from these locations to your edge location.

[!INCLUDE [cloud-subvolumes-configure-extension-identity](includes/cloud-subvolumes-configure-extension-identity.md)]

## Create a Cloud Mirror Persistent Volume Claim (PVC)

To create a PVC for your Mirror subvolume, use the following process:

1. Create a file named `cloudMirrorPVC.yaml` with the following content:

  [!INCLUDE [create-pvc](includes/create-pvc.md)]

1. To apply the **cloudMirrorPVC.yaml**, run:

  ```bash
  kubectl apply -f "cloudMirrorPVC.yaml"
  ```
  
## Attach Mirror subvolume to the Edge Volume

To create a subvolume for Mirror, using extension identity to connect to your storage account container, use the following process:

1. Get the name of the Edge Volume you created by running the following command. This will go into `spec.edgevolume` in the following step:

  ```bash
  kubectl get edgevolumes
  ```
1. Create a file named `mirrorSubvolume.yaml` with the following content:

  ```yaml
  apiVersion: "arccontainerstorage.azure.net/v1"
  kind: MirrorSubvolume
  metadata:
    name: <create-a-subvolume-name-here>
  spec:
    edgevolume: <your-edge-volume-name-here>
    path: mirrorSubDir # Don't use a preceding slash
    authentication:
    authType: MANAGED_IDENTITY
    blobAccount:
    accountEndpoint: "https://<STORAGE ACCOUNT NAME>.blob.core.windows.net/"
    containerName: <your-blob-storage-account-container-name>
    indexTagsMode: NoIndexTags
    blobFiltering:
    blobNamePrefix:
    schedule:
    frequency: "@hourly"
    oneshot:
  ```
  
  [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

  - `metadata.name`: Create a name for your subvolume.
  - `spec.edgevolume`: This name was retrieved from the previous step.
  - `spec.path`: Create your own subdirectory name under the mount path. The default name is `mirrorSubDir`.
  - `spec.authentication.authType`: This should be `MANAGED_IDENTITY` or `WORKLOAD_IDENTITY`, depending on the authentication mechanism chosen.
  - `spec.blobAccount.accountEndpoint`: Navigate to your storage account in the Azure portal. On the Overview page, near the top right of the screen, select JSON View. You can find the link under *properties.primaryEndpoints.blob*. Copy the entire link.
  - `spec.blobAccount.containerName`: The container name in your storage account.
  - `spec.blobAccount.indexTagsMode`: `NoIndexTags` or `MirrorIndexTags`. `MirrorIndexTags` requires "Storage Blob Data Owner" permissions, and when set, index tags will be translated to corresponding `azindex.<name> xattrs`. `NoIndexTags` only requires "Storage Blob Data Reader" permissions, and when set, it will keep index tag xattrs unset.
  - `spec.blobFiltering.blobNamePrefix`: Optional prefix to filter blobs. For example, if the value is `blobNamePrefix: a` it will only mirror blobs with names starting with "a".
  - `spec.schedule.frequency`: Schedule for when mirroring should run. Options are: `@annually`, `@yearly`, `@monthly`, `@weekly`, `@daily`, `@hourly`, `"never"`, or cron syntax (5 digits, first is minutes (0-59), second is hours (0-23), third is day (1-31), fourth is month (1-12), fifth is day of the week (0-6))
  - `spec.schedule.oneshot`: By default, left blank. If at any time a uuid is specified here, will trigger an 'immediate' sync. If a uuid is specified on creation, the subvolume will perform a sync on initial create, and thereafter according to the frequency. If this parameter is empty on creation, the subvolume will initially create without a sync, and will sync according to the frequency. Uuids can be generated at [uuidgenerator.net](https://www.uuidgenerator.net/version4) or with `sed -i "s/oneshot: .*/oneshot: $(uuidgen)/" mirrorSubvolume.yaml`


1. To apply the **mirrorSubvolume.yaml**, run:

  ```bash
  kubectl apply -f "mirrorSubvolume.yaml"
  ```
  
## Attach your app (Kubernetes native application)

To configure a generic single pod (Kubernetes native application) against the PVC to use the Mirror capabilities, use the following process:

1. Create a file named `deploymentExample.yaml` with the following content:

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    ### This must be unique for each deployment you choose to create. ###
    name: cloudmirrorsubvol-deployment
  spec:
    replicas: 2
    selector:
    matchLabels:
      name: acsa-testclientdeployment
    template:
    metadata:
      name: acsa-testclientdeployment
      labels:
      name: acsa-testclientdeployment
    spec:
      affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
          matchExpressions:
          - key: app
          operator: In
          values:
          - acsa-testclientdeployment
        topologyKey: kubernetes.io/hostname
      ### Specify the container in which to launch the busy box ###
      containers:
      ### This name can be anything; default name shared here ###
      - name: mirror-deployment-container
        image: mcr.microsoft.com/azure-cli:2.57.0@sha256:c7c8a97f2dec87539983f9ded34cd40397986dcbed23ddbb5964a18edae9cd09
        command:
        - "/bin/sh"
        - "-c"
        - "while true; do ls /data/mirrorSubDir &>/dev/null || break; sleep 1; done"
        volumeMounts:
        ### This name must match the volumes.name attribute below ###
        - name: acsa-volume
          ### This mountPath is where the PVC is attached to the pod's filesystem ###
          mountPath: "/data"
      volumes:
      ### User-defined 'name' that's used to link the volumeMounts ###
      - name: acsa-volume
        persistentVolumeClaim:
        ### This claimName must refer to your PVC metadata.name (line 5 in cloudMirrorPVC.yaml) ###
        claimName: <your-pvc-metadata-name-from-line-5-of-pvc-yaml>
  ```

  [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

  - Edit the `containers.name` and `volumes.persistentVolumeClaim.claimName` values.
  - If you edited the `spec.path` value in **mirrorSubvolume.yaml**, the value `mirrorSubDir` on this file must be updated with your new path name.
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
  > Because `spec.replicas` from **deploymentExample.yaml** was specified with 2, two pods will

1. Run the following command to start exec into the pod. Replace `<name-of-pod>` with your pod name from the previous step:

    ```bash
    kubectl exec -it <name-of-pod> -- sh
    ```

1. Change directories into the `/data` mount path as specified from your **deploymentExample.yaml** file:

    ```bash
    cd /data
    ```

1. You should see a directory that matches the value you set for `spec.path`in **mirrorSubvolume.yaml**. If you used the default value it's name is *mirrorSubDir*. Change to that subdirectory, and check for any contents mirrored there:

    ```bash
    cd mirrorSubDir
    ls
    ```

  If you specified a uuid in the `spec.schedule.oneshot` field upon creation, and/or the specified `spec.schedule.frequency` requirements have been satisfied, the subvolume should have performed a sync on initial create, and you should see data here mirrored from your specified storage account container. If either of those conditions is not met, this directory should be empty.

## Check the status of the Cloud Mirror subvolume Synchronization

First, check the status of the `mirrorSubvolume`. Next, ensure that you have data in your Storage Account Container in Azure. Finally, check the contents of the Mirror subvolume again to ensure that the data has been properly Mirrored.

### Check Mirror subvolume Status

Check the status of our Mirror subvolume, in particular, check that the **BACKENDCONNECTION** field is *Connected*:

  ```bash
  kubectl get mirrorsubvolumes
  ```

### Add data to your Storage Account Container

If you don't already have data in your specified container, use one of the following methods to upload files:

- **Azure Portal:**  
  [Upload blobs using Azure Portal](https://learn.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal)

- **AZCopy:**  
  [Upload blobs using AZCopy](https://learn.microsoft.com/azure/storage/common/storage-use-azcopy-blobs#upload-files)

Add or move files to your blob storage account container so they can be mirrored on your edge cluster.

### Trigger additional mirror

1. To make sure the new data is mirrored from the cloud to the subvolume, update the `spec.schedule.oneshot` field in the file **mirrorSubvolume.yaml** to a new uuid. Uuids can be generated at [uuidgenerator.net](https://www.uuidgenerator.net/version4) or with the following command:

    ```bash
    sed -i "s/oneshot: .*/oneshot: $(uuidgen)/" mirrorSubvolume.yaml
    ```

1. To apply the change and thus trigger the *oneshot* sync, run:

    ```bash
    kubectl apply -f "mirrorSubvolume.yaml"
    ```

### Check Mirror subvolume again

1. Check the status of our Mirror subvolume again to ensure the **BACKENDCONNECTION** field is still *Connected*:

    ```bash
    kubectl get mirrorsubvolumes
    ```
1. Once complete, we can check the contents of our Mirror subvolume. You can do that by connecting to the example pod we created by running:

    ```bash
    kubectl exec -it <name-of-pod> -- sh
    ```

    You should now see the files from your Cloud Storage Account mirrored into this subvolume.

## Next Steps

- To learn how to configure Cloud Ingest subvolumes, see [Configure Cloud Ingest subvolumes](howto-configure-cloud-ingest-subvolumes.md).
- To learn how to use Edge Volumes together, see [Using Edge Volumes together](storage-options.md).