---
title: "Azure RBAC on Azure Arc-enabled Kubernetes clusters"
ms.date: 02/25/2025
ms.topic: how-to
description: "Use Azure RBAC for authorization checks on Azure Arc-enabled Kubernetes clusters."
ms.custom:
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: As a Kubernetes administrator, I want to implement Azure RBAC on my Azure Arc-enabled clusters, so that I can effectively manage user access and authorizations for Kubernetes objects within my environment.
---

# Use Azure RBAC on Azure Arc-enabled Kubernetes clusters

You can use [Microsoft Entra ID ](/entra/fundamentals/whatis)and [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview) to control authorization checks on your Azure Arc-enabled Kubernetes cluster. Azure role assignments let you granularly control which users can read, write, and delete Kubernetes objects such as deployment, pod, and service. Kubernetes [ClusterRoleBinding and RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) object types help to define authorization in Kubernetes natively.

For a conceptual overview of this feature, see [Azure RBAC on Azure Arc-enabled Kubernetes](conceptual-azure-rbac.md).

## Prerequisites

- [Install or upgrade the Azure CLI](/cli/azure/install-azure-cli) to the latest version.

- Install the latest version of the `connectedk8s` Azure CLI extension:

    ```azurecli
    az extension add --name connectedk8s
    ```

    If the `connectedk8s` extension is already installed, you can update it to the latest version by using the following command:

    ```azurecli
    az extension update --name connectedk8s
    ```

- Connect an existing Azure Arc-enabled Kubernetes cluster:
  - If you haven't connected a cluster yet, use our [quickstart](quickstart-connect-cluster.md).
  - [Upgrade your agents](agent-upgrade.md#manually-upgrade-agents) to the latest version.

> [!NOTE]
> Azure RBAC isn't supported for Red Hat OpenShift or managed Kubernetes offerings where user access to the API server is restricted (such as Amazon Elastic Kubernetes Service (EKS) or Google Kubernetes Engine (GKE)).
>
> Azure RBAC doesn't currently support Kubernetes clusters operating on Arm64 architecture. For these clusters, use [Kubernetes RBAC](identity-access-overview.md#kubernetes-rbac-authorization) to manage access control.
>
> For Azure Kubernetes Service (AKS) clusters, this [feature is available natively](/azure/aks/manage-azure-rbac) and doesn't require the AKS cluster to be connected to Azure Arc.
>
> For Azure Kubernetes Service (AKS) clusters enabled by Azure Arc on Azure Local, version 23H2, Azure RBAC is currently supported only if enabled when the clusters are created. To create an AKS cluster enabled by Azure Arc with Azure RBAC enabled, see [Use Azure RBAC for Kubernetes authorization](/azure/aks/hybrid/azure-rbac-23h2). Azure RBAC isn't supported for Azure Local, version 22H2.

## Enable Azure RBAC on the cluster

1. Get the cluster MSI identity by running the following command:

   ```azurecli
   az connectedk8s show -g <resource-group> -n <connected-cluster-name>
   ```

1. Get the ID (`identity.principalId`) from the output and run the following command to assign the **Connected Cluster Managed Identity CheckAccess Reader** role to the cluster MSI:

   ```azurecli
   az role assignment create --role "Connected Cluster Managed Identity CheckAccess Reader" --assignee "<Cluster MSI ID>" --scope <cluster ARM ID>
   ```

1. Enable Azure role-based access control (RBAC) on your Azure Arc-enabled Kubernetes cluster by running the following command:

   ```azurecli
   az connectedk8s enable-features -n <clusterName> -g <resourceGroupName> --features azure-rbac
   ```

   > [!NOTE]
   > Before you run the `enable-features` command, ensure that the `kubeconfig` file on the machine points to the cluster on which you want to enable Azure RBAC.
   >
   > Use `--skip-azure-rbac-list` with this command for a comma-separated list of usernames, emails, and OpenID connections undergoing authorization checks by using Kubernetes native `ClusterRoleBinding` and `RoleBinding` objects instead of Azure RBAC.

Next, follow the steps in the appropriate section, depending on whether you're using a generic cluster where no reconciler is running on the `apiserver` specification, or a cluster created by using Cluster API.

### Generic cluster where no reconciler is running on the apiserver specification

1. SSH into every master node of the cluster, then complete the following steps:

    **If your `kube-apiserver` is a [static pod](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/):**

    1. The `azure-arc-guard-manifests` secret in the `kube-system` namespace contains two files: `guard-authn-webhook.yaml` and `guard-authz-webhook.yaml`. Copy these files to the `/etc/guard` directory of the node.

        ```console
        sudo mkdir -p /etc/guard
        kubectl get secrets azure-arc-guard-manifests -n kube-system -o json | jq -r '.data."guard-authn-webhook.yaml"' | base64 -d > /etc/guard/guard-authn-webhook.yaml
        kubectl get secrets azure-arc-guard-manifests -n kube-system -o json | jq -r '.data."guard-authz-webhook.yaml"' | base64 -d > /etc/guard/guard-authz-webhook.yaml
        ```

    1. Open the `apiserver` manifest in edit mode:

       ```console
       sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
       ```

    1. Add the following specification under `volumes`:

       ```yml
       - hostPath:
           path: /etc/guard
           type: Directory
         name: azure-rbac
       ```

    1. Add the following specification under `volumeMounts`:

       ```yml
       - mountPath: /etc/guard
         name: azure-rbac
         readOnly: true
       ```

    **If your `kube-apiserver` isn't a static pod:**

    1. Open the `apiserver` manifest in edit mode:

       ```console
       sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
       ```

    1. Add the following specification under `volumes`:

       ```yml
       - name: azure-rbac
         secret:
           secretName: azure-arc-guard-manifests
       ```

    1. Add the following specification under `volumeMounts`:

       ```yml
       - mountPath: /etc/guard
         name: azure-rbac
         readOnly: true
       ```

1. Add the following `apiserver` arguments:

   ```yml
   - --authentication-token-webhook-config-file=/etc/guard/guard-authn-webhook.yaml
   - --authentication-token-webhook-cache-ttl=5m0s
   - --authorization-webhook-cache-authorized-ttl=5m0s
   - --authorization-webhook-config-file=/etc/guard/guard-authz-webhook.yaml
   - --authorization-webhook-version=v1
   - --authorization-mode=Node,RBAC,Webhook
   ```

   Set the following `apiserver` argument:

   ```yml
   - --authentication-token-webhook-version=v1
   ```

1. Save and close the editor to update the `apiserver` pod.

### Cluster created by using Cluster API

1. Copy the guard secret that contains authentication and authorization webhook configuration files from the workload cluster onto your machine:

   ```console
   kubectl get secret azure-arc-guard-manifests -n kube-system -o yaml > azure-arc-guard-manifests.yaml
   ```

1. Change the `namespace` field in the *azure-arc-guard-manifests.yaml* file to the namespace within the management cluster where you're applying the custom resources for creation of workload clusters.

1. Apply this manifest:

   ```console
   kubectl apply -f azure-arc-guard-manifests.yaml
   ```

1. Edit the `KubeadmControlPlane` object by running `kubectl edit kcp <clustername>-control-plane`:

   1. Add the following specification under `files`:

      ```yml
      - contentFrom:
          secret:
            key: guard-authn-webhook.yaml
            name: azure-arc-guard-manifests
        owner: root:root
        path: /etc/kubernetes/guard-authn-webhook.yaml
        permissions: "0644"
      - contentFrom:
          secret:
            key: guard-authz-webhook.yaml
            name: azure-arc-guard-manifests
        owner: root:root
        path: /etc/kubernetes/guard-authz-webhook.yaml
        permissions: "0644"
      ```

   1. Add the following specification under `apiServer` > `extraVolumes`:

      ```yml
      - hostPath: /etc/kubernetes/guard-authn-webhook.yaml
        mountPath: /etc/guard/guard-authn-webhook.yaml
        name: guard-authn
        readOnly: true
      - hostPath: /etc/kubernetes/guard-authz-webhook.yaml
        mountPath: /etc/guard/guard-authz-webhook.yaml
        name: guard-authz
        readOnly: true
      ```

   1. Add the following specification under `apiServer` > `extraArgs`:

      ```yml
      authentication-token-webhook-cache-ttl: 5m0s
      authentication-token-webhook-config-file: /etc/guard/guard-authn-webhook.yaml
      authentication-token-webhook-version: v1
      authorization-mode: Node,RBAC,Webhook
      authorization-webhook-cache-authorized-ttl: 5m0s
      authorization-webhook-config-file: /etc/guard/guard-authz-webhook.yaml
      authorization-webhook-version: v1
      ```

   1. Save and close to update the `KubeadmControlPlane` object. Wait for these changes to appear on the workload cluster.

## Create role assignments for users to access the cluster

Owners of the Azure Arc-enabled Kubernetes resource can use either built-in roles or custom roles to grant other users access to the Kubernetes cluster.

### Built-in roles

The following built-in roles provide access to perform common tasks on Kubernetes clusters. These roles can be granted to Microsoft Entra ID users, groups, or service principals.

| Role | Description |
|---|---|
| [Azure Arc Kubernetes Viewer](/azure/role-based-access-control/built-in-roles#azure-arc-kubernetes-viewer) | Allows read-only access to see most objects in a namespace. This role doesn't allow viewing secrets, because `read` permission on secrets would enable access to `ServiceAccount` credentials in the namespace. These credentials would in turn allow API access through that `ServiceAccount` value (a form of privilege escalation). |
| [Azure Arc Kubernetes Writer](/azure/role-based-access-control/built-in-roles#azure-arc-kubernetes-writer) | Allows read/write access to most objects in a namespace. This role doesn't allow viewing or modifying roles or role bindings. However, this role allows accessing secrets and running pods as any `ServiceAccount` value in the namespace, so it can be used to gain the API access levels of any `ServiceAccount` value in the namespace. |
| [Azure Arc Kubernetes Admin](/azure/role-based-access-control/built-in-roles#azure-arc-kubernetes-admin) | Allows admin access. This role is often granted within a namespace through [the `RoleBinding` object](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding). If you use it in `RoleBinding`, it allows read/write access to most resources in a namespace, including the ability to create roles and role bindings within the namespace. However, this role doesn't allow write access to resource quota or to the namespace itself. |
| [Azure Arc Kubernetes Cluster Admin](/azure/role-based-access-control/built-in-roles#azure-arc-kubernetes-cluster-admin) | Allows the ability to execute any action on any resource within the granted scope. When you use it in [`ClusterRoleBinding`](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding), it allows full control over every resource in the cluster and in all namespaces. When you use it in [`RoleBinding`](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding), it allows full control over every resource in the role binding's namespace, including the namespace itself.|

You can create built-in role assignments scoped to the cluster by using either the Azure portal or the Azure CLI. However, only Azure CLI can be used to create role assignments scoped to namespaces.

To create role assignments scoped to the Azure Arc-enabled Kubernetes cluster in the Azure portal, navigate to the cluster and then select **Access Control (IAM)** from the service menu.

To create role assignments by using Azure CLI, use the following command:

```azurecli
az role assignment create --role "Azure Arc Kubernetes Cluster Admin" --assignee <AZURE-AD-ENTITY-ID> --scope $ARM_ID
```

`AZURE-AD-ENTITY-ID` can be a username (for example, `testuser@mytenant.onmicrosoft.com`) or the `appId` value of a service principal.

To create a role assignment scoped to a specific namespace within the cluster, modify the scope:

```azurecli
az role assignment create --role "Azure Arc Kubernetes Viewer" --assignee <AZURE-AD-ENTITY-ID> --scope $ARM_ID/namespaces/<namespace-name>
```

### Custom roles

You can choose to create your own role definition for use in role assignments. For more information, see [the full list of data actions that you can use to construct a role definition](/azure/role-based-access-control/resource-provider-operations#microsoftkubernetes).

The following example shows a custom role definition that allows a user to read deployments, but provides no other access. The custom role uses one of the data actions and lets you view all deployments in the scope (cluster or namespace) where the role assignment is created.

```json
{
    "Name": "Arc Deployment Viewer",
    "Description": "Lets you view all deployments in cluster/namespace.",
    "Actions": [],
    "NotActions": [],
    "DataActions": [
        "Microsoft.Kubernetes/connectedClusters/apps/deployments/read"
    ],
    "NotDataActions": [],
    "assignableScopes": [
        "/subscriptions/<subscription-id>"
    ]
}
```

To use this role definition, copy the following JSON object into a file called *custom-role.json*. Replace the `<subscription-id>` placeholder with the actual subscription ID. Then, complete these steps:

1. Create the role definition by running the following command from the folder where you saved *custom-role.json*:

   ```azurecli
   az role definition create --role-definition @custom-role.json
   ```

1. Create a role assignment to assign this custom role definition:

   ```azurecli
   az role assignment create --role "Arc Deployment Viewer" --assignee <AZURE-AD-ENTITY-ID> --scope $ARM_ID/namespaces/<namespace-name>
   ```

## Configure kubectl with user credentials

There are two ways to get the *kubeconfig* file that you need to access the cluster:

- Use the [cluster connect](cluster-connect.md) feature (`az connectedk8s proxy`) of the Azure Arc-enabled Kubernetes cluster.
- The cluster admin can share the *kubeconfig* file with every user.

### Use cluster connect

Run the following command to start the proxy process:

```azurecli
az connectedk8s proxy -n <clusterName> -g <resourceGroupName>
```

After the proxy process is running, you can open another tab in your console to [start sending your requests to the cluster](#send-requests-to-the-cluster).

### Use a shared kubeconfig file

1. Run the following command to set the credentials for the user. Specify `serverApplicationId` as `6256c85f-0aad-4d50-b960-e6e9b21efe35` and `clientApplicationId` as `3f4439ff-e698-4d6d-84fe-09c9d574f06b`:

   ```console
   kubectl config set-credentials <testuser>@<mytenant.onmicrosoft.com> \
   --auth-provider=azure \
   --auth-provider-arg=environment=AzurePublicCloud \
   --auth-provider-arg=client-id=<clientApplicationId> \
   --auth-provider-arg=tenant-id=<tenantId> \
   --auth-provider-arg=apiserver-id=<serverApplicationId>
   ```

1. Open the *kubeconfig* file that you created earlier. Under `contexts`, verify that the context associated with the cluster points to the user credentials that you created in the previous step. To set the current context to these user credentials, run the following command:

   ```console
   kubectl config set-context --current=true --user=<testuser>@<mytenant.onmicrosoft.com>
   ```

1. Add the **config-mode** setting under `user` > `config`:
  
   ```console
   name: testuser@mytenant.onmicrosoft.com
   user:
       auth-provider:
       config:
           apiserver-id: $SERVER_APP_ID
           client-id: $CLIENT_APP_ID
           environment: AzurePublicCloud
           tenant-id: $TENANT_ID
           config-mode: "1"
       name: azure
   ```

1. [Exec plugin](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins) is a Kubernetes authentication strategy that allows `kubectl` to execute an external command to receive user credentials to send to `apiserver`. Starting with Kubernetes version 1.26, in order to use the exec plugin to receive user credentials, you must use [Azure kubelogin](https://azure.github.io/kubelogin/index.html), a `client-go` credential (exec) plugin that implements Azure authentication. To install Azure kubelogin:

   - For Windows or Mac, follow the [Azure kubelogin installation instructions](https://azure.github.io/kubelogin/install.html#installation).
   - For Linux or Ubuntu, download the [latest version of kubelogin](https://github.com/Azure/kubelogin/releases), then run the following commands:

     ```bash
     curl -LO https://github.com/Azure/kubelogin/releases/download/"$KUBELOGIN_VERSION"/kubelogin-linux-amd64.zip 

     unzip kubelogin-linux-amd64.zip 

     sudo mv bin/linux_amd64/kubelogin /usr/local/bin/ 

     sudo chmod +x /usr/local/bin/kubelogin 
     ```

1. Kubelogin can be used to authenticate with Azure Arc-enabled clusters by requesting a proof-of-possession (PoP) token. [Convert](https://azure.github.io/kubelogin/concepts/azure-arc.html) the kubeconfig using kubelogin to use the appropriate [login mode](https://azure.github.io/kubelogin/concepts/login-modes.html). Below are the commands for interactive login or device code login with an Entra user account:

For [interactive login with a Microsoft Entra user](https://azure.github.io/kubelogin/concepts/login-modes/interactive.html), the command would be as follows:

   ```bash
   export KUBECONFIG=/path/to/kubeconfig

   kubelogin convert-kubeconfig -l interactive --pop-enabled --pop-claims "u=/ARM/ID/OF/CLUSTER"
   ```

For [device code login](https://azure.github.io/kubelogin/concepts/login-modes/devicecode.html) with a Microsoft Entra user, the commands would be as follows:

   ```bash
   export KUBECONFIG=/path/to/kubeconfig

   kubelogin convert-kubeconfig --pop-enabled --pop-claims 'u=<ARM ID of cluster>"
   ```

## Send requests to the cluster

1. Run any `kubectl` command. For example:

   - `kubectl get nodes`
   - `kubectl get pods`

1. After you're prompted for browser-based authentication, copy the device login URL (`https://microsoft.com/devicelogin`) and open it in your web browser.

1. Enter the code printed on your console. Copy and paste the code on your terminal into the prompt for device authentication input.

1. Enter the username (`testuser@mytenant.onmicrosoft.com`) and the associated password.

1. If you see an error message that says the users doesn't have access to the resource in Azure, it means you're unauthorized to access the requested resource. In this case, an administrator in your Azure tenant needs to create a new role assignment that authorizes this user to have access on the resource.

<a name='use-conditional-access-with-azure-ad'></a>

## Use Conditional Access with Microsoft Entra ID

When you're integrating Microsoft Entra ID with your Azure Arc-enabled Kubernetes cluster, you can also use [Conditional Access](/azure/active-directory/conditional-access/overview) to control access to your cluster.

> [!NOTE]
> [Microsoft Entra Conditional Access](/azure/active-directory/conditional-access/overview) is a Microsoft Entra ID P2 capability. For more information about Microsoft Entra ID SKUs, see the [pricing guide](https://www.microsoft.com/security/business/microsoft-entra-pricing).

To create an example Conditional Access policy to use with the cluster:

1. At the top of the Azure portal, search for and select **Microsoft Entra ID**.
1. In the service menu, under **Manage**, select **Enterprise applications**.
1. In the service menu, under **Security**, select **Conditional Access**.
1. In the service menu, select **Policies**. Then select **Create new policy**.
1. Enter a name for the policy, such as `arc-k8s-policy`.
1. Under **Assignments**, select the current value under **Users or workload identities**. Then, under **What does this policy apply to?**, verify that **Users and groups** is selected.
1. Under **Include**, choose **Select users and groups**. Then choose the users and groups where you want to apply the policy. For this example, choose the same Microsoft Entra group that has administrative access to your cluster.
1. Select the current value under **Cloud apps or actions**. Then, under **Select what this policy applies to**, verify that **Cloud apps** is selected.
1. Under **Include**, choose **Select resources**. Then search for and select the server application that you created earlier.
1. Under **Access controls**, select the current value under **Grant**. Then, select **Grant access**.
1. Check the box for **Require device to be marked as compliant**, then select **Select**.
1. Under **Enable policy**, select **On**.
1. To apply the Conditional Access policy, select **Create**.

Access the cluster again. For example, run the `kubectl get nodes` command to view nodes in the cluster:

```console
kubectl get nodes
```

To confirm that the policy is applied correctly, follow the instructions to sign in again. An error message states that you're successfully logged in, but your admin requires the device that's requesting access to be managed by Microsoft Entra ID in order to access the resource. Follow these steps to view more details:

1. In the Azure portal, go to **Microsoft Entra ID**.
1. In the service menu, under **Manage**, select **Enterprise applications**.
1. In the service menu, under **Activity**, select **Sign-in logs**.
1. Select the entry at the top that shows **Failed** for **Status** and **Success** for **Conditional Access**. Then, under **Details**, select **Conditional Access.** You'll see the Conditional Access policy that you created, requiring that your device must be compliant.

<a name='configure-just-in-time-cluster-access-with-azure-ad'></a>

## Configure just-in-time cluster access with Microsoft Entra ID

Another option for cluster access control is [Privileged Identity Management (PIM)](/azure/active-directory/privileged-identity-management/pim-configure), which enables a higher level of access for users for just-in-time requests.

>[!NOTE]
> [Microsoft Entra PIM](/azure/active-directory/privileged-identity-management/pim-configure) is a Microsoft Entra ID P2 capability. For more information about Microsoft Entra ID SKUs, see the [pricing guide](https://www.microsoft.com/security/business/microsoft-entra-pricing).

To configure just-in-time access requests for a group of users, complete the following steps:

1. At the top of the Azure portal, search for and select **Microsoft Entra ID**.
1. In the service menu, under **Manage**, select **Groups**. Then select **New group**.
1. For **Group type**, verify that **Security** is selected. Enter a group name, such as `myJITGroup`. Make any additional selections, then select **Create**.

    :::image type="content" source="media/azure-rbac/jit-new-group-created.png" alt-text="Screenshot showing details for the new group in the Azure portal.":::

1. You're brought back to the **Groups** page. Search for and select your newly created group.
1. In the service menu, under **Activity**, select **Privileged Identity Management**. Then select **Enable PIM for this group**.
1. Select **Add assignments** to begin granting access.
1. Under **Select role**, choose **Member**. Then select the users and groups to whom you want to grant cluster access. A group admin can modify these assignments at any time. When you're ready to move on, select **Next**.

    :::image type="content" source="media/azure-rbac/jit-adding-assignment.png" alt-text="Screenshot showing how to add assignments in the Azure portal.":::

1. Choose an assignment type of **Active**, choose the desired duration, and provide a justification. When you're ready to proceed, select **Assign**.

    :::image type="content" source="media/azure-rbac/jit-set-active-assignment.png" alt-text="Screenshot showing assignment properties in the Azure portal." :::

For more information about these steps and options, see [Assign eligibility for a group in Privileged Identity Management](/entra/id-governance/privileged-identity-management/groups-assign-member-owner).

After you've made the assignments, verify that just-in-time access is working by accessing the cluster. For example, use the `kubectl get nodes` command to view nodes in the cluster:

```console
kubectl get nodes
```

Note the authentication requirement and follow the steps to authenticate. If authentication is successful, you should see output similar to this:

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.

NAME      STATUS   ROLES    AGE      VERSION
node-1    Ready    agent    6m36s    v1.18.14
node-2    Ready    agent    6m42s    v1.18.14
node-3    Ready    agent    6m33s    v1.18.14
```

## Maintain Azure RBAC certificate

Azure RBAC on Azure Arc–enabled Kubernetes clusters uses the Azure Arc Guard authorization webhook. On unmanaged Kubernetes clusters (such as k3s), the guard webhook certificates are stored on disk and require manual rotation. The guard webhook certificate expires in one year from the date of creation or last refresh and should be updated annually before the expiration. The expiry date can be checked via the guard-authz-webhook certificate file or the Guard authentication file on disk.

Manual maintenance is required if Guard webhook client certificates stored on disk expire. When this occurs, API server authorization requests fail even though the cluster and workloads remain healthy. If the guard webhook certificate expires, you may see the following:
-	kubectl commands failing with TLS errors such as tls: bad certificate. 

Example: 
```
Error from server (InternalError): an error on the server ("Internal Server Error: \"/api/v1/nodes?limit=500\": Post \"https://192.168.X.X:443/subjectaccessreviews?timeout=30s\": remote error: tls: bad certificate") has prevented the request from succeeding (get nodes)
```

- Azure CLI operations against the cluster failing
-	API server or Guard logs showing TLS handshake or certificate expiration errors, such as x509: certificate has expired. Example:

```
http: TLS handshake error from 192.168.0.0:65062: tls: failed to verify certificate: x509: certificate has expired or is not yet valid:
```
To fix the issue, you should refresh the Azure RBAC guard webhook certificate. You’ll need the following access to perform the fix: 

Prerequisites
-	Read access to secrets in the kube-system namespace in the affected cluster.
-	For non-Cluster API clusters, SSH access to the system/master nodes of the cluster.
-	For Cluster API clusters, access to the management cluster where the workload clusters are created.

Once access prerequisites are met, you can refresh the guard webhook certificate by performing the following steps:
1.	Refresh the guard webhook certificate by running the following command: 
```
kubectl rollout restart deployment/guard -n azure-arc
```
1.	For a [Generic cluster where no reconciler is running on the apiserver specification](#Generic-cluster-where-no-reconciler-is-running-on-the-apiserver-specification), you should perform step 1a and then restart the kube-apiserver pod. 
1. For a [Cluster created by using Cluster API](#Cluster-created-by-using-Cluster-API), you should perform steps 1-3. After steps 1-3 are completed, you need to trigger a re-reconcile of the KubeadmControlPlane object to apply the new webhook configs to the workload cluster. Step #4 (edit the CR) may trigger a re-reconcile of the KubeadmControlPlane object.


## Next steps

- Read about the [architecture of Azure RBAC on Arc-enabled Kubernetes](conceptual-azure-rbac.md).
- Securely connect to an Arc-enabled Kubernetes cluster by using [cluster connect](cluster-connect.md).
- Help to protect your cluster in other ways by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).
