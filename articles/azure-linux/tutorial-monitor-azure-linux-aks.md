---
title: "Tutorial: Enable Telemetry and Monitoring for your Azure Linux Container Host Cluster"
description: In this Azure Linux tutorial, you learn how to enable telemetry and monitoring for your Azure Linux Container Host cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Enable telemetry and monitoring for your Azure Linux Container Host cluster

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321847)

In this tutorial, part _four of five_, you learn how to:

> [!div class="checklist"]
>
> - Enable Container Insights to monitor your existing cluster.
> - Verify the agent is deployed successfully.
> - Verify the solution is enabled.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).

In the next and last tutorial, you learn how to upgrade your Azure Linux nodes.

## Prerequisites

- In previous tutorials, you created and deployed an Azure Linux Container Host cluster. To complete this tutorial, you need an existing cluster. If you haven't done this step and would like to follow along, start with [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md).
- If you're connecting an existing AKS cluster to a Log Analytics workspace in another subscription, the Microsoft.ContainerService resource provider must be registered in the subscription with the Log Analytics workspace. For more information, see [Register resource provider](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider).
- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Connect to your cluster

Before enabling monitoring, it's important to ensure you're connected to the correct cluster. Get the credentials for your Azure Linux Container Host cluster and configure kubectl to use them using the [`az aks get-credentials`](/cli/azure/aks#az_aks_get_credentials) command.

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Enable monitoring

You can enable monitoring for your Azure Linux Container Host cluster using a [default Log Analytics workspace](#option-1-use-a-default-log-analytics-workspace) or by [specifying a Log Analytics workspace](#option-2-specify-a-log-analytics-workspace). The following steps show you how to enable monitoring for your cluster using either method.

### Option 1: Use a default Log Analytics workspace

Use the following commands to check if the monitoring add-on is already enabled for your cluster. If it isn't, the command enables monitoring for your Azure Linux Container Host cluster using a default Log Analytics workspace in the default resource group of the AKS cluster subscription. If one doesn't already exist in the region, the default workspace created resembles the following format: _DefaultWorkspace-< GUID >-< Region >_.

```bash
# Check if monitoring addon is already enabled
MONITORING_ENABLED=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "addonProfiles.omsagent.enabled" -o tsv)

if [ "$MONITORING_ENABLED" != "true" ]; then
  az aks enable-addons --addons monitoring --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP
fi
```

### Option 2: Specify a Log Analytics workspace

You can specify a Log Analytics workspace to enable monitoring of your Azure Linux Container Host cluster. The resource ID of the workspace is in the following format: `"/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>"`.

Enable monitoring with a specified workspace using the [`az aks enable-addons`](/cli/azure/aks#az_aks_enable_addons) command. Replace `<workspace-resource-id>` with the resource ID of your Log Analytics workspace.

```azurecli-interactive
az aks enable-addons -addons monitoring --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --workspace-resource-id <workspace-resource-id>
```

## Verify agent and solution deployment

1. Verify the agent is deployed successfully using the following command:

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

1. Verify deployment of the solution using the following command:

    ```bash
    kubectl get deployment ama-logs-rs -n=kube-system
    ```

    Example output:

    <!-- expected_similarity=0.3 -->
    ```text
    User@aksuser:~$ kubectl get deployment ama-logs-rs -n=kube-system
    NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    ama-logs-rs    1         1         1            1            3h
    ```

## Verify solution configuration

Get the configuration of the solution using the [`az aks show`](/cli/azure/aks#az_aks_show) command. With this command, you can check if the solution is enabled, what the Log Analytics workspace resource ID is, and get summary information about the cluster.

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

## Next step

In this tutorial, you enabled telemetry and monitoring for your Azure Linux Container Host cluster. In the next tutorial, you learn how to upgrade your Azure Linux nodes.

> [!div class="nextstepaction"]
> [Upgrade Azure Linux nodes](./tutorial-upgrade-azure-linux-nodes.md)
