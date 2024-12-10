---
title: 'Quickstart: Deploy an Azure Linux Container Host for AKS cluster by using the Azure CLI'
description: Learn how to quickly create an Azure Linux Container Host for AKS cluster using the Azure CLI.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: references_regions, devx-track-azurecli, linux-related-content, innovation-engine
ms.topic: quickstart
ms.date: 04/18/2023
---

# Quickstart: Deploy an Azure Linux Container Host for AKS cluster by using the Azure CLI

Get started with the Azure Linux Container Host by using the Azure CLI to deploy an Azure Linux Container Host for AKS cluster. After installing the prerequisites, you will create a resource group, create an AKS cluster, connect to the cluster, and run a sample multi-container application in the cluster.

## Prerequisites

- [!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]
- Use the Bash environment in [Azure Cloud Shell](/azure/cloud-shell/overview). For more information, see [Azure Cloud Shell Quickstart - Bash](/azure/cloud-shell/quickstart).

  :::image type="icon" source="~/reusable-content/ce-skilling/azure/media/cloud-shell/launch-cloud-shell-button.png" border="false" link="https://portal.azure.com/#cloudshell/":::

- If you prefer to run CLI reference commands locally, [install](/cli/azure/install-azure-cli) the Azure CLI. If you're running on Windows or macOS, consider running Azure CLI in a Docker container. For more information, see [How to run the Azure CLI in a Docker container](/cli/azure/run-azure-cli-docker).

  - If you're using a local installation, sign in to the Azure CLI by using the [az login](/cli/azure/reference-index#az-login) command. To finish the authentication process, follow the steps displayed in your terminal. For other sign-in options, see [Sign in with the Azure CLI](/cli/azure/authenticate-azure-cli).
  - When you're prompted, install the Azure CLI extension on first use. For more information about extensions, see [Use extensions with the Azure CLI](/cli/azure/azure-cli-extensions-overview).
  - Run [`az version`](/cli/azure/reference-index?#az-version) to find the version and dependent libraries that are installed. To upgrade to the latest version, run [az upgrade](/cli/azure/reference-index?#az-upgrade).

## Create a resource group

An Azure resource group is a logical group in which Azure resources are deployed and managed. When creating a resource group, it is required to specify a location. This location is:

- The storage location of your resource group metadata.
- Where your resources will run in Azure if you don't specify another region when creating a resource.

Create a resource group using the `az group create` command.

```azurecli-interactive
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myAzureLinuxResourceGroup$RANDOM_ID"
export REGION="westeurope"

az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```

Results:
<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/$MY_RESOURCE_GROUP_NAMExxxxxx",
  "location": "$REGION",
  "managedBy": null,
  "name": "$MY_RESOURCE_GROUP_NAME",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create an Azure Linux Container Host cluster

Create an AKS cluster using the `az aks create` command with the `--os-sku` parameter to provision the AKS cluster with an Azure Linux image.

```azurecli-interactive
export MY_AZ_CLUSTER_NAME="myAzureLinuxCluster$RANDOM_ID"

az aks create --name $MY_AZ_CLUSTER_NAME --resource-group $MY_RESOURCE_GROUP_NAME --os-sku AzureLinux
```

After a few minutes, the command completes and returns JSON-formatted information about the cluster.

## Connect to the cluster

To manage a Kubernetes cluster, use the Kubernetes command-line client, `kubectl`. `kubectl` is already installed if you use Azure Cloud Shell. To install `kubectl` locally, use the `az aks install-cli` command.

1. Configure `kubectl` to connect to your Kubernetes cluster using the `az aks get-credentials` command. This command downloads credentials and configures the Kubernetes CLI to use them.

    ```azurecli-interactive
    az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AZ_CLUSTER_NAME
    ```

1. Verify the connection to your cluster using the `kubectl get` command. This command returns a list of the cluster nodes.

    ```bash
    kubectl get nodes
    ```

## Deploy the application

To deploy the application, you use a manifest file to create all the objects required to run the [AKS Store application](https://github.com/Azure-Samples/aks-store-demo). A Kubernetes manifest file defines a cluster's desired state, such as which container images to run. The manifest includes the following Kubernetes deployments and services:

:::image type="content" source="media/aks-store-architecture.png" alt-text="Screenshot of Azure Store sample architecture." lightbox="media/aks-store-architecture.png":::

- **Store front**: Web application for customers to view products and place orders.
- **Product service**: Shows product information.
- **Order service**: Places orders.
- **Rabbit MQ**: Message queue for an order queue.

> [!NOTE]
> We don't recommend running stateful containers, such as Rabbit MQ, without persistent storage for production. These are used here for simplicity, but we recommend using managed services, such as Azure CosmosDB or Azure Service Bus.

1. Create a file named `aks-store-quickstart.yaml` and copy in the following manifest:

    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: rabbitmq
    spec:
      serviceName: rabbitmq
      replicas: 1
      selector:
        matchLabels:
          app: rabbitmq
      template:
        metadata:
          labels:
            app: rabbitmq
        spec:
          nodeSelector:
            "kubernetes.io/os": linux
          containers:
          - name: rabbitmq
            image: mcr.microsoft.com/mirror/docker/library/rabbitmq:3.10-management-alpine
            ports:
            - containerPort: 5672
              name: rabbitmq-amqp
            - containerPort: 15672
              name: rabbitmq-http
            env:
            - name: RABBITMQ_DEFAULT_USER
              value: "username"
            - name: RABBITMQ_DEFAULT_PASS
              value: "password"
            resources:
              requests:
                cpu: 10m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            volumeMounts:
            - name: rabbitmq-enabled-plugins
              mountPath: /etc/rabbitmq/enabled_plugins
              subPath: enabled_plugins
          volumes:
          - name: rabbitmq-enabled-plugins
            configMap:
              name: rabbitmq-enabled-plugins
              items:
              - key: rabbitmq_enabled_plugins
                path: enabled_plugins
    ---
    apiVersion: v1
    data:
      rabbitmq_enabled_plugins: |
        [rabbitmq_management,rabbitmq_prometheus,rabbitmq_amqp1_0].
    kind: ConfigMap
    metadata:
      name: rabbitmq-enabled-plugins            
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: rabbitmq
    spec:
      selector:
        app: rabbitmq
      ports:
        - name: rabbitmq-amqp
          port: 5672
          targetPort: 5672
        - name: rabbitmq-http
          port: 15672
          targetPort: 15672
      type: ClusterIP
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: order-service
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: order-service
      template:
        metadata:
          labels:
            app: order-service
        spec:
          nodeSelector:
            "kubernetes.io/os": linux
          containers:
          - name: order-service
            image: ghcr.io/azure-samples/aks-store-demo/order-service:latest
            ports:
            - containerPort: 3000
            env:
            - name: ORDER_QUEUE_HOSTNAME
              value: "rabbitmq"
            - name: ORDER_QUEUE_PORT
              value: "5672"
            - name: ORDER_QUEUE_USERNAME
              value: "username"
            - name: ORDER_QUEUE_PASSWORD
              value: "password"
            - name: ORDER_QUEUE_NAME
              value: "orders"
            - name: FASTIFY_ADDRESS
              value: "0.0.0.0"
            resources:
              requests:
                cpu: 1m
                memory: 50Mi
              limits:
                cpu: 75m
                memory: 128Mi
            startupProbe:
              httpGet:
                path: /health
                port: 3000
              failureThreshold: 5
              initialDelaySeconds: 20
              periodSeconds: 10
            readinessProbe:
              httpGet:
                path: /health
                port: 3000
              failureThreshold: 3
              initialDelaySeconds: 3
              periodSeconds: 5
            livenessProbe:
              httpGet:
                path: /health
                port: 3000
              failureThreshold: 5
              initialDelaySeconds: 3
              periodSeconds: 3
          initContainers:
          - name: wait-for-rabbitmq
            image: busybox
            command: ['sh', '-c', 'until nc -zv rabbitmq 5672; do echo waiting for rabbitmq; sleep 2; done;']
            resources:
              requests:
                cpu: 1m
                memory: 50Mi
              limits:
                cpu: 75m
                memory: 128Mi    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: order-service
    spec:
      type: ClusterIP
      ports:
      - name: http
        port: 3000
        targetPort: 3000
      selector:
        app: order-service
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: product-service
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: product-service
      template:
        metadata:
          labels:
            app: product-service
        spec:
          nodeSelector:
            "kubernetes.io/os": linux
          containers:
          - name: product-service
            image: ghcr.io/azure-samples/aks-store-demo/product-service:latest
            ports:
            - containerPort: 3002
            env: 
            - name: AI_SERVICE_URL
              value: "http://ai-service:5001/"
            resources:
              requests:
                cpu: 1m
                memory: 1Mi
              limits:
                cpu: 2m
                memory: 20Mi
            readinessProbe:
              httpGet:
                path: /health
                port: 3002
              failureThreshold: 3
              initialDelaySeconds: 3
              periodSeconds: 5
            livenessProbe:
              httpGet:
                path: /health
                port: 3002
              failureThreshold: 5
              initialDelaySeconds: 3
              periodSeconds: 3
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: product-service
    spec:
      type: ClusterIP
      ports:
      - name: http
        port: 3002
        targetPort: 3002
      selector:
        app: product-service
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: store-front
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: store-front
      template:
        metadata:
          labels:
            app: store-front
        spec:
          nodeSelector:
            "kubernetes.io/os": linux
          containers:
          - name: store-front
            image: ghcr.io/azure-samples/aks-store-demo/store-front:latest
            ports:
            - containerPort: 8080
              name: store-front
            env: 
            - name: VUE_APP_ORDER_SERVICE_URL
              value: "http://order-service:3000/"
            - name: VUE_APP_PRODUCT_SERVICE_URL
              value: "http://product-service:3002/"
            resources:
              requests:
                cpu: 1m
                memory: 200Mi
              limits:
                cpu: 1000m
                memory: 512Mi
            startupProbe:
              httpGet:
                path: /health
                port: 8080
              failureThreshold: 3
              initialDelaySeconds: 5
              periodSeconds: 5
            readinessProbe:
              httpGet:
                path: /health
                port: 8080
              failureThreshold: 3
              initialDelaySeconds: 3
              periodSeconds: 3
            livenessProbe:
              httpGet:
                path: /health
                port: 8080
              failureThreshold: 5
              initialDelaySeconds: 3
              periodSeconds: 3
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: store-front
    spec:
      ports:
      - port: 80
        targetPort: 8080
      selector:
        app: store-front
      type: LoadBalancer
    ```

    If you create and save the YAML file locally, then you can upload the manifest file to your default directory in CloudShell by selecting the **Upload/Download files** button and selecting the file from your local file system.

1. Deploy the application using the [`kubectl apply`][kubectl-apply] command and specify the name of your YAML manifest.

    ```bash
    kubectl apply -f aks-store-quickstart.yaml
    ```

## Test the application

You can validate that the application is running by visiting the public IP address or the application URL.

Get the application URL using the following commands:

```azurecli-interactive
runtime="5 minutes"
endtime=$(date -ud "$runtime" +%s)
while [[ $(date -u +%s) -le $endtime ]]
do
   STATUS=$(kubectl get pods -l app=store-front -o 'jsonpath={..status.conditions[?(@.type=="Ready")].status}')
   echo $STATUS
   if [ "$STATUS" == 'True' ]
   then
      export IP_ADDRESS=$(kubectl get service store-front --output 'jsonpath={..status.loadBalancer.ingress[0].ip}')
      echo "Service IP Address: $IP_ADDRESS"
      break
   else
      sleep 10
   fi
done
```

```azurecli-interactive
curl $IP_ADDRESS
```

Results:
<!-- expected_similarity=0.3 -->
```HTML
<!doctype html>
<html lang="">
   <head>
      <meta charset="utf-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width,initial-scale=1">
      <link rel="icon" href="/favicon.ico">
      <title>store-front</title>
      <script defer="defer" src="/js/chunk-vendors.df69ae47.js"></script>
      <script defer="defer" src="/js/app.7e8cfbb2.js"></script>
      <link href="/css/app.a5dc49f6.css" rel="stylesheet">
   </head>
   <body>
      <div id="app"></div>
   </body>
</html>
```

```OUTPUT
echo "You can now visit your web server at $IP_ADDRESS"
```

## Delete the cluster

If you no longer need them, you can clean up unnecessary resources to avoid Azure charges. You can remove the resource group, container service, and all related resources using the `az group delete` command.

## Next steps

In this quickstart, you deployed an Azure Linux Container Host cluster. To learn more about the Azure Linux Container Host, and walk through a complete cluster deployment and management example, continue to the Azure Linux Container Host tutorial.

> [!div class="nextstepaction"]
> [Azure Linux Container Host tutorial](./tutorial-azure-linux-create-cluster.md)

<!-- LINKS -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
