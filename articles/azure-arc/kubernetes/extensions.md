---
title: "Deploy and manage an Azure Arc-enabled Kubernetes cluster extension"
ms.custom: devx-track-azurecli
ms.date: 02/08/2024
ms.topic: how-to
description: "Learn how to create and manage an extension instance on an Azure Arc-enabled Kubernetes cluster."
# Customer intent: As a Kubernetes administrator, I want to deploy and manage extensions on my Azure Arc-enabled Kubernetes cluster, so that I can effectively access and utilize Azure services to enhance my cluster's capabilities.
---

# Deploy and manage an Azure Arc-enabled Kubernetes cluster extension

You can use an extension in an Azure Arc-enabled Kubernetes cluster to access Azure services and scenarios. This article describes how to create extension instances and set required and optional parameters, including options for updates and configurations. You also learn how to view, list, update, and delete extension instances.

Before you begin, read the [overview of Azure Arc-enabled Kubernetes cluster extensions](conceptual-extensions.md) and review the [list of currently available extensions](extensions-release.md).

## Prerequisites

* The latest version of the [Azure CLI](/cli/azure/install-azure-cli).
* The latest versions of the `connectedk8s` and `k8s-extension` Azure CLI extensions. To install these extensions, run the following commands:
  
    ```azurecli
    az extension add --name connectedk8s
    az extension add --name k8s-extension
    ```

    If the `connectedk8s` and `k8s-extension` extensions are already installed, make sure that they're updated to the latest version by using these commands:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-extension
    ```

* An existing Azure Arc-enabled Kubernetes connected cluster, with at least one node of operating system and architecture type `linux/amd64`. If you deploy [Flux (GitOps)](extensions-release.md#flux-gitops), you can use an ARM64-based cluster without using a `linux/amd64` node.
  * If you haven't connected a cluster yet, use our [quickstart](quickstart-connect-cluster.md) to connect one.
  * [Upgrade your agents](agent-upgrade.md#manually-upgrade-agents) to the latest version.

## Create an extension instance

To create a new extension instance, use the `k8s-extension create` command. Use values from your scenario for the required parameter placeholders.

This example creates a [Container insights in Azure Monitor](extensions-release.md#container-insights-in-azure-monitor) extension instance on an Azure Arc-enabled Kubernetes cluster:

```azurecli
az k8s-extension create --name azuremonitor-containers  --extension-type Microsoft.AzureMonitor.Containers --scope cluster --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
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
> The service doesn't retain sensitive information beyond 48 hours. If Azure Arc-enabled Kubernetes agents don't have network connectivity for more than 48 hours and can't determine whether to create an extension on the cluster, the extension transitions to a `Failed` state. In that scenario, you must run `k8s-extension create` again to create a fresh extension Azure resource.
>
> Only one Container insights in Azure Monitor extension is required per cluster. Before you install Container insights via an extension, you must delete any previous Helm chart installations of Container insights that don't use extensions. Before you run `az k8s-extension create`, complete the steps to [delete the Helm chart](/azure/azure-monitor/containers/kubernetes-monitoring-disable#remove-container-insights-with-helm).

### Required parameters

The following table describes parameters that are required when you use `az k8s-extension create` to create an extension instance:

| Parameter name | Description |
|----------------|------------|
| `--name` | The name of the extension instance. |
| `--extension-type` | The [type of extension](extensions-release.md) you want to install on the cluster. For example, `Microsoft.AzureMonitor.Containers` or `microsoft.azuredefender.kubernetes`. |
| `--scope` | The [scope of installation](conceptual-extensions.md#extension-scope) for the extension. Use `cluster` or `namespace`. |
| `--cluster-name` | The name of the Azure Arc-enabled Kubernetes resource on which to create the extension instance. |
| `--resource-group` | The resource group that contains the Azure Arc-enabled Kubernetes resource. |
| `--cluster-type` | The cluster type on which to create the extension instance. For most scenarios, use `connectedClusters`, the cluster type for an Azure Arc-enabled Kubernetes cluster. |

### Optional parameters

You can use one or more of these optional parameters with the required parameters for your scenario.

> [!NOTE]
> You can choose to automatically upgrade your extension instance to the latest minor and patch versions by setting `auto-upgrade-minor-version` to `true`. You also can set the version of the extension instance manually using the `--version` parameter. We recommend enabling automatic upgrades for minor and patch versions so that you always have the latest security patches and capabilities.
>
> Because major version upgrades may include breaking changes, automatic upgrades for new major versions of an extension instance aren't supported. You can choose when to [manually upgrade an extension instances](#upgrade-an-extension-instance) to a new major version.

| Parameter name | Description |
|--------------|------------|
| `--auto-upgrade-minor-version` | A Boolean property that sets whether the extension minor version upgrades automatically. The default setting is `true`. If this parameter is set to `true`, you can't set the `version` parameter because the version is dynamically updated. If this parameter is set to `false`, the extension isn't be automatically upgraded, even for patch versions. |
| `--version` | The version of the extension to be installed (the specific version to pin the extension instance to). You can't set the `version` parameter if `auto-upgrade-minor-version` is set to `true`. |
| `--configuration-settings` | Settings that can be passed into the extension to control its functionality. These settings are passed in as space-separated `key=value` pairs after the parameter name. If this parameter is used in the command, you can't pass `--configuration-settings-file` in the same command. |
| `--configuration-settings-file` | The path to a JSON file with `key=value` pairs to be used for passing configuration settings into the extension. If this parameter is used in the command, you can't use `--configuration-settings` in the same command. |
| `--configuration-protected-settings` | Settings that aren't retrievable using `GET` API calls or `az k8s-extension show` commands. Typically used to pass in sensitive settings. These settings are passed in as space-separated `key=value` pairs after the parameter name. If this parameter is used in the command, you can't use `--configuration-protected-settings-file` in the same command. |
| `--configuration-protected-settings-file` | The path to a JSON file with `key=value` pairs to be used to pass sensitive settings into the extension. If this parameter is used in the command, you can't use `--configuration-protected-settings` in the same command. |
| `--release-namespace` | This parameter indicates the namespace in which to create the release. This parameter is relevant only if `scope` is set to `cluster`. |
| `--release-train` |  The author of an extension can publish versions in different release trains, such as `Stable` or `Preview`. If this parameter isn't set explicitly, `Stable` is the default.  |
| `--target-namespace` | Indicates the namespace within which to create the release. Permissions for the system account that's created for this extension instance is restricted to this namespace. This setting is relevant only if `scope` is set to `namespace`. |

## Show extension details

To view details of a currently installed extension instance, use the `k8s-extension show` command. In the code, use values from your scenario for the required parameter placeholders.

```azurecli
az k8s-extension show --name azuremonitor-containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
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

To view a list of all extensions that are installed on a cluster, use the `k8s-extension list` command. In the code, use values from your scenario for the required parameter placeholders.

```azurecli
az k8s-extension list --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

Check for output that looks like this example:

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
>To understand the specific settings in `--configuration-settings` and `--configuration-protected-settings` that can be updated, see the documentation for the specific extension type. For `--configuration-protected-settings`, provide all settings, even if only one setting is updated. If any of these settings are omitted, the omitted settings will be considered obsolete, and they are deleted.

To update an existing extension instance, use `k8s-extension update`. Pass in values for the mandatory and optional parameters. The mandatory and optional parameters are slightly different from the parameters that you use to create an extension instance.

This example updates the `auto-upgrade-minor-version` setting for an Azure Machine Learning extension instance to `true`:

```azurecli
az k8s-extension update --name azureml --extension-type Microsoft.AzureML.Kubernetes --scope cluster --cluster-name <clusterName> --resource-group <resourceGroupName> --auto-upgrade-minor-version true --cluster-type managedClusters
```

### Required parameters for an update

| Parameter name | Description |
|----------------|------------|
| `--name` | The name of the extension instance. |
| `--cluster-name` | The name of the cluster on which the extension instance is created. |
| `--resource-group` | The resource group that contains the cluster. |
| `--cluster-type` | The cluster type on which the extension instance has to be created. For Azure Arc-enabled Kubernetes clusters, use `connectedClusters`. For AKS clusters, use `managedClusters`.|

### Optional parameters for an update

| Parameter name | Description |
|--------------|------------|
| `--auto-upgrade-minor-version` | A Boolean property that specifies whether the extension minor version is automatically upgraded. The default setting is `true`. If this parameter is set to `true`, you can't set the `version` parameter because the version is dynamically updated. If the parameter is set to `false`, the extension isn't automatically upgraded, even for patch versions. |
| `--version` | The version of the extension to be installed (a specific version to pin the extension instance to). Must not be supplied if `auto-upgrade-minor-version` is set to `true`. |
| `--configuration-settings` | Settings that can be passed into the extension to control its functionality. These settings are passed in as space-separated `key=value` pairs after the parameter name. If the parameter is used in the command, `--configuration-settings-file` can't be used in the same command. Only the settings that require an update need to be provided. The provided settings are replaced with the specified values. |
| `--configuration-settings-file` | The path to a JSON file that contains `key=value` pairs to be used for passing in configuration settings to the extension. If this parameter is used in the command, you can't use `--configuration-settings` in the same command. |
| `--configuration-protected-settings` | Settings that aren't retrievable by using `GET` API calls or `az k8s-extension show` commands. Typically used to pass in sensitive settings. These settings are passed in as space-separated `key=value` pairs after the parameter name. If this parameter is used in the command, the `--configuration-protected-settings-file` can't be used in the same command. When you update a protected setting, configure all protected settings. If any of the settings are omitted, those settings are considered obsolete, and they're deleted.  |
| `--configuration-protected-settings-file` | The path to a JSON file that contains `key=value` pairs to be used for passing in sensitive settings to the extension. If this parameter is used in the command, you can't use `--configuration-protected-settings` in the same command. |
| `--scope` | The scope of installation for the extension. Use either `cluster` or `namespace`. |
| `--release-train` | The author of an extension can publish versions in different release trains, such as `Stable` or `Preview`. If this parameter isn't set explicitly, `Stable` is the default.  |

## Upgrade an extension instance

As noted earlier, if you set `auto-upgrade-minor-version` to true, the extension is automatically upgraded when a new minor version is released. For most scenarios, we recommend that you enable automatic upgrades. If you set `auto-upgrade-minor-version` to `false`, you must upgrade the extension manually if you want a more recent version.

Manual upgrades also are required to get a new major instance of an extension. You can choose when to upgrade to avoid any unexpected breaking changes in major version upgrades.

To manually upgrade an extension instance, use `k8s-extension update` and set the `version` parameter.

This example updates an Azure Machine Learning extension instance to version `x.y.z`:

```azurecli
az k8s-extension update --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters --name azureml --version x.y.z
```

## Delete an extension instance

To delete an extension instance on a cluster, use the `k8s-extension delete` command. Use values from your scenario for the required parameter placeholders.

```azurecli
az k8s-extension delete --name azuremonitor-containers --cluster-name <clusterName> --resource-group <resourceGroupName> --cluster-type connectedClusters
```

> [!NOTE]
> The Azure resource that represents this extension is immediately deleted. The Helm release on the cluster that's associated with this extension is deleted only when the agents running on the Kubernetes cluster have network connectivity and can reach Azure services to get the desired state.

## Related content

* For a comprehensive list of commands and parameters, review the [az k8s-extension CLI reference](/cli/azure/k8s-extension).
* Learn more about [how extensions work with Azure Arc-enabled Kubernetes clusters](conceptual-extensions.md).
* Review [cluster extensions that are available for Azure Arc-enabled Kubernetes](extensions-release.md).
* Get help with [troubleshooting extension issues](extensions-troubleshooting.md).
