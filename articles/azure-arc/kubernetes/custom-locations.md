---
title: "Create and manage custom locations on Azure Arc-enabled Kubernetes"
ms.date: 02/27/2025
ms.topic: how-to
ms.custom: references_regions, devx-track-azurecli
description: "Use custom locations to deploy Azure PaaS services on Azure Arc-enabled Kubernetes clusters"
---

# Create and manage custom locations on Azure Arc-enabled Kubernetes

 The *custom locations* feature provides a way to configure your Azure Arc-enabled Kubernetes clusters as target locations for deploying instances of Azure offerings. Examples of Azure offerings that can be deployed on top of custom locations include databases, such as SQL Managed Instance enabled by Azure Arc and Azure Arc-enabled PostgreSQL server, or application instances, such as Container Apps, Logic Apps, Event Grid, and API Management.

A [custom location](conceptual-custom-locations.md) has a one-to-one mapping to a namespace within the Azure Arc-enabled Kubernetes cluster. The custom location Azure resource combined with Azure role-based access control (Azure RBAC) can be used to grant granular permissions to application developers or database admins, enabling them to deploy resources such as databases or application instances on top of Arc-enabled Kubernetes clusters in a multitenant environment.

In this article, you learn how to enable custom locations on an Arc-enabled Kubernetes cluster, and how to create a custom location.

## Prerequisites

- [Install or upgrade Azure CLI](/cli/azure/install-azure-cli) to the latest version.

- Install the latest versions of the following Azure CLI extensions:
  - `connectedk8s`
  - `k8s-extension`
  - `customlocation`
  
    ```azurecli
    az extension add --name connectedk8s
    az extension add --name k8s-extension
    az extension add --name customlocation
    ```

    If you have already installed the `connectedk8s`, `k8s-extension`, and `customlocation` extensions, update to the **latest version** by using the following command:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-extension
    az extension update --name customlocation
    ```

- Verify completed provider registration for `Microsoft.ExtendedLocation`.

   1. Enter the following commands:

        ```azurecli
        az provider register --namespace Microsoft.ExtendedLocation
        ```

   1. Monitor the registration process. Registration may take up to 10 minutes.

        ```azurecli
        az provider show -n Microsoft.ExtendedLocation -o table
        ```

        Once registered, the `RegistrationState` state will have the `Registered` value.

- Verify you have an existing [Azure Arc-enabled Kubernetes connected cluster](quickstart-connect-cluster.md), and [upgrade your agents](agent-upgrade.md#manually-upgrade-agents) to the latest version. Confirm that the machine on which you will run the commands described in this article has a `kubeconfig` file that points to this cluster.

## Enable custom locations on your cluster

> [!IMPORTANT]
> The custom locations feature is dependent on the [cluster connect](cluster-connect.md) feature. Both features must be enabled in the cluster for custom locations to function.
> 
> The Custom Location Object ID (OID) is needed to enable custom location. If your user account has the required permissions, the OID is automatically retrieved during feature enablement. If you do not have a valid user account, then the manually passed OID is used but the OID can't be validated. If the OID is invalid, then custom location may not be properly enabled. 

### To enable the custom locations feature as a Microsoft Entra user, follow the steps below:

1. Sign into Azure CLI as a Microsoft Entra user and run the following command:

```azurecli
az connectedk8s enable-features -n <clusterName> -g <resourceGroupName> --features cluster-connect custom-locations
```


### To enable the custom locations feature with a service principal, follow the steps below:

Manually retrieve the custom location OID by following these steps:

1. Sign in to Azure CLI as a Microsoft Entra user.

1. Run the following command to fetch the custom location `oid` (object ID), where `--id` refers to the Custom Location service app itself, and is predefined and set to `bc313c14-388c-4e7d-a58e-70017303ee3b`: 

   **Important!** Copy and run the command exactly as it is shown below. Do not replace the value passed to the `--id` parameter with a different value.

   ```azurecli
   az ad sp show --id bc313c14-388c-4e7d-a58e-70017303ee3b --query id -o tsv
   ```

1. Sign in to Azure CLI using the service principal. Run the following command to enable the custom locations feature on the cluster, using the `oid` (object ID) value from the previous step for the `--custom-locations-oid` parameter:

    ```azurecli
    az connectedk8s enable-features -n <cluster-name> -g <resource-group-name> --custom-locations-oid <cl-oid> --features cluster-connect custom-locations
    ```

## Create custom location

1. Deploy the Azure service cluster extension of the Azure service instance you want to install on your cluster:

   - [Azure Arc-enabled data services](../data/create-data-controller-direct-prerequisites.md)

     > [!NOTE]
     > Outbound proxy without authentication and outbound proxy with basic authentication are supported by the Azure Arc-enabled data services cluster extension. Outbound proxy that expects trusted certificates is currently not supported.

   - [Azure Container Apps on Azure Arc](/azure/container-apps/azure-arc-enable-cluster?tabs=azure-cli#install-the-container-apps-extension)

   - [Event Grid on Kubernetes](/azure/event-grid/kubernetes/install-k8s-extension)

1. Get the Azure Resource Manager identifier of the Azure Arc-enabled Kubernetes cluster, referenced in later steps as `connectedClusterId`:

    ```azurecli
    az connectedk8s show -n <clusterName> -g <resourceGroupName>  --query id -o tsv
    ```

1. Get the Azure Resource Manager identifier of the cluster extension you deployed to the Azure Arc-enabled Kubernetes cluster, referenced in later steps as `extensionId`:

    ```azurecli
    az k8s-extension show --name <extensionInstanceName> --cluster-type connectedClusters -c <clusterName> -g <resourceGroupName>  --query id -o tsv
    ```

1. Create the custom location by referencing the Azure Arc-enabled Kubernetes cluster and the extension:

    ```azurecli
    az customlocation create -n <customLocationName> -g <resourceGroupName> --namespace <name of namespace> --host-resource-id <connectedClusterId> --cluster-extension-ids <extensionId> 
    ```

   - Required parameters:

     | Parameter name | Description |
     |----------------|------------|
     | `--name, --n` | Name of the custom location. |
     | `--resource-group, --g` | Resource group of the custom location.  |
     | `--namespace` | Namespace in the cluster bound to the custom location being created. |
     | `--host-resource-id` | Azure Resource Manager identifier of the Azure Arc-enabled Kubernetes cluster (connected cluster). |
     | `--cluster-extension-ids` | Azure Resource Manager identifier of a cluster extension instance installed on the connected cluster. For multiple extensions, provide a space-separated list of cluster extension IDs |

   - Optional parameters:

     | Parameter name | Description |
     |--------------|------------|
     | `--location, --l` | Location of the custom location Azure Resource Manager resource in Azure. If not specified, the location of the connected cluster is used. |
     | `--tags` | Space-separated list of tags in the format `key[=value]`. Use '' to clear existing tags. |
     | `--kubeconfig` | Admin `kubeconfig` of cluster. |

1. Confirm that custom location was successfully enabled by running the following command and checking that `ProvisioningState` is `Succeeded`:

```azurecli
az customlocation show -n <customLocationName> -g <resourceGroupName>
```

## Show details of a custom location

To show the details of a custom location, use the following command:

```azurecli
az customlocation show -n <customLocationName> -g <resourceGroupName> 
```

## List custom locations

To list all custom locations in a resource group, use the following command:

```azurecli
az customlocation list -g <resourceGroupName> 
```

## Update a custom location

Use the `update` command to add new values for `--tags` or associate new `--cluster-extension-ids` to the custom location, while retaining existing values for tags and associated cluster extensions.

```azurecli
az customlocation update -n <customLocationName> -g <resourceGroupName> --namespace <name of namespace> --host-resource-id <connectedClusterId> --cluster-extension-ids <extensionIds> 
```

## Patch a custom location

Use the `patch` command to replace existing values for `--cluster-extension-ids` or `--tags`. Previous values are not retained.

```azurecli
az customlocation patch -n <customLocationName> -g <resourceGroupName> --namespace <name of namespace> --host-resource-id <connectedClusterId> --cluster-extension-ids <extensionIds> 
```

## Delete a custom location

To delete a custom location, use the following command:

```azurecli
az customlocation delete -n <customLocationName> -g <resourceGroupName> 
```

## Troubleshooting

If custom location creation fails with the error `Unknown proxy error occurred`, modify your network policy to allow pod-to-pod internal communication within the `azure-arc` namespace. Be sure to also add the `azure-arc` namespace as part of the no-proxy exclusion list for your configured policy.

If you try to enable custom location while logged into Azure CLI using a service principal, you may observe the following warning:

```console
Unable to fetch oid of 'custom-locations' app. Proceeding without enabling the feature. Insufficient privileges to complete the operation.
```
This warning occurs because the service principal lacks the necessary permissions to retrieve the `oid` (object ID) of the custom location used by the Azure Arc service. Follow the instructions provided above to enable the custom location feature using a service principal. 

## Next steps

- Securely connect to the cluster using [Cluster Connect](cluster-connect.md).
- Continue with [Azure Container Apps on Azure Arc](/azure/container-apps/azure-arc-overview) for end-to-end instructions on installing extensions, creating custom locations, and creating the Container Apps connected environment.
- Create an Event Grid topic and an event subscription for [Event Grid on Kubernetes](/azure/event-grid/kubernetes/overview).
- Learn more about currently available [Azure Arc-enabled Kubernetes extensions](extensions-release.md).
