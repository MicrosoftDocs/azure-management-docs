---
title: "Quickstart: Connect an existing Kubernetes cluster to Azure Arc"
description: In this quickstart, you learn how to connect an Azure Arc-enabled Kubernetes cluster.
ms.topic: quickstart
ms.date: 10/08/2024
ms.custom: template-quickstart, mode-other, devx-track-azurecli, devx-track-azurepowershell
ms.devlang: azurecli
# Customer intent: As a Kubernetes operator, I want to connect my existing Kubernetes cluster to Azure Arc, so that I can manage and monitor it using Azure's capabilities and tools.
---

# Quickstart: Connect an existing Kubernetes cluster to Azure Arc

Get started with Azure Arc-enabled Kubernetes by using Azure CLI or Azure PowerShell to connect an existing Kubernetes cluster to Azure Arc.

For a conceptual look at connecting clusters to Azure Arc, see [Azure Arc-enabled Kubernetes agent overview](./conceptual-agent-overview.md). To try things out in a sample/practice experience, visit the [Azure Arc Jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_k8s).

## Prerequisites

> [!IMPORTANT]
> In addition to these prerequisites, be sure to meet all [network requirements for Azure Arc-enabled Kubernetes](network-requirements.md).

### [Azure CLI](#tab/azure-cli)

* An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* A basic understanding of [Kubernetes core concepts](/azure/aks/concepts-clusters-workloads).
* An [identity (user or service principal)](system-requirements.md#azure-ad-identity-requirements) that you can use to [sign in to Azure CLI](/cli/azure/authenticate-azure-cli) and connect your cluster to Azure Arc.
* The latest version of [Azure CLI](/cli/azure/install-azure-cli).
* The latest version of **connectedk8s** Azure CLI extension, installed by running the following command:

  ```azurecli
  az extension add --name connectedk8s
  ```

* A running Kubernetes cluster. If you don't have one, you can create a cluster by using one of these options:
  * [Kubernetes in Docker (KIND)](https://kind.sigs.k8s.io/)
  * Use Docker for [Linux](https://docs.docker.com/engine/install), [Mac](https://docs.docker.com/desktop/setup/install/mac-install/) or [Windows](hhttps://docs.docker.com/desktop/setup/install/windows-install/)
  * Self-managed Kubernetes cluster using [Cluster API](https://cluster-api.sigs.k8s.io/user/quick-start.html)

    > [!NOTE]
    > The cluster needs to have at least one node of operating system and architecture type `linux/amd64` and/or `linux/arm64`. See [Cluster requirements](system-requirements.md#cluster-requirements) for more about ARM64 scenarios.

* At least 850 MB free for the Arc agents that you deploy on the cluster, and capacity to use approximately 7% of a single CPU.
* A [kubeconfig file](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) and context pointing to your cluster. For more information, see [Configure access to multiple clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).

### [Azure PowerShell](#tab/azure-powershell)

* An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
* A basic understanding of [Kubernetes core concepts](/azure/aks/concepts-clusters-workloads).
* An [identity (user or service principal)](system-requirements.md#azure-ad-identity-requirements) that you can use to [sign in to Azure PowerShell](/powershell/azure/authenticate-azureps) and connect your cluster to Azure Arc.
* [Azure PowerShell version 6.6.0 or later](/powershell/azure/install-azure-powershell)
* The **Az.ConnectedKubernetes** Azure PowerShell module, installed by running the following command:

    ```azurepowershell-interactive
    Install-Module -Name Az.ConnectedKubernetes
    ```

* A running Kubernetes cluster. If you don't have one, you can create a cluster by using one of these options:
  * [Kubernetes in Docker (KIND)](https://kind.sigs.k8s.io/)
  * Use Docker for [Linux](https://docs.docker.com/engine/install), [Mac](https://docs.docker.com/desktop/setup/install/mac-install/) or [Windows](hhttps://docs.docker.com/desktop/setup/install/windows-install/)
  * Self-managed Kubernetes cluster using [Cluster API](https://cluster-api.sigs.k8s.io/user/quick-start.html)

    > [!NOTE]
    > The cluster needs to have at least one node of operating system and architecture type `linux/amd64` and/or `linux/arm64`. See [Cluster requirements](system-requirements.md#cluster-requirements) for more about ARM64 scenarios.

* At least 850 MB free for the Arc agents that you deploy on the cluster, and capacity to use approximately 7% of a single CPU.
* A [kubeconfig file](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) and context pointing to your cluster. For more information, see [Configure access to multiple clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).

---

## Register providers for Azure Arc-enabled Kubernetes

### [Azure CLI](#tab/azure-cli)

1. Enter the following commands:

    ```azurecli
    az provider register --namespace Microsoft.Kubernetes
    az provider register --namespace Microsoft.KubernetesConfiguration
    az provider register --namespace Microsoft.ExtendedLocation
    ```

1. Monitor the registration process. Registration can take up to 10 minutes.

    ```azurecli
    az provider show -n Microsoft.Kubernetes -o table
    az provider show -n Microsoft.KubernetesConfiguration -o table
    az provider show -n Microsoft.ExtendedLocation -o table
    ```

    When the registration finishes, the `RegistrationState` for these namespaces changes to `Registered`.

### [Azure PowerShell](#tab/azure-powershell)

1. Enter the following commands:

    ```azurepowershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
    Register-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
    Register-AzResourceProvider -ProviderNamespace Microsoft.ExtendedLocation
    ```

1. Monitor the registration process. Registration can take up to 10 minutes.

    ```azurepowershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
    Get-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
    Get-AzResourceProvider -ProviderNamespace Microsoft.ExtendedLocation
    ```

    When the registration finishes, the `RegistrationState` for these namespaces changes to `Registered`.

---

## Create a resource group

Run the following command to create a resource group for your Azure Arc-enabled Kubernetes cluster. In this example, the resource group is named AzureArcTest and is created in the EastUS region.

### [Azure CLI](#tab/azure-cli)

```azurecli
az group create --name AzureArcTest --location EastUS --output table
```

Output:

```output
Location    Name
----------  ------------
eastus      AzureArcTest
```

### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup -Name AzureArcTest -Location EastUS
```

Output:

```output
ResourceGroupName : AzureArcTest
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/AzureArcTest
```

---

## Connect an existing Kubernetes cluster

Run the following command to connect your cluster. This command deploys the Azure Arc agents to the cluster and installs Helm v3.6.3 to the `.azure` folder of the deployment machine. This Helm 3 installation is only used for Azure Arc, and it doesn't remove or change any previously installed versions of Helm on the machine.

In this example, the cluster's name is AzureArcTest1.

### [Azure CLI](#tab/azure-cli)

```azurecli
az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
```

Output:

```output
Helm release deployment succeeded

    {
      "aadProfile": {
        "clientAppId": "",
        "serverAppId": "",
        "tenantId": ""
      },
      "agentPublicKeyCertificate": "xxxxxxxxxxxxxxxxxxx",
      "agentVersion": null,
      "connectivityStatus": "Connecting",
      "distribution": "gke",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1",
      "identity": {
        "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "type": "SystemAssigned"
      },
      "infrastructure": "gcp",
      "kubernetesVersion": null,
      "lastConnectivityTime": null,
      "location": "eastus",
      "managedIdentityCertificateExpirationTime": null,
      "name": "AzureArcTest1",
      "offering": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "AzureArcTest",
      "tags": {},
      "totalCoreCount": null,
      "totalNodeCount": null,
      "type": "Microsoft.Kubernetes/connectedClusters"
    }
```

> [!TIP]
> If you don't specify the location parameter in the preceding command, the Azure Arc-enabled Kubernetes resource is created in the same location as the resource group. To create the Azure Arc-enabled Kubernetes resource in a different location, specify either `--location <region>` or `-l <region>` when running the `az connectedk8s connect` command.

### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzConnectedKubernetes -ClusterName AzureArcTest1 -ResourceGroupName AzureArcTest -Location eastus
```

Output:

```output
Location Name          Type
-------- ----          ----
eastus   AzureArcTest1 microsoft.kubernetes/connectedclusters
```

---

> [!IMPORTANT]
> If deployment fails due to a timeout error, see the [troubleshooting guide](troubleshooting.md#helm-timeout-error) for help with resolving the issue.

## Connect using an outbound proxy server

If your cluster is behind an outbound proxy server protects your cluster, route requests through the outbound proxy server.

### [Azure CLI](#tab/azure-cli)

1. On the deployment machine, set the environment variables needed for Azure CLI to use the outbound proxy server:

    ```bash
    export HTTP_PROXY=<proxy-server-ip-address>:<port>
    export HTTPS_PROXY=<proxy-server-ip-address>:<port>
    export NO_PROXY=<cluster-apiserver-ip-address>:<port>
    ```

1. On the Kubernetes cluster, run the connect command with the `proxy-https` and `proxy-http` parameters specified. If your proxy server is set up with both HTTP and HTTPS, be sure to use `--proxy-http` for the HTTP proxy and `--proxy-https` for the HTTPS proxy. If your proxy server only uses HTTP, you can use that value for both parameters.

    ```azurecli
    az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --proxy-cert <path-to-cert-file>
    ```

   > [!NOTE]
   >
   > * Some network requests, such as the ones involving in-cluster service-to-service communication, need to be separated from the traffic that is routed via the proxy server for outbound communication. Use the `--proxy-skip-range` parameter to specify the CIDR range and endpoints in a comma-separated way so that any communication from the agents to these endpoints doesn't go via the outbound proxy. At a minimum, specify the CIDR range of the services in the cluster as the value for this parameter. For example, let's say `kubectl get svc -A` returns a list of services where all the services have ClusterIP values in the range `10.0.0.0/16`. Then the value to specify for `--proxy-skip-range` is `10.0.0.0/16,kubernetes.default.svc,.svc.cluster.local,.svc`.
   > * Most outbound proxy environments expect `--proxy-http`, `--proxy-https`, and `--proxy-skip-range`. The `--proxy-cert` parameter is *only* required if you need to inject trusted certificates that the proxy expects into the trusted certificate store of agent pods.
   > * The outbound proxy must be configured to allow websocket connections.

   For outbound proxy servers, if you're only providing a trusted certificate, run `az connectedk8s connect` with just the `--proxy-cert` parameter specified:

   ```azurecli
   az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-cert <path-to-cert-file>
   ```

   If there are multiple trusted certificates, combine the certificate chain (Leaf cert, Intermediate cert, Root cert) into a single file which you pass in the `--proxy-cert` parameter.

   > [!TIP]
   >
   > * `--custom-ca-cert` is an alias for `--proxy-cert`. Use either parameter interchangeably. If you pass both parameters in the same command, the one passed last is honored.

### [Azure PowerShell](#tab/azure-powershell)

1. On the deployment machine, set the environment variables needed for Azure PowerShell to use the outbound proxy server:

    ```powershell
    $Env:HTTP_PROXY = "<proxy-server-ip-address>:<port>"
    $Env:HTTPS_PROXY = "<proxy-server-ip-address>:<port>"
    $Env:NO_PROXY = "<cluster-apiserver-ip-address>:<port>"
    ```

1. On the Kubernetes cluster, run the `connect` command with the proxy parameter specified:

    ```azurepowershell
    New-AzConnectedKubernetes -ClusterName <cluster-name> -ResourceGroupName <resource-group> -Location eastus -Proxy 'https://<proxy-server-ip-address>:<port>'
    ```

Azure PowerShell doesn't currently support passing in only the proxy certificate without proxy server endpoint details.

---

## Verify cluster connection

Verify that the cluster is connected to Azure by running the following command:

### [Azure CLI](#tab/azure-cli)

```azurecli
az connectedk8s list --resource-group AzureArcTest --output table
```

Output:

```output
Name           Location    ResourceGroup
-------------  ----------  ---------------
AzureArcTest1  eastus      AzureArcTest
```

### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzConnectedKubernetes -ResourceGroupName AzureArcTest
```

Output:

```output
Location Name          Type
-------- ----          ----
eastus   AzureArcTest1 microsoft.kubernetes/connectedclusters
```

---

If the connection wasn't successful, see [Diagnose connection issues for Azure Arc-enabled Kubernetes clusters](diagnose-connection-issues.md) for troubleshooting steps.

> [!NOTE]
> After onboarding the cluster, it can take up to ten minutes for cluster metadata (such as cluster version and number of nodes) to appear on the overview page of the Azure Arc-enabled Kubernetes resource in the Azure portal.

## View Azure Arc agents for Kubernetes

Azure Arc-enabled Kubernetes deploys several agents into the `azure-arc` namespace.

1. View these deployments and pods by using the following command:

   ```bash
   kubectl get deployments,pods -n azure-arc
   ```

1. Verify all pods are in a `Running` state.

   Output:

   ```output
    NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/cluster-metadata-operator   1/1     1            1           13d
    deployment.apps/clusterconnect-agent        1/1     1            1           13d
    deployment.apps/clusteridentityoperator     1/1     1            1           13d
    deployment.apps/config-agent                1/1     1            1           13d
    deployment.apps/controller-manager          1/1     1            1           13d
    deployment.apps/extension-manager           1/1     1            1           13d
    deployment.apps/flux-logs-agent             1/1     1            1           13d
    deployment.apps/kube-aad-proxy              1/1     1            1           13d
    deployment.apps/metrics-agent               1/1     1            1           13d
    deployment.apps/resource-sync-agent         1/1     1            1           13d

    NAME                                            READY   STATUS    RESTARTS   AGE
    pod/cluster-metadata-operator-9568b899c-2stjn   2/2     Running   0          13d
    pod/clusterconnect-agent-576758886d-vggmv       3/3     Running   0          13d
    pod/clusteridentityoperator-6f59466c87-mm96j    2/2     Running   0          13d
    pod/config-agent-7cbd6cb89f-9fdnt               2/2     Running   0          13d
    pod/controller-manager-df6d56db5-kxmfj          2/2     Running   0          13d
    pod/extension-manager-58c94c5b89-c6q72          2/2     Running   0          13d
    pod/flux-logs-agent-6db9687fcb-rmxww            1/1     Running   0          13d
    pod/kube-aad-proxy-67b87b9f55-bthqv             2/2     Running   0          13d
    pod/metrics-agent-575c565fd9-k5j2t              2/2     Running   0          13d
    pod/resource-sync-agent-6bbd8bcd86-x5bk5        2/2     Running   0          13d
   ```

For more information about these agents, see [Azure Arc-enabled Kubernetes agent overview](conceptual-agent-overview.md).

## Clean up resources

### [Azure CLI](#tab/azure-cli)

To delete the Azure Arc-enabled Kubernetes resource, along with any associated configuration resources and agents running on the cluster, use the following command:

```azurecli
az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
```

If the deletion process fails, use the following command to force deletion. Add `-y` to bypass the confirmation prompt:

```azurecli
az connectedk8s delete -n AzureArcTest1 -g AzureArcTest --force
```

Use this command if you experience problems when creating a new cluster deployment due to previously created resources that aren't completely removed.

>[!NOTE]
> If you delete the Azure Arc-enabled Kubernetes resource in the Azure portal, you remove associated configuration resources, but agents running on the cluster aren't removed. For this reason, we recommend using `az connectedk8s delete` to delete the Azure Arc-enabled Kubernetes resource.

### [Azure PowerShell](#tab/azure-powershell)

You can delete the Azure Arc-enabled Kubernetes resource, any associated configuration resources, and any agents running on the cluster by using the following command:

```azurepowershell
Remove-AzConnectedKubernetes -ClusterName AzureArcTest1 -ResourceGroupName AzureArcTest
```

>[!NOTE]
> If you delete the Azure Arc-enabled Kubernetes resource in the Azure portal, you remove associated configuration resources but agents running on the cluster aren't removed. For this reason, we recommend using `Remove-AzConnectedKubernetes` to delete the Azure Arc-enabled Kubernetes resource.

---

## Next steps

* Learn how to [deploy configurations using GitOps with Flux v2](tutorial-use-gitops-flux2.md).
* [Troubleshoot common Azure Arc-enabled Kubernetes issues](troubleshooting.md).
* Experience Azure Arc-enabled Kubernetes automated scenarios with [Azure Arc Jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_k8s).
* Help protect your cluster by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).