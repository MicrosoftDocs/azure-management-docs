---
title: Configure cloud Mirror subvolumes    
description: Configure cloud Mirror subvolumes for Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.service: azure-arc
ms.topic: how-to
ms.date: 09/24/2025

#CustomerIntent: As a cloud administrator, I want to configure Cloud Mirror subvolumes so that I can enable syncing data from the cloud to the edge.
---

# Configure Cloud Mirror subvolumes

This article describes how to configure Cloud Mirror subvolumes (syncing data from the cloud to the edge) in Azure Container Storage enabled by Azure Arc. Conceptually, a Cloud Mirror subvolume is a location that will *mirror* data from a cloud destination to the edge as a read only copy. The frequency controls when a mirror sync occurs without direct user intervention, for instance, every hour, or once a day at a certain time. The OneShot functionality allows a user to perform a sync *right now* at a time of their choosing but having the system do a *one off* sync operation, after which it will return to its normal schedule per the frequency.


## Prerequisites

If your mirror root location is blob storage or ADLSgen2, continue following the prerequisites and instructions below. If your mirror root location is OneLake, follow the instructions in [Alternate: OneLake configuration for Cloud Ingest Edge Volumes](howto-configure-cloud-ingest-onelake.md) first.

[!INCLUDE [cloud-subvolumes-prerequisites](includes/cloud-subvolumes-prerequisites.md)]

## Configure Extension Identity

[!INCLUDE [cloud-subvolumes-configure-extension-identity](includes/cloud-subvolumes-configure-extension-identity.md)]

## Create a Cloud Mirror Persistent Volume Claim (PVC)

1. Create a file named ```cloudMirrorPVC.yaml``` with the following contents. Edit the ```metadata.name``` line and create a name for your Persistent Volume Claim. This name is referenced on the last line of ```deploymentExample.yaml``` in the next step. Also, update the ```metadata.namespace``` value with your intended consuming pod. If you don't have an intended consuming pod, the ```metadata.namespace``` value is ```default```. The ```spec.resources.requests.storage``` parameter determines the size of the persistent volume. It's 2 GB in this example, but can be modified to fit your needs:

**Note:** Use only lowercase letters and dashes. For more information, see the Kubernetes object naming documentation.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  ### Create a name for your PVC ###
  name: <create-persistent-volume-claim-name-here>
  ### Use a namespace that matched your intended consuming pod, or "default" ###
  namespace: <intended-consuming-pod-or-default-here>
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: cloud-backed-sc
```

2. To apply ```cloudMirrorPVC.yaml```, run:
```bash
kubectl apply -f "cloudMirrorPVC.yaml"
```

## Attach Mirror subvolume to the Edge Volume

For a reminder on the difference between an Edge Volume and a Subvolume, see [this article](https://learn.microsoft.com/en-us/azure/azure-arc/container-storage/volumes-subvolumes). 
To create a subvolume for Mirror using extension identity to connect to your storage account container, use the following process:

1. Get the name of the Edge Volume you created by running the following command. This will go into ```spec.edgevolume``` in the following step:
```bash
kubectl get edgevolumes
```
2. Create a file named ```mirrorSubvolume.yaml``` and copy the following contents. These variables must be updated with your information:

**Note:** Use only lowercase letters and dashes. For more information, see the Kubernetes object naming documentation.

* ```metadata.name```: Create a name for your subvolume.
* ```spec.edgevolume```: This name was retrieved from the previous step using ```kubectl get edgevolumes```.
* ```spec.path```: Create your own subdirectory name under the mount path. The following example already contains an example name (```exampleSubDir```). If you change this path name, line 33 in ```deploymentExample.yaml``` must be updated with the new path name. If you choose to rename the path, don't use a preceding slash.
* ```spec.authentication.authType```: This should be ```MANAGED_IDENTITY``` or ```WORKLOAD_IDENTITY```, depending on the authentication mechanism chosen.
* ```spec.blobAccount.accountEndpoint```: Navigate to your storage account in the Azure portal. On the Overview page, near the top right of the screen, select JSON View. You can find the ```storageaccountendpoint``` link under ```properties.primaryEndpoints.blob```. Copy the entire link; for example, https://mytest.blob.core.windows.net/.
* ```spec.blobAccount.containerName```: The container name in your storage account.
* ```spec.blobAccount.indexTagsMode```: ```NoIndexTags``` or ```MirrorIndexTags```. Mode ```MirrorIndexTags``` requires "Storage Blob Data Owner" permissions, and when set, index tags will be translated to corresponding ```azindex.<name> xattrs```. ```NoIndexTags``` only requires "Storage Blob Data Reader" permissions, and when set, it will keep index tag xattrs unset.
* ```spec.blobFiltering.blobNamePrefix```: Optional prefix to filter blobs. For example, "blobNamePrefix: a" will only mirror blobs with names starting with "a".
* ```spec.schedule.frequency```: Schedule for when mirroring should run. Options are: ```@annually```, ```@yearly```, ```@monthly```, ```@weekly```, ```@daily```, ```@hourly```, ```"never"```, or cron syntax (5 digits, first is minutes (0-59), second is hours (0-23), third is day (1-31), fourth is month (1-12), fifth is day of the week (0-6))
* ```spec.schedule.oneshot```: By default, left blank. If at any time a uuid is specified here, will trigger an 'immediate' sync. If a uuid is specified on creation, the subvolume will perform a sync on initial create, and thereafter according to the frequency. If this parameter is empty on creation, the subvolume will initially create without a sync, and will sync according to the frequency. Uuids can be generated at [uuidgenerator.net](https://www.uuidgenerator.net/version4) or with ```sed -i "s/oneshot: .*/oneshot: $(uuidgen)/" mirrorSubvolume.yaml```

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

3. To apply the ```mirrorSubvolume.yaml```, run:
```bash
kubectl apply -f "mirrorSubvolume.yaml"
```

## Attach your app (Kubernetes native application)

1. To configure a generic single pod (Kubernetes native application) against the PVC to use the Mirror capabilities, create a file named ```deploymentExample.yaml``` as follows. 
Modify the ```containers.name``` and ```volumes.persistentVolumeClaim.claimName``` values. If you updated the path name from ```cloudMirrorSubvolume.yaml```, ```exampleSubDir``` on line 33 must be updated with your new path name. The ```spec.replicas``` parameter determines the number of replica pods to create. It's 2 in this example, but can be modified to fit your needs:

**Note:** Use only lowercase letters and dashes. For more information, see the [Kubernetes object naming documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudmirrorsubvol-deployment ### This must be unique for each deployment you choose to create.
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
      containers:
        ### Specify the container in which to launch the busy box. ###
        - name: mirror-deployment-container ### this can be anything; a default name shared here
          image: mcr.microsoft.com/azure-cli:2.57.0@sha256:c7c8a97f2dec87539983f9ded34cd40397986dcbed23ddbb5964a18edae9cd09
          command:
            - "/bin/sh"
            - "-c"
            - "while true; do ls /data &>/dev/null || break; sleep 1; done"
          volumeMounts:
            ### This name must match the volumes.name attribute below ###
            - name: acsa-volume
              ### This mountPath is where the PVC is attached to the pod's filesystem ###
              mountPath: "/data"
      volumes:
         ### User-defined 'name' that's used to link the volumeMounts. This name must match volumeMounts.name as previously specified. ###
        - name: acsa-volume
          persistentVolumeClaim:
            ### This claimName must refer to your PVC metadata.name (Line 5)
            claimName: <your-pvc-metadata-name-from-line-5-of-pvc-yaml>
```
2. Then create the pod with:
```bash
kubectl apply -f deploymentexample.yaml
```
3. Use ```bash kubectl get pods``` to find the name of your pod. Copy this name to use in the next step.

**Note:** Because ```spec.replicas``` from ```deploymentExample.yaml``` was specified as 2, two pods appear using ```kubectl get pods```. You can choose either pod name to use for the next step.

4. Next, you can run the following command to exec into the pod (name copied from the last step): 
```bash
kubectl exec -it <name of pod> -- bash
```
5. Change directories into the ```/data``` mount path as specified from your ```deploymentExample.yaml```

6. You should see a directory with the name you specified as your path in Step 2 of the Attach subvolume to Edge Volume section, in this case, ```exampleSubDir```. Change to that directory, and run an ```ls``` to check for any contents mirrored here. If you specified a uuid in the ```oneshot``` field upon creation, and/or the specified ```frequency``` requirements have been satisfied, the subvolume should have performed a sync on initial create, and you should see data here mirrored from your specified storage account container. If either of those conditions is not met, this directory should be empty. 

## Check the status of the Cloud to Edge Mirror Volume Synchronization

### Overview

First, we will check the status of the ```mirrorSubvolume```.

Next, we will ensure that we have data in our Storage Account Container in Azure.

Finally, we can check the contents of the Mirror Subvolume again to ensure that the data has been properly Mirrored. 

### Check Mirror Subvolume

First, we can check the status of our Mirror subvolume. In particular, check that the BACKENDCONNECTION field is "Connected".
```bash
kubectl get mirrorsubvolumes
```

### Move some Data to your Storage Account Container

If you already have some data in your specified container, you can skip this step. If not, using Azure Portal or AZCopy, create some new data or move some existing data to your blob storage account container so that it can be mirrored on your edge cluster. 

### Trigger additional mirror

1. To make sure our new data is mirrored from the cloud to our subvolume, update the ```oneshot``` field in the file ```mirrorSubvolume.yaml``` to a new uuid.
Uuids can be generated at [uuidgenerator.net](https://www.uuidgenerator.net/version4) or with 

```bash
sed -i "s/oneshot: .*/oneshot: $(uuidgen)/" mirrorSubvolume.yaml
```

2. To apply the change and thus trigger the oneshot sync, use
```bash
kubectl apply -f mirrorSubvolume.yaml
```

### Check Mirror Subvolume Again

You can check on your sync by running the following command:
```bash
kubectl get mirrorsubvolumes
```
Once complete, we can check the contents of our Mirror subvolume. You can do that by connecting to the example pod we created by running:
```bash
kubectl exec -it <name of pod> -- bash
```
You should see the files from your Cloud Storage Account now Mirrored into this Subvolume. 

## Next Steps

If you have any issues or need help with configuration, or to give feedback, please contact: ACSA@microsoft.com