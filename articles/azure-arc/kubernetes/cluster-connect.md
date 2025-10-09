---
title: "Use cluster connect to securely connect to Azure Arc-enabled Kubernetes clusters."
ms.date: 01/24/2025
ms.topic: how-to
ms.custom: devx-track-azurecli
description: "With cluster connect, you can securely connect to Azure Arc-enabled Kubernetes clusters from anywhere without requiring any inbound port to be enabled on the firewall."
# Customer intent: As an IT administrator managing Azure Arc-enabled Kubernetes clusters, I want to securely connect to the cluster without requiring firewall adjustments, so that I can perform debugging and access Azure services seamlessly from any location.
---

# Use cluster connect to securely connect to Azure Arc-enabled Kubernetes clusters

With cluster connect, you can securely connect to Azure Arc-enabled Kubernetes clusters from anywhere without requiring any inbound port to be enabled on the firewall.

Access to the `apiserver` of the Azure Arc-enabled Kubernetes cluster enables the following scenarios:

- Interactive debugging and troubleshooting.
- Cluster access to Azure services for [custom locations](custom-locations.md) and other resources created on top of it.

Before you begin, review the [conceptual overview of the cluster connect feature](conceptual-cluster-connect.md).

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).

- An existing Azure Arc-enabled Kubernetes connected cluster.
  - If you haven't connected a cluster yet, use our [quickstart](quickstart-connect-cluster.md).
  - [Upgrade your agents](agent-upgrade.md#manually-upgrade-agents) to the latest version.
  
- Enable the [network requirements for Arc-enabled Kubernetes](network-requirements.md), including all endpoints listed as required for cluster connect.

### [Azure CLI](#tab/azure-cli)

- [Install](/cli/azure/install-azure-cli) or [update](/cli/azure/update-azure-cli) Azure CLI to the latest version.

- Install the latest version of the `connectedk8s` Azure CLI extension:

  ```azurecli
  az extension add --name connectedk8s
  ```

  If you've already installed the `connectedk8s` extension, update the extension to the latest version:

  ```azurecli
  az extension update --name connectedk8s
  ```

- Replace the placeholders and run the below command to set the environment variables used in this document:

  ```azurecli
  CLUSTER_NAME=<cluster-name>
  RESOURCE_GROUP=<resource-group-name>
  ARM_ID_CLUSTER=$(az connectedk8s show -n $CLUSTER_NAME -g $RESOURCE_GROUP --query id -o tsv)
  ```

### [Azure PowerShell](#tab/azure-powershell)

- Install [the latest version of Azure PowerShell](/powershell/azure/install-azure-powershell).

- Replace the placeholders and run the below command to set the environment variables used in this document:

  ```azurepowershell
  $CLUSTER_NAME = <cluster-name>
  $RESOURCE_GROUP = <resource-group-name>
  $ARM_ID_CLUSTER = (Get-AzConnectedKubernetes -ResourceGroupName $RESOURCE_GROUP -Name $CLUSTER_NAME).Id
  ```

---

## Set up authentication

On the existing Arc-enabled cluster, create the ClusterRoleBinding with either Microsoft Entra authentication or service account token.

<a name='azure-active-directory-authentication-option'></a>

### Microsoft Entra authentication option

#### [Azure CLI](#tab/azure-cli)

1. Get the `objectId` associated with your Microsoft Entra entity. For single user accounts, get the user principal name (UPN) associated with your Microsoft Entra entity.

   - For a Microsoft Entra group account:

     ```azurecli
     AAD_ENTITY_ID=$(az ad signed-in-user show --query id -o tsv)
     ```

   - For a Microsoft Entra single user account:

     ```azurecli
     AAD_ENTITY_ID=$(az ad signed-in-user show --query userPrincipalName -o tsv)
     ```

   - For a Microsoft Entra application:

     ```azurecli
     AAD_ENTITY_ID=$(az ad sp show --id <id> --query id -o tsv)
     ```

1. Authorize the entity with appropriate permissions.

   - If you use Kubernetes native ClusterRoleBinding or RoleBinding for authorization checks on the cluster, with the `kubeconfig` file pointing to the `apiserver` of your cluster for direct access, you can create one mapped to the Microsoft Entra entity that needs to access this cluster. For example:

      ```console
      kubectl create clusterrolebinding demo-user-binding --clusterrole cluster-admin --user=$AAD_ENTITY_ID
      ```

   - If you use Azure RBAC for authorization checks on the cluster, you can create an applicable [Azure role assignment](azure-rbac.md#built-in-roles) mapped to the Microsoft Entra entity. For example:

     ```azurecli
     az role assignment create --role "Azure Arc Kubernetes Viewer" --assignee $AAD_ENTITY_ID --scope $ARM_ID_CLUSTER
     az role assignment create --role "Azure Arc Enabled Kubernetes Cluster User Role" --assignee $AAD_ENTITY_ID --scope $ARM_ID_CLUSTER
     ```

#### [Azure PowerShell](#tab/azure-powershell)

1. Get the `objectId` associated with your Microsoft Entra entity. For single user accounts, get the user principal name (UPN) associated with your Microsoft Entra entity.

   - For a Microsoft Entra group account:

     ```azurepowershell
     $AAD_ENTITY_ID = (az ad signed-in-user show --query id -o tsv)
     ```

   - For a Microsoft Entra single user account:

     ```azurepowershell
     $AAD_ENTITY_ID = (az ad signed-in-user show --query userPrincipalName -o tsv)
     ```

   - For a Microsoft Entra application:

     ```azurepowershell
     $AAD_ENTITY_ID = (az ad sp show --id <id> --query objectId -o tsv)
     ```

1. Authorize the entity with appropriate permissions.

   - If you use native Kubernetes ClusterRoleBinding or RoleBinding for authorization checks on the cluster, with the `kubeconfig` file pointing to the `apiserver` of your cluster for direct access, you can create one mapped to the Microsoft Entra entity that needs to access this cluster. For example:

     ```console
     kubectl create clusterrolebinding demo-user-binding --clusterrole cluster-admin --user=$AAD_ENTITY_ID
     ```

   - If you use [Azure RBAC for authorization checks](azure-rbac.md) on the cluster, you can create an applicable [Azure role assignment](azure-rbac.md#built-in-roles) mapped to the Microsoft Entra entity. For example:

     ```azurepowershell
     az role assignment create --role "Azure Arc Kubernetes Viewer" --assignee $AAD_ENTITY_ID --scope $ARM_ID_CLUSTER
     az role assignment create --role "Azure Arc Enabled Kubernetes Cluster User Role" --assignee $AAD_ENTITY_ID --scope $ARM_ID_CLUSTER
     ```

---

### Service account token authentication option

#### [Azure CLI](#tab/azure-cli)

1. With the `kubeconfig` file pointing to the `apiserver` of your Kubernetes cluster, run this command to create a service account. This example creates the service account in the default namespace, but you can substitute any other namespace for `default`.

   ```console
   kubectl create serviceaccount demo-user -n default
   ```

1. Create a ClusterRoleBinding to [grant this service account the appropriate permissions on the cluster](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-rolebinding). If you used a different namespace in the first command, substitute it here for `default`.

    ```console
    kubectl create clusterrolebinding demo-user-binding --clusterrole cluster-admin --serviceaccount default:demo-user
    ```

1. Create a service account token:

    ```console
    kubectl apply -f - <<EOF
    apiVersion: v1
    kind: Secret
    metadata:
      name: demo-user-secret
      annotations:
        kubernetes.io/service-account.name: demo-user
    type: kubernetes.io/service-account-token
    EOF
    ```

    ```console
    TOKEN=$(kubectl get secret demo-user-secret -o jsonpath='{$.data.token}' | base64 -d | sed 's/$/\n/g')
    ```

1. Get the token to output to console
  
     ```console
     echo $TOKEN
     ```

#### [Azure PowerShell](#tab/azure-powershell)

1. With the `kubeconfig` file pointing to the `apiserver` of your Kubernetes cluster, run this command to create a service account. This example creates the service account in the default namespace, but you can substitute any other namespace for `default`.

   ```console
   kubectl create serviceaccount demo-user -n default
   ```

1. Create a ClusterRoleBinding or RoleBinding to [grant this service account the appropriate permissions on the cluster](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-rolebinding). If you used a different namespace in the first command, substitute it here for `default`.

    ```console
    kubectl create clusterrolebinding demo-user-binding --clusterrole cluster-admin --serviceaccount default:demo-user
    ```

1. Create a service account token. Create a `demo-user-secret.yaml` file with the following content:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: demo-user-secret
      annotations:
        kubernetes.io/service-account.name: demo-user
    type: kubernetes.io/service-account-token
    ```

   Then run these commands:

    ```console
    kubectl apply -f demo-user-secret.yaml
    ```

    ```console
    $TOKEN = ([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String((kubectl get secret demo-user-secret -o jsonpath='{$.data.token}'))))
    ```

1. Get the token to output to console.
  
     ```console
     echo $TOKEN
     ```

---

## Access your cluster from a client device

Now you can access the cluster from a different client. Run the following steps on another client device.

1. Sign in using either Microsoft Entra authentication or service account token authentication.

1. Get the cluster connect `kubeconfig` needed to communicate with the cluster from anywhere (even outside the firewall surrounding the cluster), based on the authentication option used:

   - For Microsoft Entra authentication:

     ```azurecli
     az connectedk8s proxy -n $CLUSTER_NAME -g $RESOURCE_GROUP
     ```

   - For service account token authentication:

     ```azurecli
     az connectedk8s proxy -n $CLUSTER_NAME -g $RESOURCE_GROUP --token $TOKEN
     ```

     > [!NOTE]
     > This command opens the proxy and blocks the current shell.

1. In a different shell session, use `kubectl` to send requests to the cluster:

   ```powershell
   kubectl get pods -A
   ```

You should now see a response from the cluster containing the list of all pods under the `default` namespace.

## Known limitations

When making requests to the Kubernetes cluster, if the Microsoft Entra service principal used is a part of more than 200 groups, you might see the following error:

`Overage claim (users with more than 200 group membership) for SPN is currently not supported. For troubleshooting, please refer to aka.ms/overageclaimtroubleshoot`

This is a known limitation. To get past this error:

1. Create a [service principal](/cli/azure/create-an-azure-service-principal-azure-cli), which is less likely to be a member of more than 200 groups.
1. [Sign in](/cli/azure/create-an-azure-service-principal-azure-cli#sign-in-using-a-service-principal) to Azure CLI with the service principal before running the `az connectedk8s proxy` command.

## Next steps

- Set up [Microsoft Entra RBAC](azure-rbac.md) on your clusters.
- Deploy and manage [cluster extensions](extensions.md).
- Help to protect your cluster in other ways by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).
