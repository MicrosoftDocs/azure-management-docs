---
title: Previous versions
description: Learn about previous versions of Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: concept-article
ms.date: 09/29/2025

#CustomerIntent: As a cloud administrator, I want to review the history of previous versions of Azure Container Storage enabled by Azure Arc.
---

# Previous versions

In releases 2.7 and earlier, there was an **edgeSubvolume** CRD. Documentation for that configuration can be found here. 

In release 2.8.2 and going forward, this **edgeSubvolume** CRD has been replaced by the new **ingestSubvolume** and **mirrorSubvolume** CRDs. These two should be used going forward. 

For users still using 2.7 and earlier, the following documentation can be used for the **edgeSubvolume** configuration:

## Attach subvolume to Edge Volume

To create a subvolume using extension identity to connect to your storage account container, use the following process:

1. Get the name of your Ingest Edge Volume using the following command:

    ```bash
    kubectl get edgevolumes
    ```

1. Create a file named `edgeSubvolume.yaml` and copy the following contents. These variables must be updated with your information:

   [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

   - `metadata.name`: Create a name for your subvolume.
   - `spec.edgevolume`: This name was retrieved from the previous step using `kubectl get edgevolumes`.
   - `spec.path`: Create your own subdirectory name under the mount path. The following example already contains an example name (`exampleSubDir`). If you change this path name, line 33 in `deploymentExample.yaml` must be updated with the new path name. If you choose to rename the path, don't use a preceding slash.
   - `spec.container`: The container name in your storage account.
   - `spec.storageaccountendpoint`: Navigate to your storage account in the Azure portal. On the **Overview** page, near the top right of the screen, select **JSON View**. You can find the `storageaccountendpoint` link under **properties.primaryEndpoints.blob**. Copy the entire link; for example, `https://mytest.blob.core.windows.net/`.

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

