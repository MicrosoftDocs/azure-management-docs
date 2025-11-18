---
title: Configure a Local Shared Edge Volume (preview)
description: In this tutorial, you learn how to configure a Local Shared Edge Volume (preview).
ms.topic: tutorial
ms.date: 11/11/2024
ms.author: sethm 
author: sethmanheim


# Customer intent: "As a Kubernetes administrator, I want to configure a Local Shared Edge Volume so that I can manage read-write-many storage effectively for applications running in edge environments."
---

# Tutorial: Configure a Local Shared Edge Volume (preview)

In this tutorial, you learn how to configure a Local Shared Edge Volume. This volume type offers read-write-many storage, local to your Kubernetes cluster. This shared storage type remains independent of cloud infrastructure, making it ideal for scratch space, temporary storage, and locally persistent data unsuitable for cloud destinations. It's also a good destination for data that's actively being worked on, changed, or processed at the edge.

The tutorial covers the following tasks:

> [!div class="checklist"]
> * Create a Local Shared Edge Volume PVC
> * Create an example deployment
> * Connect to your pod
> * Write a sample file

## Prerequisites

- The tutorial assumes that you already have an Arc-enabled Kubernetes cluster. To connect an existing Kubernetes cluster to Azure Arc, see [Connect an existing Kubernetes cluster to Azure Arc](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli).
- You should have already installed the Azure Container Storage extension. If you haven't, see [Install Edge Volumes](howto-install-edge-volumes.md).

## Create a Local Shared Edge Volume PVC

This section creates a Persistent Volume Claim, or PVC, for your Local Shared Volume. You specify how much space you want to provision for this volume.

Create a file named `localSharedPVC.yaml` with the following contents. Modify the `metadata.name` value with a name for your Persistent Volume Claim. Then, in line 8, specify the namespace that matches your intended consuming pod. The `metadata.name` value is referenced on the last line of `deploymentExample.yaml` in the next step. The `spec.resources.requests.storage` parameter determines the size of the persistent volume. It's 2 GB in this example, but can be modified to fit your needs:

   [!INCLUDE [lowercase-note](includes/lowercase-note.md)]

   ```yaml
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     ### Create a name for your PVC ###
     name: <create-a-pvc-name-here>
     ### Use a namespace that matches your intended consuming pod, or "default" ###
     namespace: <intended-consuming-pod-or-default-here>
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 2Gi
     storageClassName: unbacked-sc
   ```

Then, run this command to apply the YAML file and create the PVC:

```bash
kubectl apply -f "localSharedPVC.yaml"
```

## Create an example deployment or use your own

If you already have an application to deploy against, you can use it here. For the sake of simplicity and to get started, we include an example deployment here. To create a deployment using our example configuration, create a file named **deploymentExample.yaml** with the following contents. Add values for `containers.name` and `volumes.persistentVolumeClaim.claimName`. The `spec.replicas` parameter determines the number of replica pods to create. It's 2 in this example, but can be modified to fit your needs:

[!INCLUDE [lowercase-note](includes/lowercase-note.md)]

```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: localsharededgevol-deployment ### This will need to be unique for every volume you choose to create 

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
      containers: 

        ### Specify the container in which to launch the busy box. ### 
        - name: <create-a-container-name-here> 
          image: 'mcr.microsoft.com/mirror/docker/library/busybox:1.35' 
          command: 
            - "/bin/sh" 
            - "-c" 
            - "dd if=/dev/urandom of=/data/acsalocalsharedtestfile count=16 bs=1M && while true; do ls /data &>/dev/null || break; sleep 1; done" 
          volumeMounts: 

            ### This name must match the following volumes::name attribute ### 
            - name: wyvern-volume 

              ### This mountPath is where the PVC will be attached to the pod's filesystem ### 
              mountPath: /data 
      volumes: 

        ### User-defined name that is used to link the volumeMounts. This name must match volumeMounts::name as previously specified. ### 
        - name: wyvern-volume 
          persistentVolumeClaim: 

            ### This claimName must refer to your PVC metadata::name from lsevPVC.yaml. 
            claimName: <your-pvc-metadata-name-from-line-5-of-pvc-yaml>
```

Then, run this command to apply the YAML file and create the deployment:

```bash
kubectl apply -f "deploymentExample.yaml"
```

## Connect to your pod

Run `kubectl get pods` to find the name of your pod. Copy this name, as it's needed in the next step.

> [!NOTE]
> Because `spec::replicas` from **deploymentExample.yaml** was specified as **2**, two pods appear using `kubectl get pods`. You can use either pod name for the next step.

Run the following command and replace `POD_NAME_HERE` with your copied value from the previous step:

```bash
kubectl exec -it POD_NAME_HERE -- sh 
```

Change directories to the `/data` mount path, as specified in **deploymentExample.yaml**.

## Write an example file

Now that you're connected to the pod and in the mount path, any files you write here appear in your volume.

As an example, create a file named **file1.txt** and write to it using `echo "Hello World" > file1.txt`.

You can now see that the file you wrote is in the directory.

## Next steps

After you complete these steps, begin monitoring your deployment using Azure Monitor and Kubernetes Monitoring, or third-party monitoring with Prometheus and Grafana.

- [Monitor your Edge Volumes deployment](howto-azure-monitor-kubernetes.md)
