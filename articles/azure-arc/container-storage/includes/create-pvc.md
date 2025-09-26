---
ms.service: azure-arc
ms.subservice: azure-arc-container-storage
ms.topic: include
ms.date: 09/26/2025
author: asergaz
ms.author: sergaz
---
     
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

[!INCLUDE [lowercase-note](lowercase-note.md)]

* Edit the `metadata.name` value and create a name for your Persistent Volume Claim. This name is referenced on the last line of **deploymentExample.yaml** in the next step. 
* Edit the `metadata.namespace` value with your intended consuming pod. If you don't have an intended consuming pod, set it's value to `default`. 
* The `spec.resources.requests.storage` parameter determines the size of the persistent volume. It's 2 GB in this example, but can be modified to fit your needs.