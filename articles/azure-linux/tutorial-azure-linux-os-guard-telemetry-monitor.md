---
title: Azure Linux with OS Guard (preview) for Azure Kubernetes Service (AKS) tutorial - Enable telemetry and monitoring for Azure Linux with OS Guard (preview)
description: In this Azure Linux with OS Guard for AKS tutorial, you'll learn how to enable telemetry and monitoring for Azure Linux with OS Guard.
author: florataagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 09/24/2025
# Customer intent: "As a cloud administrator, I want to enable telemetry and monitoring for my Azure Linux with OS Guard cluster, so that I can ensure optimal performance and gain insights into the cluster's health."
---

# Tutorial: Enable telemetry and monitoring for your Azure Linux with OS Guard (preview)cluster

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321847)

In this tutorial, part four of five, you set up Container Insights to monitor an Azure Linux with OS Guard cluster. You'll learn how to: 

> [!div class="checklist"]
> - Enable monitoring for an existing cluster.
> - Verify that the agent is deployed successfully.
> - Verify that the solution is enabled.

In the next tutorial, you learn how to upgrade your Azure Linux nodes.

## Prerequisites

- In previous tutorials, you created and deployed an Azure Linux with OS Guard cluster. To complete this tutorial, you need an existing cluster. If you haven't completed this step and want to follow along, start with [Tutorial 1: Create a cluster with Azure Linux with OS Guard for AKS](./tutorial-azure-linux-os-guard-create-cluster.md).
- If you're connecting an existing AKS cluster to a Log Analytics workspace in another subscription, you need to register the `Microsoft.ContainerService` resource provider in the subscription with the Log Analytics workspace. For more information, see [Register resource provider](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider).
- You need the latest version of Azure CLI. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

## Connect to your cluster

Before enabling monitoring, it's important to ensure you're connected to the correct cluster. Get the credentials for your Azure Linux with OS Guard cluster and configure kubectl to use them using the [`az aks get-credentials`](/cli/azure/aks#az-aks-get-credentials) command.

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Enable monitoring

### Use a default Log Analytics workspace

The following step enables monitoring for your Azure Linux with OS Guard cluster using Azure CLI. In this example, you aren't required to specify an existing workspace. The [`az aks enable-addons`](/cli/azure/aks#az-aks-enable-addons) command simplifies the process for you by creating a default workspace in the default resource group of the AKS cluster subscription. If one doesn't already exist in the region, the default workspace created will resemble the format `DefaultWorkspace-< GUID >-< Region >`. 

```azurecli-interactive
# Check if monitoring addon is already enabled
MONITORING_ENABLED=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "addonProfiles.omsagent.enabled" -o tsv)

if [ "$MONITORING_ENABLED" != "true" ]; then
  az aks enable-addons --addons monitoring --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP
fi
```

### Option 2: Specify a Log Analytics workspace

In this example, you can specify a Log Analytics workspace to enable monitoring of your Azure Linux with OS Guard cluster. The resource ID of the workspace will be in the form `"/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<WorkspaceName>"`.

```azurecli-interactive
az aks enable-addons --addons monitoring --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --workspace-resource-id <workspace-resource-id>
```

## Verify agent and solution deployment

Verify that the agent is successfully deployed using the `kubectl get ds` command.

```bash
kubectl get ds ama-logs --namespace=kube-system
```

Example output:

<!-- expected_similarity=0.3 -->
```output
User@aksuser:~$ kubectl get ds ama-logs --namespace=kube-system
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ama-logs   3         3         3       3            3           <none>          3m22s
```

Verify deployment of the solution using the `kubectl get deployment` command.

```bash
kubectl get deployment ama-logs-rs -n=kube-system
```

Example output:

<!-- expected_similarity=0.3 -->
```output
User@aksuser:~$ kubectl get deployment ama-logs-rs -n=kube-system
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
ama-logs-rs    1         1         1            1            3h
```

## Verify solution configuration

Verify the solution is enabled, retrieve the Log Analytics resource ID, and get summary information about the cluster using the [`az aks show`](/cli/azure/aks#az-aks-show) command.

```azurecli-interactive
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "addonProfiles.omsagent"
```

After a few minutes, the command completes and returns JSON-formatted information about the solution. The results of the command should show the monitoring add-on profile and resemble the following example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "config": {
    "logAnalyticsWorkspaceResourceID": "/subscriptions/xxxxx/resourceGroups/xxxxx/providers/Microsoft.OperationalInsights/workspaces/xxxxx"
  },
  "enabled": true
}
```

## Next steps

In this tutorial, you enabled telemetry and monitoring for your Azure Linux with OS Guard cluster. In the next tutorial, you learn how to upgrade your Azure Linux with OS Guard nodes.

> [!div class="nextstepaction"]
> [Upgrade Azure Linux with OS Guard nodes](./tutorial-azure-linux-os-guard-upgrade.md)