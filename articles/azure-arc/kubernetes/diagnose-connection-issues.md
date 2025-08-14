---
title: "Diagnose connection issues for Azure Arc-enabled Kubernetes clusters"
ms.date: 12/15/2023
ms.topic: how-to
description: "Learn how to resolve common issues when connecting Kubernetes clusters to Azure Arc."
ms.custom:
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: As a Kubernetes administrator, I want to diagnose and resolve connection issues for Azure Arc-enabled Kubernetes clusters, so that I can ensure reliable connectivity and management of my clusters from within Azure.
---

# Diagnose connection issues for Azure Arc-enabled Kubernetes clusters

If you are experiencing issues connecting a cluster to Azure Arc, it's probably due to one of the issues listed here. We provide two flowcharts with guided help: one if you're [not using a proxy server](#connections-without-a-proxy), and one that applies if your network connection [uses a proxy server](#connections-with-a-proxy-server).

> [!TIP]
> The steps in this flowchart apply whether you're using Azure CLI or Azure PowerShell to [connect your cluster](quickstart-connect-cluster.md). However, some of the steps require the use of Azure CLI. If you haven't already [installed Azure CLI](/cli/azure/install-azure-cli), be sure to do so before you begin.

## Connections without a proxy

Review this flowchart in order to diagnose your issue when attempting to connect a cluster to Azure Arc without a proxy server. More details about each step are provided below.

:::image type="content" source="media/diagnose-connection-issues/no-proxy-flowchart.png" alt-text="Flowchart showing a visual representation of checking for connection issues when not using a proxy." lightbox="media/diagnose-connection-issues/no-proxy-flowchart.png":::

### Does the Azure identity have sufficient permissions?

Review the [prerequisites for connecting a cluster](quickstart-connect-cluster.md?tabs=azure-cli#prerequisites) and make sure that the identity you're using to connect the cluster has the necessary permissions.

### Are you running the latest version of Azure CLI?

Make sure you [have the latest version installed](/cli/azure/install-azure-cli).

If you connected your cluster by using Azure PowerShell, make sure you are [running the latest version](/powershell/azure/install-azure-powershell).

### Is the `connectedk8s` extension the latest version?

Update the Azure CLI `connectedk8s` extension to the latest version by running this command:

```azurecli
az extension update --name connectedk8s
```

If you haven't installed the extension yet, you can do so by running the following command:

```azurecli
az extension add --name connectedk8s
```

### Is kubeconfig pointing to the right cluster?

Run `kubectl config get-contexts` to confirm the target context name. Then set the default context to the right cluster by running `kubectl config use-context <target-cluster-name>`.

### Are all required resource providers registered?

Be sure that the Microsoft.Kubernetes, Microsoft.KubernetesConfiguration, and Microsoft.ExtendedLocation resource providers are [registered](quickstart-connect-cluster.md#register-providers-for-azure-arc-enabled-kubernetes).

### Are all network requirements met?

Review the [network requirements](network-requirements.md) and ensure that no required endpoints are blocked.

### Are all pods in the `azure-arc` namespace running?

If everything is working correctly, your pods should all be in the `Running` state. Run `kubectl get pods -n azure-arc` to confirm whether any pod's state is not `Running`.

### Still having problems?

The steps above will resolve many common connection issues, but if you're still unable to connect successfully, generate a troubleshooting log file and then [open a support request](../../azure-portal/supportability/how-to-create-azure-support-request.md) so we can investigate the problem further.

To generate the troubleshooting log file, run the following command:

```azurecli
az connectedk8s troubleshoot -g <myResourceGroup> -n <myK8sCluster>
```

When you [create your support request](../../azure-portal/supportability/how-to-create-azure-support-request.md), in the **Additional details** section, use the **File upload** option to upload the generated log file.

## Connections with a proxy server

If you are using a proxy server on at least one machine, complete the first five steps of the non-proxy flowchart (through resource provider registration) for basic troubleshooting steps. Then, if you are still encountering issues, review the next flowchart for additional troubleshooting steps. More details about each step are provided below.

:::image type="content" source="media/diagnose-connection-issues/proxy-flowchart.png" alt-text="Flowchart showing a visual representation of checking for connection issues when using a proxy." lightbox="media/diagnose-connection-issues/proxy-flowchart.png":::

### Is the machine executing commands behind a proxy server?

If the machine is executing commands behind a proxy server, you'll need to set all of the necessary environment variables. For more information, see [Connect using an outbound proxy server](quickstart-connect-cluster.md#connect-using-an-outbound-proxy-server).

For example:

```bash
export HTTP_PROXY="http://<proxyIP>:<proxyPort>"
export HTTPS_PROXY="https://<proxyIP>:<proxyPort>"
export NO_PROXY="<cluster-apiserver-ip-address>:<proxyPort>"
```

### Does the proxy server only accept trusted certificates?

Be sure to include the certificate file path by including `--proxy-cert <path-to-cert-file>` when running the `az connectedk8s connect` command.

```azurecli
az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-cert <path-to-cert-file>
```

### Is the proxy server able to reach required network endpoints?

Review the [network requirements](network-requirements.md) and ensure that no required endpoints are blocked.

### Is the proxy server only using HTTP?

If your proxy server only uses HTTP, you can use `proxy-http` for both parameters.

If your proxy server is set up with both HTTP and HTTPS, run the `az connectedk8s connect` command with the `--proxy-https` and `--proxy-http` parameters specified. Be sure you are using `--proxy-http` for the HTTP proxy and `--proxy-https` for the HTTPS proxy.

```azurecli
az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port>  
```

### Does the proxy server require skip ranges for service-to-service communication?

If you require skip ranges, use `--proxy-skip-range <excludedIP>,<excludedCIDR>` in your `az connectedk8s connect` command.

```azurecli
az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR>
```

### Are all pods in the `azure-arc` namespace running?

If everything is working correctly, your pods should all be in the `Running` state. Run `kubectl get pods -n azure-arc` to confirm whether any pod's state is not `Running`.


### Check whether the DNS resolution is successful for the endpoint

From within the pod, you can run a DNS lookup to the endpoint.

What if you can't run the [kubectl exec](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec) command to connect to the pod and install the DNS Utils package? In this situation, you can [start a test pod in the same namespace as the problematic pod](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/#create-a-simple-pod-to-use-as-a-test-environment), and then run the tests.

> [!NOTE]
>
> If the DNS resolution or egress traffic doesn't let you install the necessary network packages, you can use the `rishasi/ubuntu-netutil:1.0` docker image. In this image, the required packages are already installed.

Here's an example procedure for checking DNS resolution:

1. Start a test pod in the same namespace as the problematic pod:

   ```bash
   kubectl run -it --rm test-pod --namespace <namespace> --image=debian:stable
   ```

   After the test pod is running, you'll gain access to the pod.

1. Run the following `apt-get` commands to install other tool packages:

   ```bash
   apt-get update -y
   apt-get install dnsutils -y
   apt-get install curl -y
   apt-get install netcat -y
   ```

1. After the packages are installed, run the [nslookup](/windows-server/administration/windows-commands/nslookup) command to test the DNS resolution to the endpoint:

   ```console
   $ nslookup microsoft.com
   Server:         10.0.0.10
   Address:        10.0.0.10#53
   ...
   ...
   Name:   microsoft.com
   Address: 20.53.203.50
   ```

1. Try the DNS resolution from the upstream DNS server directly. This example uses Azure DNS:

   ```console
   $ nslookup microsoft.com 168.63.129.16
   Server:         168.63.129.16
   Address:        168.63.129.16#53
   ...
   ...
   Address: 20.81.111.85
   ```

1. Run the `host` command to check whether the DNS requests are routed to the upstream server:

   ```console
   $ host -a microsoft.com
   Trying "microsoft.com.default.svc.cluster.local"
   Trying "microsoft.com.svc.cluster.local"
   Trying "microsoft.com.cluster.local"
   Trying "microsoft.com.00idcnmrrm4edot5s2or1onxsc.bx.internal.cloudapp.net"
   Trying "microsoft.com"
   Trying "microsoft.com"
   ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62884
   ;; flags: qr rd ra; QUERY: 1, ANSWER: 27, AUTHORITY: 0, ADDITIONAL: 5
   
   ;; QUESTION SECTION:
   ;microsoft.com.                 IN      ANY
   
   ;; ANSWER SECTION:
   microsoft.com.          30      IN      NS      ns1-39.azure-dns.com.
   ...
   ...
   ns4-39.azure-dns.info.  30      IN      A       13.107.206.39
   
   Received 2121 bytes from 10.0.0.10#53 in 232 ms
   ```

1. Run a test pod in the Windows node pool:

   ```bash
   # For a Windows environment, use the Resolve-DnsName cmdlet.
   kubectl run dnsutil-win --image='mcr.microsoft.com/windows/servercore:1809' --overrides='{"spec": { "nodeSelector": {"kubernetes.io/os": "windows"}}}' -- powershell "Start-Sleep -s 3600"
   ```

1. Run the [kubectl exec](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec) command to connect to the pod by using PowerShell:

   ```bash
   kubectl exec -it dnsutil-win -- powershell
   ```

1. Run the [Resolve-DnsName](/powershell/module/dnsclient/resolve-dnsname) cmdlet in PowerShell to check whether the DNS resolution is working for the endpoint:

   ```console
   PS C:\> Resolve-DnsName www.microsoft.com 
   
   Name                           Type   TTL   Section    NameHost
   ----                           ----   ---   -------    --------
   www.microsoft.com              CNAME  20    Answer     www.microsoft.com-c-3.edgekey.net
   www.microsoft.com-c-3.edgekey. CNAME  20    Answer     www.microsoft.com-c-3.edgekey.net.globalredir.akadns.net
   net
   www.microsoft.com-c-3.edgekey. CNAME  20    Answer     e13678.dscb.akamaiedge.net
   net.globalredir.akadns.net
   
   Name       : e13678.dscb.akamaiedge.net 
   QueryType  : AAAA
   TTL        : 20
   Section    : Answer
   IP6Address : 2600:1408:c400:484::356e   
   
   
   Name       : e13678.dscb.akamaiedge.net 
   QueryType  : AAAA
   TTL        : 20
   Section    : Answer
   IP6Address : 2600:1408:c400:496::356e 
   
   
   Name       : e13678.dscb.akamaiedge.net
   QueryType  : A
   TTL        : 12
   Section    : Answer
   IP4Address : 23.200.197.152
   ```

If the DNS resolution is not successful, verify the DNS configuration for the cluster.



### Still having problems?

The steps above will resolve many common connection issues, but if you're still unable to connect successfully, generate a troubleshooting log file and then [open a support request](../../azure-portal/supportability/how-to-create-azure-support-request.md) so we can investigate the problem further.

To generate the troubleshooting log file, run the following command:

```azurecli
az connectedk8s troubleshoot -g <myResourceGroup> -n <myK8sCluster>
```

When you [create your support request](../../azure-portal/supportability/how-to-create-azure-support-request.md), in the **Additional details** section, use the **File upload** option to upload the generated log file.

## Next steps

- View more [troubleshooting tips for using Azure Arc-enabled Kubernetes](troubleshooting.md).
- Review the process to [connect an existing Kubernetes cluster to Azure Arc](quickstart-connect-cluster.md).
