---
title: "Deploy and manage an Azure Arc-enabled Kubernetes cluster extension"
ms.custom: devx-track-azurecli
ms.date: 03/04/2026
ms.topic: how-to
description: "Learn how to create and manage an extension instance on an Azure Arc-enabled Kubernetes cluster."
# Customer intent: As a Kubernetes administrator, I want to deploy and manage extensions on my Azure Arc-enabled Kubernetes cluster, so that I can effectively access and utilize Azure services to enhance my cluster's capabilities.
---

# Deploy and manage an Azure Arc-enabled Kubernetes cluster extension

You can use extensions in Azure Arc-enabled Kubernetes clusters to enable Azure services and scenarios. This article describes how to create extension instances and set required and optional parameters, including options for updates and configurations. You also learn how to view, list, update, and delete extension instances.

Before you begin, read the [overview of Azure Arc-enabled Kubernetes cluster extensions](conceptual-extensions.md) and review the [list of currently available extensions](extensions-release.md). The steps in this article work for any extension, but review the documentation for the specific extension you want to deploy to understand any specific configuration settings or guidance.

## Prerequisites

* The latest version of the [Azure CLI](/cli/azure/install-azure-cli).
* The latest versions of the [`connectedk8s`](/cli/azure/connectedk8s) and [`k8s-extension`](/cli/azure/k8s-extension) Azure CLI extensions. To install or update the latest version of these extensions, run the following commands:
  
    ```azurecli
    az extension add --upgrade --name connectedk8s
    az extension add --upgrade --name k8s-extension
    ```

* An existing Azure Arc-enabled Kubernetes connected cluster, with at least one node of operating system and architecture type `linux/amd64`. If you only deploy the [Flux (GitOps) extension](extensions-release.md#flux-gitops), you can use an ARM64-based cluster without using a `linux/amd64` node.
  * If you haven't connected a cluster yet, use our [quickstart](quickstart-connect-cluster.md) to connect one.
  * [Upgrade your agents](agent-upgrade.md#manually-upgrade-agents) to the latest version.

## Create an extension instance

To create a new extension instance, use the [`k8s-extension create`](/cli/azure/k8s-extension#az-k8s-extension-create) command, and provide parameter values for your environment.

This example creates a [Container insights in Azure Monitor](extensions-release.md#container-insights-in-azure-monitor) extension instance on an Azure Arc-enabled Kubernetes cluster:

```azurecli
az k8s-extension create --name azuremonitor-containers --extension-type Microsoft.AzureMonitor.Containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

Check for output that looks like this example:

```json
{
  "autoUpgradeMinorVersion": true,
  "configurationProtectedSettings": null,
  "configurationSettings": {
    "logAnalyticsWorkspaceResourceID": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/defaultresourcegroup-eus/providers/microsoft.operationalinsights/workspaces/defaultworkspace-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-eus"
  },
  "creationTime": "2021-04-02T12:13:06.7534628+00:00",
  "errorInfo": {
    "code": null,
    "message": null
  },
  "extensionType": "microsoft.azuremonitor.containers",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/demo/providers/Microsoft.Kubernetes/connectedClusters/demo/providers/Microsoft.KubernetesConfiguration/extensions/azuremonitor-containers",
  "identity": null,
  "installState": "Pending",
  "lastModifiedTime": "2021-04-02T12:13:06.753463+00:00",
  "lastStatusTime": null,
  "name": "azuremonitor-containers",
  "releaseTrain": "Stable",
  "resourceGroup": "demo",
  "scope": {
    "cluster": {
      "releaseNamespace": "azuremonitor-containers"
    },
    "namespace": null
  },
  "statuses": [],
  "systemData": null,
  "type": "Microsoft.KubernetesConfiguration/extensions",
  "version": "2.8.2"
}
```

> [!NOTE]
> If Azure Arc-enabled Kubernetes agents don't have network connectivity for more than 48 hours, they can't create the extension on the cluster, and the extension transitions to a `Failed` state. If this condition occurs, run `k8s-extension create` again to create the extension resource.
> 
> Only one Container insights in Azure Monitor extension is required per cluster. Before you install Container insights via an extension, you must delete any previous Helm chart installations of Container insights that don't use extensions. Before you run `az k8s-extension create`, complete the steps to [delete the Helm chart](/azure/azure-monitor/containers/kubernetes-monitoring-disable#remove-container-insights-with-helm).

### Required parameters

The following table describes required parameters for using `az k8s-extension create` to create an extension instance:

| Parameter name | Description |
|----------------|------------|
| `--name` | The name of the extension instance. |
| `--extension-type` | The [type of extension](extensions-release.md) you want to install on the cluster. For example, `Microsoft.AzureMonitor.Containers` or `microsoft.azuredefender.kubernetes`. |
| `--cluster-name` | The name of the Azure Arc-enabled Kubernetes resource on which to create the extension instance. |
| `--resource-group` | The resource group that contains the Azure Arc-enabled Kubernetes resource. |
| `--cluster-type` | The cluster type on which to create the extension instance. For most scenarios, use `connectedClusters`, the cluster type for an Azure Arc-enabled Kubernetes cluster. |

### Optional parameters

Use one or more of these optional parameters with the required parameters for your scenario. Some of these parameters can't be used together. Review the description for each parameter and use the ones that apply to your scenario.

> [!NOTE]
> To automatically upgrade your extension instance to the latest minor and patch versions, set `auto-upgrade-minor-version` to `true`. We recommend enabling automatic upgrades for minor and patch versions so that you always have the latest security patches and capabilities. You can also set the version of the extension instance manually using the `--version` parameter.
>
> Because major version upgrades can include breaking changes, there's no automatic upgrade support for new major versions of an extension instance. You can choose when to [manually upgrade an extension instance](#upgrade-an-extension-instance) to a new major version.

| Parameter name | Description |
|--------------|------------|
| `--auto-upgrade-minor-version` | A Boolean property that sets whether the extension minor version upgrades automatically. The default setting is `true`. If you set this parameter to `true`, you can't set the `version` parameter, because the version is dynamically updated. If you set this parameter to `false`, the extension isn't automatically upgraded, even for patch versions. |
| `--version` | The version of the extension to install (the specific version to pin the extension instance to). You can't set the `version` parameter if `auto-upgrade-minor-version` is set to `true`. |
| `--configuration-settings` | Settings that you can pass into the extension to control its functionality. Pass in these settings as space-separated `key=value` pairs after the parameter name. If you use this parameter in the command, you can't pass `--config-settings-file` in the same command. |
| `--config-settings-file` | The path to a JSON file with `key=value` pairs to use for passing configuration settings into the extension. If you use this parameter in the command, you can't use `--configuration-settings` in the same command. |
| `--config-protected-settings` | Settings that aren't retrievable by using `GET` API calls or `az k8s-extension show` commands. Typically used to pass in sensitive settings. Pass in these settings as space-separated `key=value` pairs after the parameter name. If you use this parameter in the command, you can't use `--config-protected-settings-file` in the same command. |
| `--config-protected-settings-file` | The path to a JSON file with `key=value` pairs to use to pass sensitive settings into the extension. If you use this parameter in the command, you can't use `--config-protected-settings` in the same command. |
| `--release-train` |  The release train, if the extension has published versions in different release trains such as `Stable` or `Preview`. If you don't set this parameter explicitly, `Stable` is the default. |
| `--scope` | The [scope of installation](conceptual-extensions.md#extension-scope) for the extension (`cluster` or `namespace`). Most extensions are cluster-scoped, so this parameter is only required if the extension is namespace-scoped. |
| `--release-namespace` | When `scope` is set to `cluster`, this parameter specifies the namespace in which to create the release. |
| `--target-namespace` | When `scope` is set to `namespace`, this parameter specifies the namespace in which to create the release. Permissions for the system account that's created for this extension instance are restricted to this namespace. |

For information about other optional parameters, see the [`az k8s extension create`](/cli/azure/k8s-extension#az-k8s-extension-create) reference documentation.

## Show extension details

To view details of a currently installed extension instance, use the [`k8s-extension show`](/cli/azure/k8s-extension#az-k8s-extension-show) command. Use values from your scenario for the required parameter placeholders.

```azurecli
az k8s-extension show --name azuremonitor-containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

You should see output similar to the following example:

```json
{
  "autoUpgradeMinorVersion": true,
  "configurationProtectedSettings": null,
  "configurationSettings": {
    "logAnalyticsWorkspaceResourceID": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/defaultresourcegroup-eus/providers/microsoft.operationalinsights/workspaces/defaultworkspace-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-eus"
  },
  "creationTime": "2021-04-02T12:13:06.7534628+00:00",
  "errorInfo": {
    "code": null,
    "message": null
  },
  "extensionType": "microsoft.azuremonitor.containers",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/demo/providers/Microsoft.Kubernetes/connectedClusters/demo/providers/Microsoft.KubernetesConfiguration/extensions/azuremonitor-containers",
  "identity": null,
  "installState": "Installed",
  "lastModifiedTime": "2021-04-02T12:13:06.753463+00:00",
  "lastStatusTime": "2021-04-02T12:13:49.636+00:00",
  "name": "azuremonitor-containers",
  "releaseTrain": "Stable",
  "resourceGroup": "demo",
  "scope": {
    "cluster": {
      "releaseNamespace": "azuremonitor-containers"
    },
    "namespace": null
  },
  "statuses": [],
  "systemData": null,
  "type": "Microsoft.KubernetesConfiguration/extensions",
  "version": "2.8.2"
}
```

## List all extensions installed on the cluster

To view a list of all extensions that are installed on a cluster, use the [`k8s-extension list`](/cli/azure/k8s-extension#az-k8s-extension-list) command. Use values from your scenario for the required parameter placeholders.

```azurecli
az k8s-extension list --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

You should see output similar to the following example:

```json
[
  {
    "autoUpgradeMinorVersion": true,
    "creationTime": "2020-09-15T02:26:03.5519523+00:00",
    "errorInfo": {
      "code": null,
      "message": null
    },
    "extensionType": "Microsoft.AzureMonitor.Containers",
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myRg/providers/Microsoft.Kubernetes/connectedClusters/myCluster/providers/Microsoft.KubernetesConfiguration/extensions/myExtInstanceName",
    "identity": null,
    "installState": "Pending",
    "lastModifiedTime": "2020-09-15T02:48:45.6469664+00:00",
    "lastStatusTime": null,
    "name": "myExtInstanceName",
    "releaseTrain": "Stable",
    "resourceGroup": "myRG",
    "scope": {
      "cluster": {
        "releaseNamespace": "myExtInstanceName1"
      }
    },
    "statuses": [],
    "type": "Microsoft.KubernetesConfiguration/extensions",
    "version": "0.1.0"
  },
  {
    "autoUpgradeMinorVersion": true,
    "creationTime": "2020-09-02T00:41:16.8005159+00:00",
    "errorInfo": {
      "code": null,
      "message": null
    },
    "extensionType": "microsoft.azuredefender.kubernetes",
    "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/myRg/providers/Microsoft.Kubernetes/connectedClusters/myCluster/providers/Microsoft.KubernetesConfiguration/extensions/defender",
    "identity": null,
    "installState": "Pending",
    "lastModifiedTime": "2020-09-02T00:41:16.8005162+00:00",
    "lastStatusTime": null,
    "name": "microsoft.azuredefender.kubernetes",
    "releaseTrain": "Stable",
    "resourceGroup": "myRg",
    "scope": {
      "cluster": {
        "releaseNamespace": "myExtInstanceName2"
      }
    },
    "type": "Microsoft.KubernetesConfiguration/extensions",
    "version": "0.1.0"
  }
]
```

## Update an extension instance

> [!NOTE]
> For information about specific settings in `--configuration-settings` and `--config-protected-settings` that you can update, see the documentation for the [specific extension](extensions-release.md). For `--config-protected-settings`, provide all settings, even if you update only one setting. If you omit any of these settings, the omitted settings are deleted.

To update an existing extension instance, use [`k8s-extension update`](/cli/azure/k8s-extension#az-k8s-extension-update). Pass in values for the mandatory and optional parameters. These parameters differ slightly from the parameters that you use to create an extension instance.

This example updates the `auto-upgrade-minor-version` setting for an Azure Machine Learning extension instance to `true`:

```azurecli
az k8s-extension update --name azureml --extension-type Microsoft.AzureML.Kubernetes --cluster-name <clusterName> --resource-group <resourceGroupName> --auto-upgrade-minor-version true --cluster-type managedClusters
```

### Required parameters for an extension update

| Parameter name | Description |
|----------------|------------|
| `--name` | The name of the extension instance. |
| `--cluster-name` | The name of the cluster on which you create the extension instance. |
| `--resource-group` | The resource group that contains the cluster. |
| `--cluster-type` | The cluster type on which to create the extension instance. For Azure Arc-enabled Kubernetes clusters, use `connectedClusters`. For AKS clusters, use `managedClusters`.|

### Optional parameters for an extension update

| Parameter name | Description |
|--------------|------------|
| `--auto-upgrade-minor-version` | A Boolean property that specifies whether the extension minor version is automatically upgraded. The default setting is `true`. If this parameter is set to `true`, you can't set the `version` parameter because the version is dynamically updated. If the parameter is set to `false`, the extension isn't automatically upgraded, even for patch versions. |
| `--version` | The version of the extension to install (a specific version to pin the extension instance to). Must not be supplied if `auto-upgrade-minor-version` is set to `true`. |
| `--configuration-settings` | Settings that you can pass into the extension to control its functionality. Pass in these settings as space-separated `key=value` pairs after the parameter name. If you use this parameter in the command, you can't use `--config-settings-file` in the same command. Only provide the settings that require an update. The provided settings are replaced with the specified values. |
| `--config-settings-file` | The path to a JSON file that contains `key=value` pairs to use for passing in configuration settings to the extension. If you use this parameter in the command, you can't use `--configuration-settings` in the same command. |
| `--config-protected-settings` | Settings that aren't retrievable by using `GET` API calls or `az k8s-extension show` commands. Typically used to pass in sensitive settings. Pass in these settings as space-separated `key=value` pairs after the parameter name. If you use this parameter in the command, you can't use the `--config-protected-settings-file` in the same command. When you update a protected setting, configure all protected settings. If you omit any of the settings, those settings are considered obsolete, and they're deleted.  |
| `--config-protected-settings-file` | The path to a JSON file that contains `key=value` pairs to use for passing in sensitive settings to the extension. If you use this parameter in the command, you can't use `--config-protected-settings` in the same command. |
| `--scope` | The [scope of installation](conceptual-extensions.md#extension-scope) for the extension (`cluster` or `namespace`). Most extensions are cluster-scoped, so this parameter is only required if the extension is namespace-scoped. |
| `--release-train` | The release train, if the extension has published versions in different release trains such as `Stable` or `Preview`. If you don't set this parameter explicitly, `Stable` is the default. |

For information about other optional parameters, see the [`az k8s extension update`](/cli/azure/k8s-extension#az-k8s-extension-update) reference documentation.

## Upgrade an extension instance

As noted earlier, if you set `auto-upgrade-minor-version` to `true`, the extension automatically upgrades when a new minor version is released. For most scenarios, enable automatic upgrades. If you set `auto-upgrade-minor-version` to `false`, you must upgrade the extension manually if you want a more recent version.

You need to manually upgrade to get a new major version of an extension. To avoid unexpected breaking changes in major version upgrades, you can choose when to upgrade your extensions.

To manually upgrade an extension instance, use [`k8s-extension update`](/cli/azure/k8s-extension#az-k8s-extension-update) and set the `version` parameter.

This example updates an Azure Machine Learning extension instance to version `x.y.z`:

```azurecli
az k8s-extension update --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters --name azureml --version x.y.z
```

## Delete an extension instance

To delete an extension instance on a cluster, use [`k8s-extension delete`](/cli/azure/k8s-extension#az-k8s-extension-delete). Use values from your scenario for the required parameter placeholders.

```azurecli
az k8s-extension delete --name azuremonitor-containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

> [!NOTE]
> This command immediately deletes the Azure extension resource. However, the command deletes the Helm release on the cluster that's associated with this extension only when the agents running on the Kubernetes cluster have network connectivity and can reach Azure services.

## Related content

* For a comprehensive list of commands and parameters, see the [az k8s-extension CLI reference](/cli/azure/k8s-extension).
* Learn [how extensions work with Azure Arc-enabled Kubernetes clusters](conceptual-extensions.md).
* Review [cluster extensions that are available for Azure Arc-enabled Kubernetes](extensions-release.md).
* For help with troubleshooting, see [troubleshooting extension issues](extensions-troubleshooting.md).
