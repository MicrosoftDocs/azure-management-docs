---
title: "Tutorial: Enable Telemetry and Monitoring for your Azure Container Linux (ACL) Cluster"
description: In this Azure Linux tutorial, you learn how to enable telemetry and monitoring for your Azure Container Linux (ACL) cluster.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: tutorial
ms.date: 04/28/2026
---

# Tutorial: Enable telemetry and monitoring for your Azure Container Linux (ACL) cluster

In this tutorial, part _four of five_, you learn how to:

> [!div class="checklist"]
>
> - Enable Container Insights to monitor your existing cluster.
> - Verify the agent is deployed successfully.
> - Verify the solution is enabled.

The commands in this tutorial use the environment variables set in [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).

In the next tutorial, you learn how to upgrade your ACL nodes.

## Prerequisites

- In previous tutorials, you created and deployed an ACL cluster. To complete this tutorial, you need an existing cluster. If you haven't completed this step and want to follow along, start with [Tutorial 1: Create a cluster with ACL for AKS](./tutorial-create-cluster-acl-aks.md).
- If you're connecting an existing AKS cluster to a Log Analytics workspace in another subscription, you need to register the `Microsoft.ContainerService` resource provider in the subscription with the Log Analytics workspace. For more information, see [Register resource provider](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider).
- Azure Container Linux requires Azure CLI version 2.86.0 or higher. Use the [`az version`](/cli/azure/reference-index?#az-version) command to find the version. To upgrade to the latest version, use the [`az upgrade`](/cli/azure/reference-index?#az-upgrade) command.

[!INCLUDE [azure container linux limitations](./includes/acl-limitations.md)]

## Connect to your cluster

Before enabling monitoring, it's important to ensure you're connected to the correct cluster. Get the credentials for your ACL cluster and configure kubectl to use them using the [`az aks get-credentials`](/cli/azure/aks#az-aks-get-credentials) command.

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Enable monitoring

You can enable monitoring for your ACL cluster using a [default Log Analytics workspace](#option-1-use-a-default-log-analytics-workspace) or by [specifying a Log Analytics workspace](#option-2-specify-a-log-analytics-workspace). The following steps show you how to enable monitoring for your cluster using either method.

### Option 1: Use a default Log Analytics workspace

Use the following commands to check if the monitoring add-on is already enabled for your cluster. If it isn't, the command enables monitoring for your ACL cluster using a default Log Analytics workspace in the default resource group of the AKS cluster subscription. If one doesn't already exist in the region, the default workspace created resembles the following format: _DefaultWorkspace-< GUID >-< Region >_.

```bash
# Check if monitoring addon is already enabled
MONITORING_ENABLED=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "addonProfiles.omsagent.enabled" -o tsv)

if [ "$MONITORING_ENABLED" != "true" ]; then
  az aks enable-addons --addons monitoring --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP
fi
```

### Option 2: Specify a Log Analytics workspace

You can specify a Log Analytics workspace to enable monitoring of your ACL cluster. The resource ID of the workspace is in the following format: `"/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>"`.

Enable monitoring with a specified workspace using the [`az aks enable-addons`](/cli/azure/aks#az_aks_enable_addons) command. Replace `<workspace-resource-id>` with the resource ID of your Log Analytics workspace.

```azurecli-interactive
az aks enable-addons --addons monitoring --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --workspace-resource-id <workspace-resource-id>
```

## Verify agent and solution deployment

1. Verify the agent is deployed successfully using the following command:

    ```bash
    kubectl get ds ama-logs --namespace=kube-system
    ```

    Example output:

    <!-- expected_similarity=0.3 -->
    ```output
    NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    ama-logs   3         3         3       3            3           <none>          3m22s
    ```

1. Verify deployment of the solution using the following command:

    ```bash
    kubectl get deployment ama-logs-rs -n=kube-system
    ```

    Example output:

    <!-- expected_similarity=0.3 -->
    ```output
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

In this tutorial, you enabled telemetry and monitoring for your ACL cluster. In the next tutorial, you learn how to upgrade your ACL nodes.

> [!div class="nextstepaction"]
> [Upgrade ACL nodes](./tutorial-upgrade-acl-nodes.md)