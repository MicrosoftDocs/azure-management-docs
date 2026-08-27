---
title: Troubleshoot Azure Arc resource bridge issues
description: This article tells how to troubleshoot and resolve issues with the Azure Arc resource bridge when trying to deploy or connect to the service.
ms.date: 08/25/2026
ms.topic: troubleshooting

# Customer intent: As a cloud administrator, I want to troubleshoot Azure Arc resource bridge deployment issues, so that I can ensure stable and reliable connectivity to my on-premises infrastructure.
---

# Troubleshoot Azure Arc resource bridge issues

This article provides information on troubleshooting and resolving issues that could occur while attempting to deploy, use, or remove the Azure Arc resource bridge. The resource bridge is a packaged virtual machine, which hosts a *management* Kubernetes cluster. For general information, see [Azure Arc resource bridge overview](./overview.md).

> [!NOTE]
> - For **Arc-enabled System Center Virtual Machine Manager**, refer to the [Arc-enabled SCVMM troubleshoot guide](../system-center-virtual-machine-manager/troubleshoot-scvmm.md).
> - For **Azure Local**, refer to  [Troubleshoot Azure Arc VM management for Azure Local](/azure/azure-local/manage/troubleshoot-arc-enabled-vms) or contact Microsoft Support. Arc resource bridge is a critical component of Azure Local and shouldn't be deleted without guidance from Microsoft Support.

## General issues

### Logs collection

For issues encountered with Arc resource bridge, collect logs for further investigation by using the Azure CLI [`az arcappliance logs`](/cli/azure/arcappliance/logs) command. Run this command from the management machine you use to deploy the Arc resource bridge. If you're using a different machine, the machine must meet the [management machine requirements](system-requirements.md#management-machine-requirements).

If you can't collect logs, the management machine most likely can't reach the Appliance VM. Contact your network administrator to allow SSH communication from the management machine to the Appliance VM on TCP port 22.

You can collect the Arc resource bridge logs by passing either the appliance VM IP or the kubeconfig in the logs command.

To collect Arc resource bridge logs on VMware by using the appliance VM IP address:

```azurecli
az arcappliance logs vmware \
  --ip <appliance VM IP> \
  --username <vSphere username> \
  --password <vSphere password> \
  --address <vCenter address> \
  --out-dir <path to output directory>
```

To collect Arc Resource Bridge logs for Azure Local, see [Collect logs](/azure/azure-local/manage/collect-logs).

If you're unsure of your appliance VM IP, use the kubeconfig. Retrieve the kubeconfig by running the [get-credentials command](/cli/azure/arcappliance) and then run the logs command.

To retrieve the kubeconfig and log key, then collect logs for Arc-enabled VMware from a different machine than the one you used to deploy Arc resource bridge for Arc-enabled VMware:

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <Arc resource bridge name> -g <resource group name>
az arcappliance logs vmware --kubeconfig kubeconfig --out-dir <path to specified output directory>
```

### Restore from backup or snapshot isn't supported

In general, you can't restore the Arc resource bridge from snapshots or VM backups. If you roll back the appliance by using a VM snapshot or backup because an upgrade failed, the state might no longer match Azure's recorded state. This mismatch is irreversible. Avoid using hypervisor-level or VM-level snapshot or restore operations on the appliance. These operations break upgrade integrity, block future updates, or prevent the resource from returning to a healthy state. If you restore the resource bridge by using these unsupported methods, the only supported recovery path is a recovery operation.

Follow the recovery guidance for your Arc private cloud:

- **Arc-enabled VMware:** Follow the [Arc-enabled VMware recovery guide](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion).
- **Azure Local:** Contact Microsoft Support. Arc resource bridge is an integrated component of Azure Local, and the support team should guide any changes.
- **Arc-enabled SCVMM:** Follow the [Arc-enabled SCVMM recovery guide](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery).

### Get authentication credentials error on Azure CLI v2.70.0

You might encounter an error when running `az arcappliance` commands that looks like this:

```output
`File "C:\Program Files\Common Files\AzureCliExtensionDirectory\arcappliance\azext_arcappliance\helpers.py", line 103, in get_tenant_id_and_cloud`

_, _, tenant = profile.get_login_credentials(resource=cmd.cli_ctx.cloud.endpoints.active_directory_graph_resource_id)

TypeError: get_login_credentials() got an unexpected keyword argument 'resource'
```

Azure CLI v2.70.0 released a breaking change that triggers this error in `arcappliance` CLI extension v1.4.0 and below. A fix is available in `arcappliance` CLI extension 1.4.1 for compatibility with Azure CLI v2.70.0. You can get the latest `arcappliance` CLI extension by running the following command:


```azurecli
az extension add --upgrade --name arcappliance
```

If you are on `az arcappliance` extension is 1.4.0 or lower, you need to downgrade Azure CLI to v2.69.0.

If you used the Azure CLI installer, you can uninstall the current version and install Azure CLI v2.69.0 from the [`Azure CLI installation page`](/cli/azure/install-azure-cli). If you used the `pip` installer, you can run the following command to downgrade: `pip install azure-cli==2.69.0`.

Also, for the Arc-enabled VMware onboarding script, you might need to comment out the following code in the script to prevent updating the Azure CLI to the latest version:

```azurecli
if (shouldInstallAzCli) {
   installAzCli64Bit
}
```

### Error downloading release file information

> [!WARNING]
> For Azure Local, you must use the built-in LCM tool to upgrade Arc resource bridge. If you attempt to manual upgrade by using the Azure CLI command, your environment breaks and can't be recovered. If you need assistance with an Arc resource bridge upgrade, contact Microsoft Support.

When you upgrade Arc resource bridge by using Azure CLI, you might get the following error:

```output
az arcappliance upgrade vmware' failed: (DownloadError) "{\n\"message\": \"Error downloading file release information.: Unable to find file release: ^mariner-2-0-(.*)-vhdx-rpm-(.*)$ with version:  in product release: arc-appliance-stable-releases\"\n}
```

If you're using an `az arcappliance` Azure CLI extension version that's earlier than 1.4.0 and attempting to upgrade to appliance version 1.4.0, update your Azure CLI extension to the latest version:

```azurecli
az extension add --upgrade --name arcappliance
```

When your `az arcappliance` extension is 1.4.0, retry the upgrade to appliance version 1.4.0. When you upgrade an Arc resource bridge, you upgrade to the next version, which might not be the latest version. For more information, see [Arc resource bridge release notes](release-notes.md).

### Download/upload connectivity wasn't successful

If your network speed is slow, you might not be able to successfully download the Arc resource bridge VM image, resulting in this error:

```output
ErrorCode: ValidateKvaError, Error: Pre-deployment validation of your download/upload connectivity was not successful. Timeout error occurred during download and preparation of appliance image to the on-premises fabric storage. Common causes of this timeout error are slow network download/upload speeds, a proxy limiting the network speed or slow storage performance.
```

As a workaround, try creating a VM directly on the on-premises private cloud, and then run the Arc resource bridge deployment script from that VM. This approach should result in a faster upload of the image to the datastore.

### Invalid overlap with Kubernetes Service or Pod CIDR

When you deploy or upgrade Azure Arc resource bridge, the operation fails with an error similar to:

```output
"code": "InvalidOverlapWithServiceCIDR",
"message": "Network overlap validation failed: IP: <IP_ADDRESS> overlaps with K8s Default Service CIDR: 10.96.0.0/12. Please reconfigure the network to resolve this conflict."
```

Azure Arc resource bridge reserves specific IP ranges as defined in the [network requirements](network-requirements.md#designated-ip-ranges-for-arc-resource-bridge). This error occurs when the IP address or subnet you configure for your Arc resource bridge overlaps with a reserved Kubernetes CIDR. Overlapping CIDRs aren't supported and preflight validation blocks them. Any overlap with these ranges results in deployment or upgrade failure.

The following CIDRs are reserved by Arc resource bridge:

- Kubernetes Service CIDR: 10.96.0.0/12
- Kubernetes Pod CIDR: 10.244.0.0/16

The resolution depends on whether this error occurs during initial deployment or post-deployment. If you're deploying for the first time, choose a different IP address or subnet that doesn't overlap with the Kubernetes Service CIDR or any other reserved CIDRs. Then, retry the deployment.

If you already deployed Arc resource bridge and you encounter this error during an upgrade or configuration change, perform a recovery operation. During recovery, specify a non-overlapping IP address or subnet that meets the network requirements. Follow the recovery guidance for your Arc private cloud:

- **Arc-enabled VMware:** Follow the [Arc-enabled VMware recovery guide](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion).
- **Azure Local:** Contact Microsoft Support. Arc resource bridge is an integrated component of Azure Local and the support team should guide any changes.
- **Arc-enabled SCVMM:** Follow the [Arc-enabled SCVMM recovery guide](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery).


### Context timed out during phase `ApplyingKvaImageOperator`

When you deploy Arc resource bridge, you might see this error:

```output
Deployment of the Arc resource bridge appliance VM timed out. Collect logs with _az arcappliance logs_ and create a support ticket for help. To troubleshoot the error, refer to aka.ms/arc-rb-error   { _errorCode_: _ContextError_, _errorResponse_: _{\n\_message\_: \_Context timed out during phase _ApplyingKvaImageOperator_\_\n}_ }
```

This error typically occurs when trying to download the `KVAIO` image (400 MB compressed) over a network that is slow or experiencing intermittent connectivity. The `KVAIO` controller manager waits for the image download to complete, and times out.

Check that your network speed between the Arc resource bridge VM and Microsoft Container Registry (`mcr.microsoft.com`) is stable and at least 2 Mbps. If your network connectivity and speed are stable, and you're still getting this error, wait at least 30 minutes before retrying, as it could be due to Microsoft Container Registry receiving a high volume of traffic.

### Context timed out during phase `WaitingForAPIServer`

When you deploy Arc resource bridge, you might see this error:

```output
Deployment of the Arc resource bridge appliance VM timed out. Collect logs with _az arcappliance logs_ and create a support ticket for help. To troubleshoot the error, refer to aka.ms/arc-rb-error   { _errorCode_: _ContextError_, _errorResponse_: _{\n\_message\_: \_Context timed out during phase _WaitingForAPIServer
```

This error indicates that the deployment machine can't contact the control plane IP for Arc resource bridge within the time limit. Common causes of the error are often networking related, such as communication between the deployment machine and control plane IP being routed through a proxy. Traffic from the deployment machine to the control plane and the appliance VM IPs must not pass through proxy. If traffic is being proxied, configure the proxy settings on your network or deployment machine to not proxy traffic between the deployment machine to the control plane IP and appliance VM IPs. Another cause for this error is if a firewall is closing access to port 6443 and port 22 between the deployment machine and control plane IP or the deployment machine and appliance VM IPs.

### Context timed out during phase `WaitingForAdditionalComponents`

When deploying Arc Resource Bridge, you might encounter the following error:

```output
Deployment of the Arc resource bridge appliance VM timed out. Please collect logs with _az arcappliance logs_ and create a support ticket for help. To troubleshoot the error, refer to aka.ms/arc-rb-error   { _errorCode_: _ContextError_, _errorResponse_: _{\n\_message\_: \_Context timed out during phase _WaitingForAdditionalComponents_\_\n}_
```

This error typically indicates that the deployment process stalled while waiting for critical components, such as the clusterconnect-agent or Helm charts, to initialize. Below are the most common causes and recommended actions:

- Network Bandwidth Limitations: Slow or unstable network connections can delay container image downloads, especially for key components. Ensure that the appliance VM has a stable connection to required endpoints. The lowest network bandwidth validated for deployment of Arc resource bridge is 100 Mbps. Lower speeds might result in deployment failures.

- Network Misconfiguration: Proxy interference or firewall restrictions might block access to required endpoints, for example `mcr.microsoft.com`, `time.windows.com`, or Azure APIs. Incorrect proxy settings, such as malformed `no_proxy` lists like double commas (`,,`), can break Helm chart parsing and cause misleading errors. You can review your proxy settings, validate `no_proxy` entries for syntax errors and ensure critical endpoints are excluded from proxy routing.

- Storage Performance: Suboptimal storage configuration on the appliance VM can prevent successful unpacking and initialization of components. This includes insufficient disk space, inadequate memory, or slow IOPS. To mitigate this, verify that the appliance VM meets the minimum recommended baseline: 200 GB of disk space, 4 vCPUs, and 8 GB of memory, backed by hybrid storage (flash and HDD) or all-flash storage (SSDs or NVMe). Depending on your specific environment and workload, higher specifications might be necessary.


### Context timed out during phase `WaitingForProviderSpecifics`

When you deploy Arc resource bridge, you might see this error:

```output
Deployment of the Arc resource bridge appliance VM timed out. Please collect logs with _az arcappliance logs_ and create a support ticket for help. To troubleshoot the error, refer to aka.ms/arc-rb-error   { _errorCode_: _ContextError_, _errorResponse_: _{\n\_message\_: \_Context timed out during phase _WaitingForProviderSpecifics_\_\n}_, _errorMetadata_: { _errorCategory_: __, _errorAdditionalInfos_: null } }
```

This error indicates that the deployment process stalled while waiting for provider-specific pods like vSphere to become ready and eventually timed out. The issue is often related to a variable in the environment or network or storage configuration. In most cases, the required connections are closing, and you can't make progress downloading container images.

The error can also be caused by slow network speed causing a timeout during the container download. Ensure that the appliance VM has a stable connection to required endpoints. The lowest network bandwidth validated for deployment of Arc resource bridge is 100 Mbps. Lower speeds might result in deployment failures. You can attempt a retry in the event that the slow speed is intermittent.

### Context timed out during phase `WaitingForCluster`

When you deploy Arc resource bridge, you might see this error:

```output
Context timed out during phase 'WaitingForCluster'
```

This error indicates that the deployment process stalled while waiting for the Kubernetes cluster components to initialize. It typically occurs during the early stages of appliance setup and is often linked to environmental or configuration issues.

The most frequent cause is the appliance VM's inability to reach your private cloud endpoint—such as the vSphere or vCenter server. Ensure you meet all networking requirements and try the deployment again.


### 403 Forbidden or 404 Site Not Found

When you deploy Arc resource bridge, you might see this error:

```output
{ _errorCode_: _UploadError_, _errorResponse_: _{\n\_message\_: \_Pre-deployment validation of your download/upload connectivity was not successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_403 Forbidden` or `{ _errorCode_: _UploadError_, _errorResponse_: _{\n\_message\_: \_Pre-deployment validation of your download/upload connectivity was not successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_404 Site Not Found
```

This error occurs when the deployment machine needs to download images from Microsoft registries, but a proxy or firewall blocks the download. For more information, see [network requirements](network-requirements.md#general-network-requirements) and verify that you can reach all required URLs. You might need to update your no proxy settings to ensure that traffic from your deployment machine to Microsoft required URLs doesn't go through a proxy.

### SSH folder access denied

The CLI requires permission to access the SSH folder during deployment or operations that involve accessing files within the folder. This folder contains essential files such as the kubeconfig and logs key for the appliance VM. For example, the CLI needs to access the logs key stored in the SSH folder to collect logs from the appliance VM.

You might see this error:

```output
Access to the file in the SSH folder was denied. This may occur if the CLI doesn't have permission to the SSH folder or if another CLI instance is using the file
```

Two common causes for this issue are:

- Insufficient permissions: The CLI lacks the necessary permissions to access the SSH folder. Ensure that the user account running the CLI has appropriate permissions to access the SSH folder.
- Concurrent file access: Another instance of the CLI might be using the file in the SSH folder. This condition often happens on workstations with shared profiles. Ensure that any other CLI instance completes or terminates its operation before you proceed.

### Arc resource bridge is offline

Arc resource bridge might be offline for several reasons. In general, if Arc resource bridge can't communicate with Azure, the appliance VM goes offline. For Arc-enabled VMware and SCVMM, you might need to [update the credentials stored within Arc resource bridge](maintenance.md#update-credentials-in-the-appliance-vm). Networking changes in the infrastructure, environment, or cluster might affect communication to Azure. If you can't determine what changed, you can reboot the appliance VM, collect logs, and submit a support ticket for investigation. As a best practice, [create a resource health alert](maintenance.md#create-resource-health-alerts) to stay informed if an Arc resource bridge becomes unavailable. Arc resource bridge can't be offline for longer than 45 days. After 45 days, the security key within the appliance VM might no longer be valid and can't be refreshed. If you can't get Arc resource bridge back online, contact Microsoft Support.

### Remote PowerShell isn't supported

If you run `az arcappliance` CLI commands for Arc resource bridge via remote PowerShell, you might see an authentication handshake failure error when trying to install the resource bridge on an Azure Local instance or another type of error.

Using `az arcappliance` commands from remote PowerShell isn't currently supported. Instead, sign in to the node through Remote Desktop Protocol (RDP) or use a console session.

### Appliance Network Unavailable

If Arc resource bridge experiences network problems, you might see an `Appliance Network Unavailable` error. In general, any network or infrastructure connectivity issue to the appliance VM might cause this error. This error can also surface as `Error while dialing dial tcp xx.xx.xxx.xx:55000: connect: no route to host`. The problem could be that communication from the host to the Arc resource bridge VM needs to be opened over TCP port 22 with the help of your network administrator. A temporary network issue might prevent the host from reaching the Arc resource bridge VM. Once the network issue is resolved, you can retry the operation. You can also check that the appliance VM for Arc resource bridge isn't stopped or offline. With Azure Local, this error can be caused when the host storage is full.

### Token refresh error

When you run Azure CLI commands, you might see the following error:

```output
The refresh token has expired or is invalid due to sign-in frequency checks by conditional access.
```

This error occurs because when you sign in to Azure, the token has a maximum lifetime. When that lifetime is exceeded, you need to sign in to Azure again by using the `az login` command.

### Default host resource pools are unavailable for deployment

When you use the `az arcappliance createconfig` or `az arcappliance run` command, an interactive experience shows the list of VMware entities that you can select to deploy the virtual appliance. This list shows all user-created resource pools along with default cluster resource pools, but it doesn't list the default host resource pools. When you deploy the appliance to a host resource pool, there's no high availability if the host hardware fails. Don't deploy the appliance in a host resource pool.

### Expired credentials in the appliance VM

Arc resource bridge includes an appliance VM that you deploy to your on-premises infrastructure. The appliance VM uses locally stored credentials to maintain a connection to the management endpoint, such as VMware vCenter, on the on-premises infrastructure. If you don't update these credentials, the resource bridge can't communicate with the management endpoint. This limitation can cause problems when you try to upgrade the resource bridge or manage VMs through Azure.

To fix this problem, update the credentials in the appliance VM. For more information, see [Update credentials in the appliance VM](maintenance.md#update-credentials-in-the-appliance-vm).

### Private link is unsupported

Arc resource bridge doesn't support private link. Calls from the appliance VM shouldn't go through your private link setup. Private link IPs might conflict with the appliance IP pool range, which you can't configure on the resource bridge. Arc resource bridge reaches out to [required URLs](network-requirements.md#firewallproxy-url-allow-list) that shouldn't go through a private link connection. You must deploy Arc resource bridge on a separate network segment that's unrelated to the private link setup.

### Unapproved extension installation

Arc resource bridge is a locked-down virtual appliance that hosts only approved Azure Arc-enabled private cloud extensions. If you attempt to install any other extension on the resource bridge, you receive an error:

```output
Extension installation failed. The specified extension is not permitted on Azure Arc Resource Bridge. Only Azure Arc resource bridge approved extensions can be installed.
```

### Error downloading file release information

When you attempt to upgrade the Arc resource bridge, you might encounter the following error:

```output
'az arcappliance upgrade hci' failed: (DownloadError) "{\n\"message\": \"Error downloading file release information.: Unable to find file release: ^mariner-2-0-(.*)-vhdx-rpm-(.*)$ with version:  in product release: arc-appliance-stable-releases\"\n}",
```

This error occurs when you use an older version of the Azure CLI `arcappliance` extension that isn't forward-compatible with the upgraded version of Arc resource bridge. Before upgrading, update your Azure CLI extension for `arcappliance` by running the following Azure CLI command:

```azurecli
az extension add --upgrade --name arcappliance
```

### GLIBC version not found

You might receive the following error when deploying Arc resource bridge:

```output
"error_message": /lib64/libc.so.6: version `GLIBC_2.34_ not found (required by /root/.azure/cliextensions/arcappliance/azext_arcappliance/pkg/providers/kva//../../binaries/arcsdk.so)
```

This error message indicates that the Arc Resource Bridge CLI extension (arcappliance) tries to load a shared library (arcsdk.so) that was compiled against glibc 2.34, but your Linux system has an older version of glibc or doesn't have the required glibc version. This condition might happen if you're running an old version of Linux. You can check the current glibc version by using `ldd --version`. Use a supported Linux distribution with the required glibc version or onboard from a jumpbox or client VM that meets the glibc requirement.

## Networking issues

### Back-off pulling image error

When you try to deploy Arc resource bridge, you might see an error that contains `back-off pulling image \\\"url"\\\: FailFastPodCondition`. This error happens when the appliance VM can't reach the URL specified in the error. To fix this issue, ensure the appliance VM meets system requirements, including internet access connectivity to [required allow list URLs](network-requirements.md).

### Management machine can't reach appliance

When you try to deploy Arc resource bridge, you might receive an error message similar to:

```output
{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_Timeout occurred due to management machine being unable to reach the appliance VM IP, 10.2.196.170.  Ensure that the requirements are met: https://aka.ms/arb-machine-reqs: dial tcp 10.2.196.170:22: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.\_\n}_, _errorMetadata_: { _errorCategory_: __ }
```

This error happens when the management machine can't reach the Arc resource bridge VM IP by SSH (Port 22) or API Server (Port 6443). It could also happen if the Arc resource bridge API server is being proxied. You need to add the Arc resource bridge API server to the no-proxy settings. For more information, see [Azure Arc resource bridge network requirements](network-requirements.md#inbound-connectivity-requirements).

### Not able to connect to URL

If you receive an error that contains `Not able to connect to https://example.url.com`, check with your network administrator to ensure your network allows all of the required firewall and proxy URLs to deploy Arc resource bridge. For more information, see [Azure Arc resource bridge network requirements](network-requirements.md).

### Not able to connect - network and internet connectivity validation failed

When you deploy Arc resource bridge, you might receive an error with `errorCode` as `PostOperationsError`, `errorResponse` as code `GuestInternetConnectivityError` with a URL specifying port 53 (DNS). This error might happen because the appliance VM IPs can't reach DNS servers, so they can't resolve the endpoint specified in the error.

Error examples:

```output
{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_:\\\_GuestInternetConnectivityError\\\_,\\n\\\_message\\\_:\\\_Not able to connect to http://aszhcitest01.company.org:55000. Error returned: action failed after 5 attempts: Get \\\\\\\_http://aszhcitest01.company.org:55000\\\\\\\_: dial tcp: lookup aszhcitest01.company.org on 127.0.0.53:53: read udp 127.0.0.1:32975-\\u003e127.0.0.53:53: i/o timeout. Arc Resource Bridge network and internet connectivity validation failed: cloud-agent-connectivity-test. 1.  check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings\\\_\\n }\_\n}_ }
```

```output
{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_: \\\_GuestInternetConnectivityError\\\_,\\n  \\\_message\\\_: \\\_Not able to connect to https://linuxgeneva-microsoft.azurecr.io. Error returned: action failed after 5 attempts: Get \\\\\\\_https://linuxgeneva-microsoft.azurecr.io\\\\\\\_: dial tcp: lookup linuxgeneva-microsoft.azurecr.io on 127.0.0.53:53: server misbehaving. Arc Resource Bridge network and internet connectivity validation failed: http-connectivity-test-arc. 1. Please check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings\\\_\\n }\_\n}_ }
```

To fix these errors, work with your network administrator to allow the appliance VM IPs to reach the DNS servers. For more information, see [Azure Arc resource bridge network requirements](network-requirements.md).

### HTTP/2 server sent GOAWAY

When you try to deploy Arc resource bridge, you might receive error messages similar to the following examples:

```output
"errorResponse": "{\n\"message\": \"Post \\\"https://region.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview\\u0026releaseTrain=stable\\\": http2: server sent GOAWAY and closed the connection; LastStreamID=1, ErrCode=NO_ERROR, debug=\\\"\\\"\"\n}"
```

```output
Post \_https://canadacentral.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview\u0026releaseTrain=stable\_: read tcp 10.128.131.173:52425-\u003e52.228.84.81:443: wsarecv: An existing connection was forcibly closed by the remote host.
```

These errors might occur when a firewall or proxy has SSL/TLS inspection enabled and blocks HTTP/2 calls from the machine you use to deploy the resource bridge. To confirm the problem, run the following PowerShell cmdlet to invoke the web request with HTTP/2 (requires PowerShell version 7 or later), replacing the region in the URL and `api-version` (for example, `2019-11-01`) with values from the error:

```output
Invoke-WebRequest -HttpVersion 2.0 -UseBasicParsing -Uri https://region.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview"&"releaseTrain=stable -Method Post -Verbose
```

If the result is `The response ended prematurely while waiting for the next frame from the server`, then the HTTP/2 call is blocked and you need to allow it. Work with your network administrator to disable the SSL/TLS inspection to allow HTTP/2 calls from the machine you use to deploy the bridge.

### No such host - `.local` not supported

When you try to set the configuration for Arc resource bridge, you might receive an error message similar to:

```output
"message": "Post \"https://esx.lab.local/52c-acac707ce02c/disk-0.vmdk\": dial tcp: lookup esx.lab.local: no such host"
```

This error occurs when you provide a `.local` path for a configuration setting, such as proxy, DNS, datastore, or management endpoint (such as vCenter). Arc resource bridge appliance VM uses Azure Linux OS, which doesn't support `.local` by default. A workaround could be to provide the IP address where applicable.

### Azure Arc resource bridge is unreachable

Azure Arc resource bridge runs a Kubernetes cluster, and its control plane requires a static IP address. You specify the IP address in the `infra.yaml` file. If you assign the IP address from a DHCP server, the address can change if you don't reserve it. Rebooting the Azure Arc resource bridge or VM can trigger an IP address change and result in failing services.

Arc resource bridge might intermittently lose the reserved IP configuration. This loss is due to the behavior described in [loss of VIPs when `systemd-networkd` is restarted](https://github.com/acassen/keepalived/issues/1385). When the IP address isn't assigned to the Azure Arc resource bridge VM, any call to the resource bridge API server fails. Core operations, such as creating a new resource, connecting to your private cloud from Azure, or creating a custom location, don't function as expected.

To resolve this issue, reboot the resource bridge VM, and it should recover its IP address. If the address is assigned from a DHCP server, reserve the IP address associated with the resource bridge.

The Arc resource bridge might also be unreachable due to slow disk access. Azure Arc resource bridge uses Kubernetes extended configuration tree (ETCD), which requires [latency of 10 ms or less](https://docs.openshift.com/container-platform/4.6/scalability_and_performance/recommended-host-practices.html#recommended-etcd-practices_). If the underlying disk has low performance, operations are impacted and failures can occur.

### SSL proxy configuration issues

Ensure that the proxy server on your management machine trusts both the SSL certificate for your SSL proxy and the SSL certificate of the Microsoft download servers. For more information, see [SSL proxy configuration](network-requirements.md#ssl-proxy-configuration).

### No such host - `dp.kubernetesconfiguration.azure.com`

When deploying Arc resource bridge, you might receive an error message similar to:

```output
{ _message_: _Post \_https://eastus.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview\u0026releaseTrain=stable\_: dial tcp: lookup eastus.dp.kubernetesconfiguration.azure.com: no such host_ }
```

The error message indicates an issue reaching the URL in the error message, in this case, `eastus.dp.kubernetesconfiguration.azure.com`. This error might occur for several reasons:

- The configuration data plane might be temporarily unavailable in the specified region.
- DNS resolution issue to the `*.dp.kubernetesconfiguration.azure.com` endpoint.
- Network reachability error to the `*.dp.kubernetesconfiguration.azure.com` endpoint.

Recommended actions:
- Wait for the service to be available, and then retry the deployment.
- Verify DNS server settings on the host.
- Confirm that the firewall or proxy doesn't block outbound internet access to the endpoint.


### Certificate signed by unknown authority

You might encounter the following error when deploying Arc resource bridge:

```output
"errorResponse": "{\n\"message\": \"{\\n  \\\"code\\\": \\\"GuestInternetConnectivityError\\\",\\n  \\\"message\\\": \\\"Name: http-connectivity-test-arc. Message: Not able to connect to https://msk8s.api.cdp.microsoft.com. Error returned: action failed after 5 attempts: Get \\\\\\\"https://msk8s.api.cdp.microsoft.com\\\\\\\": **tls: failed to verify certificate: x509: certificate signed by unknown authority.** Arc Resource Bridge network and internet connectivity validation failed: http-connectivity-test-arc. 1. Please check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings\\\",\\n  \\\"category\\\": \\\"\\\"\\n }\"\n}",
```

This error occurs when SSL inspection happens within the network and prevents HTTPS/SSL trust from being established with the endpoint referenced in the error. This error most commonly occurs with an SSL proxy server that does SSL inspection or termination, intercepts the connection to the endpoint, and breaks the connectivity. If you don't configure a proxy server during deployment, your network might have a transparent proxy or a network security device that interferes with this connection. Work with your networking team to debug the cause by using your proxy, firewall, or security device logs.

### Proxy connect tcp - No such host for Arc resource bridge required URL

An error that contains an Arc resource bridge required URL with the message `proxyconnect tcp: dial tcp: lookup http: no such host` indicates that DNS can't resolve the URL. The error might look similar to this example, where the required URL is `https://msk8s.api.cdp.microsoft.com`:

```output
Error:  { _errorCode_: _InvalidEntityError_, _errorResponse_: _{\n\_message\_: \_Post \\\_https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/arc-appliance-stable-catalogs-ext/versions/latest?action=select\\\_: POST https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/arc-appliance-stable-catalogs-ext/versions/latest?action=select giving up after 6 attempt(s): Post \\\_https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/arc-appliance-stable-catalogs-ext/versions/latest?action=select\\\_: proxyconnect tcp: dial tcp: lookup http: no such host\_\n}_ }
```

This error can occur if the deployment DNS settings aren't correct or there's a problem with the DNS servers. Check if your DNS server can resolve the URL by running the following command from the management machine or a machine that has access to the DNS servers:

```console
nslookup
> set debug
> <hostname> <DNS server IP>
```

To resolve the error, configure your DNS servers to resolve all Arc resource bridge required URLs. Provide the DNS servers correctly when you deploy Arc resource bridge.

### KVA timeout error

The KVA timeout error is a generic error caused by various network misconfigurations that involve the management machine, For instance, the appliance VM or Control Plane IP might not have communication with each other, to the internet, or required URLs. These communication failures are often due to issues with DNS resolution, proxy settings, network configuration, or internet access.

For clarity, management machine refers to the machine where deployment CLI commands are being run. Appliance VM is the VM that hosts Arc resource bridge. Control Plane IP is the IP of the control plane for the Kubernetes management cluster in the Appliance VM.

#### Top causes of the KVA timeout error 

- Management machine is unable to communicate with Control Plane IP and Appliance VM IP.
- Appliance VM is unable to communicate with the management machine, vCenter endpoint (for VMware), or MOC cloud agent endpoint (for Azure Local). 
- Appliance VM doesn't have internet access.
- Appliance VM has internet access, but connectivity to one or more required URLs is being blocked, possibly due to a proxy or firewall.
- Appliance VM is unable to reach a DNS server that can resolve internal names, such as vCenter endpoint for vSphere or cloud agent endpoint for Azure Local. The DNS server must also be able to resolve external addresses, such as Azure service addresses and container registry names. 
- Proxy server configuration on the management machine or Arc resource bridge configuration files is incorrect. This can impact both the management machine and the Appliance VM. When the `az arcappliance prepare` command is run and the host proxy isn't correctly configured, the management machine can't connect and download OS images. Internet access on the Appliance VM might be broken by incorrect or missing proxy configuration, which impacts the VM's ability to pull container images. 

#### Troubleshoot KVA timeout error

To resolve the error, one or more network misconfigurations might need to be addressed.

- The first step is to collect logs by Appliance VM IP (not by kubeconfig, as the kubeconfig could be empty if the deploy command didn't complete). Problems collecting logs are most likely due to the management machine being unable to reach the Appliance VM.

  After logs are collected, extract the folder and open `kva.log`. Review the log for information that might help pinpoint the cause of the KVA timeout error.

- The management machine must be able to communicate with the Appliance VM IP and Control Plane IP. Ping the Control Plane IP and Appliance VM IP from the management machine and verify that there's a response from both IPs.

  If a request times out, the management machine can't communicate with the IPs. This issue might be caused by a closed port, network misconfiguration, or firewall block. Work with your network administrator to allow communication between the management machine to the Control Plane IP and Appliance VM IP.

- Appliance VM IP and Control Plane IP must be able to communicate with the management machine and vCenter endpoint (for VMware) or MOC cloud agent endpoint (for Azure Local). Work with your network administrator to ensure the network is configured to permit this communication. You might need to add a firewall rule to open port 443 from the Appliance VM IP and Control Plane IP to vCenter, or to open port 65000 and 55000 for Azure Local MOC cloud agent. For more infromation, see [network requirements for Azure Local](/azure/azure-local/manage/azure-arc-vm-management-prerequisites#network-port-requirements) and [VMware](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md) for Arc resource bridge.

- Appliance VM IP and Control Plane IP need internet access to [these required URLs](#not-able-to-connect-to-url). Azure Local requires [additional URLs](/azure/azure-local/manage/azure-arc-vm-management-prerequisites). Work with your network administrator to ensure that the IPs can access the required URLs.

- In a non-proxy environment, the management machine must have external and internal DNS resolution. The management machine must be able to reach a DNS server that can resolve internal names such as vCenter endpoint for vSphere or cloud agent endpoint for Azure Local. The DNS server also needs to be able to [resolve external addresses](#not-able-to-connect-to-url), such as Azure URLs and OS image download URLs. Work with your system administrator to ensure that the management machine has internal and external DNS resolution. In a proxy environment, the DNS resolution on the proxy server should resolve internal endpoints and [required external addresses](#not-able-to-connect-to-url).

  To test DNS resolution to an internal address from the management machine in a non-proxy scenario, open a command prompt and run `nslookup <vCenter endpoint or HCI MOC cloud agent IP>`. You should receive an answer if the management machine has internal DNS resolution in a non-proxy scenario. 

- Appliance VM needs to be able to reach a DNS server that can resolve internal names, such as vCenter endpoint for vSphere or cloud agent endpoint for Azure Local. The DNS server also needs to be able to resolve external/internal addresses, such as Azure service addresses and container registry names for download of the Arc resource bridge container images from the cloud.

  Verify that the DNS server IP used to create the configuration files has internal and external address resolution.

### Moving the Arc Resource Bridge VM isn't supported

Arc resource bridge and its underlying layers reference the original deployment path. Updating the config YAML file or manually relocating the VM in your management console doesn't update these internal references. As a result, making these changes can lead to upgrade failures or errors such as [cannot retrieve resource](#cannot-retrieve-resource---resource-not-found-or-doesnt-exist).

If you need to change the location of the resource bridge VM, redeploy the resource bridge in the desired location. Follow the recovery guidance for your environment:

- For Arc-enabled VMware, follow the [Arc VMware recovery guide](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion).
- For Arc-enabled SCVMM, follow the [Arc SCVMM recovery guide](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery).

## Azure Arc-enabled VMs on Azure Local issues

For general help resolving issues related to Azure Arc-enabled VMs on Azure Local, see [Troubleshoot Azure Arc VM management for Azure Local](/azure/azure-local/manage/troubleshoot-arc-enabled-vms).

If you're running Azure Local, version 23H2 or later, and your Arc Resource Bridge is offline, try restarting the Arc Resource Bridge VM to bring it back online. If the issue persists, contact [Microsoft Support](https://support.microsoft.com) for assistance. Don't delete the Arc Resource Bridge VM without guidance from Microsoft Support.

### Action failed - no such host

When you deploy Arc resource bridge, you might receive an error with `errorCode` as `PostOperationsError`, `errorResponse` as code `GuestInternetConnectivityError`, and `no such host`. The appliance VM IPs might not be able to reach the endpoint specified in the error.

Error example:

```output
{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_: \\\_GuestInternetConnectivityError\\\_,\\n  \\\_message\\\_: \\\_Not able to connect to http://aszhcitest01.company.org:55000. Error returned: action failed after 5 attempts: Get \\\\\\\_http://aszhcitest01.company.org:55000\\\\\\\_: dial tcp: lookup aszhcitest01.company.org: on 127.0.0.53:53: no such host. Arc Resource Bridge network and internet connectivity validation failed: cloud-agent-connectivity-test.
1. check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.
2. Check firewall/proxy settings
```

In the following example, the appliance VM IPs can't access `http://aszhcitest01.company.org:55000`, which is the MOC endpoint. Work with your network administrator to ensure that the DNS server can resolve the required URLs.

To test connectivity to the DNS server, run:

```console
ping <dns-server.com>
```

To check if the DNS server can resolve an address, run this command from a machine that can reach the DNS servers:

```console
Resolve-DnsName -Name "http://aszhcitest01.company.org:55000" -Server "<dns-server.com>"
```

### Authentication required

You might receive the following error when deploying Arc resource bridge:

```output
{ _message_: _Post \_https://westeurope.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview\u0026releaseTrain=stable\_: authenticationrequired_ }
```
This error likely occurs because a proxy intercepts the request and requires authentication. To successfully run Azure CLI behind this type of proxy, ensure proper proxy support.

Recommended actions:

1. Confirm whether a proxy is active in your environment.
1. If so, configure environment variables (HTTPS_PROXY, HTTP_PROXY, and optionally NO_PROXY) with authentication credentials if required. Refer to [Azure Arc resource bridge network requirements](network-requirements.md#ssl-proxy-configuration).
1. You might also need to ensure that Azure CLI can work behind a proxy. For detailed instructions, see [Azure CLI proxy troubleshooting guide](/cli/azure/use-azure-cli-successfully-troubleshooting#work-behind-a-proxy).

## Azure Arc-enabled VMware vCenter issues

### `errorResponse: error getting the vsphere sdk client`

You see errors with `errorCode: CreateConfigKvaCustomerError` and `errorResponse: error getting the vsphere sdk client` when your deployment machine tries to establish a TCP connection to your vCenter address but encounters a problem. This problem can occur if your vCenter address is incorrect (403 or 404 error), or if a network, proxy, or firewall configuration blocks the connection (connection attempt failed).

If you enter your vCenter address as a hostname and receive the error `no such host`, your deployment machine can't resolve the vCenter hostname through the client DNS. This problem might occur if the deployment machine can resolve the vCenter hostname but can't reach the IP address it received from DNS. You might also see this error if the endpoint returned by DNS isn't your vCenter address, or if a proxy intercepted the traffic. If your deployment machine can communicate with your vCenter address, confirm that your username and password are correct.

### vSphere SDK client - Connection attempt failed

Your management machine can't communicate with your vCenter server if you receive an error during deployment that states:

```output
errorCode_: _CreateConfigKvaCustomerError_, _errorResponse_: _error getting the vsphere sdk client: Post \_https://ip.address/sdk\_: dial tcp ip.address:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond._ }
```

To resolve this issue, ensure that your management machine meets the [management machine requirements](system-requirements.md#management-machine-requirements) and that there's no firewall or proxy blocking communication.

### vSphere SDK client - 403 Forbidden or 404 not found

Errors that contain the following error while deploying Arc resource bridge most likely occur because of an incorrect vCenter address. You provide this address during configuration file creation, when you're prompted to enter the vCenter address as either a hostname or IP address.

```output
errorCode_: _CreateConfigKvaCustomerError_, _errorResponse_: _error getting the vsphere sdk client: POST \_/sdk\_: 403 Forbidden` or `404 not found
```

You can find your vCenter address in different ways. One option is to access the vSphere client through its web interface. You typically use the vCenter hostname or IP address in the browser to access the vSphere client. If you're already signed in, you can view the browser's address bar, where the URL you use to access vSphere is your vCenter server's hostname or IP address. Verify your vCenter address, and then try the deployment again.

### vSphere SDK client - no such host

This error can occur during deployment if the deployment machine can't resolve the vCenter hostname to an IP address. This issue arises because the deployment process is attempting to establish a TCP connection from your deployment machine to the vCenter hostname, but the connection fails due to DNS resolution problems.

```output
{ _errorCode_: _CreateConfigKvaCustomerError_, _errorResponse_: _error getting the vsphere sdk client: Post \_https://your.vcenter.hostname/sdk\_: dial tcp: lookup your.vcenter.hostname: no such host_ }
```

To fix this error, ensure the DNS configuration on your deployment machine is correct, verify that the DNS server is online, and check for a missing DNS entry for the vCenter hostname. You can test the DNS resolution by running `nslookup your.vcenter.hostname` or `ping your.vcenter.hostname` from the deployment machine. If you specified your vCenter address as a hostname, consider using the IP address directly instead.

### Predeployment validation errors

When you deploy Arc resource bridge, you might see various `pre-deployment validation of your download\upload connectivity wasn't successful` errors, such as:

```output
Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: Service Unavailable
```

```output
Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: dial tcp 172.16.60.10:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

```output
Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: use of closed network connection.
```

```output
Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: dial tcp: lookup hostname.domain: no such host
```

A combination of these errors usually indicates that the management machine lost connection to the datastore, or that a networking issue causes the datastore to be unreachable. You need this connection to upload the OVA from the management machine that builds the appliance VM in vCenter.

To fix the issue, reestablish the connection between the management machine and datastore, and then try deploying Arc resource bridge again.

### Time difference causing x509 certificate has expired

When you deploy Arc resource bridge, you might encounter the error:

```output
Error: { _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_: \\\_GuestInternetConnectivityError\\\_,\\n  \\\_message\\\_: \\\_Not able to connect to https://msk8s.api.cdp.microsoft.com. Error returned: action failed after 3 attempts: Get \\\\\\\_https://msk8s.api.cdp.microsoft.com\\\\\\\_: x509: certificate has expired or isn't yet valid: current time 2022-01-18T11:35:56Z is before 2023-09-07T19:13:21Z. Arc Resource Bridge network and internet connectivity validation failed: http-connectivity-test-arc. 1.  check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings
```

This error occurs when there's a time difference between ESXi hosts and the management machine running the deployment commands for Arc resource bridge. To resolve this issue, turn on NTP time sync on the ESXi hosts, confirm that the management machine is also synced to NTP, and then try the deployment again.

### Clock skew error between appliance VM and management machine

If you encounter an error similar to the following:

```output
"ErrorCode": "PostOperationsError", "errorResponse": "{\n\"message\": \"{\\n  \\\"code\\\": \\\"ClockSkewError\\\",\\n  \\\"message\\\": \\\"The time in Appliance VM is too far behind in the past compared to Management Machine : Time in Appliance VM is 2025-02-24T10:59:59Z, time in Management Machine is 2025-02-24T16:49:13Z. Max allowed difference is 30m0s. Recommendation: Please verify that the time of the workstation machine and the appliance VM are in sync.
```

This error occurs when there's a time difference between ESXi hosts and the management machine running the deployment commands for Arc resource bridge. To resolve this issue, turn on NTP time sync on the ESXi hosts, confirm that the management machine is also synced to NTP, and then try the deployment again.

### Resolves to multiple networks

When you deploy or upgrade Arc resource bridge, you might encounter an error similar to:

```output
{ "ErrorCode": "PreflightcheckErrorOnPrem",
"ErrorDetails": "Upgrade Operation Failed with error: \"{\\n \\\"code\\\": \\\"PreflightcheckError\\\",\\n \\\"message\\\": \\\"{\\\\n \\\\\\\"code\\\\\\\": \\\\\\\"InvalidEntityError\\\\\\\",\\\\n \\\\\\\"message\\\\\\\": \\\\\\\"Cannot retrieve vSphere Network 'vmware-azure-arc-01': path 'vmware-azure-arc-01' resolves to multiple networks\\\\\\\",\\\\n \\\\\\\"category\\\\\\\": \\\\\\\"\\\\\\\"\\\\n }\\\",\\n \\\"category\\\": \\\"\\\"\\n }\"" }
```

This error occurs when the vSphere network segment resolves to multiple networks, due to multiple vSphere network segments using the same name that is specified in the error. To fix this error, change the duplicate network name in vCenter (not the network with the appliance VM) or deploy Arc resource bridge on a different network.

### Arc resource bridge status is disconnected

When running the initial Arc-enabled VMware onboarding script, you're prompted to provide a vSphere account. The script stores this account locally within the Arc resource bridge as an encrypted Kubernetes secret. The Arc resource bridge uses the account to interact with vCenter.

If the vSphere account stored locally within the resource bridge expires, your Arc resource bridge status can become disconnected. Update the credentials within Arc resource bridge and for Arc-enabled VMware by [following the updating vSphere account credentials instructions](/azure/azure-arc/vmware-vsphere/administer-arc-vmware#updating-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding).

### Error during host configuration

If you use the same template to deploy and delete the Arc resource bridge multiple times, you might encounter the following error:

```output
Appliance cluster deployment failed with error: Error: An error occurred during host configuration
```

To resolve this issue, manually delete the existing template. Then run [`az arcappliance prepare`](/cli/azure/arcappliance/prepare) to download a new template for deployment.

### Unable to find folders

When you deploy Arc resource bridge on VMware, you specify the folder in which to create the template and VM. The selected folder must be a VM and template folder type. You can't use other types of folder, such as storage folders, network folders, or host and cluster folders, for the resource bridge deployment.

### Cannot retrieve resource - resource not found or doesn't exist

When you deploy Arc resource bridge, you specify where to deploy the appliance VM as its location path. You can't move the appliance VM from that location path. If any component within that path changes, such as the datastore or resource pool, the appliance VM loses its Azure connection. If you change the Arc resource bridge location and try to upgrade, you might see errors similar to the following:

```output
{\n  \"code\": \"PreflightcheckError\",\n  \"message\": \"{\\n  \\\"code\\\": \\\"InvalidEntityError\\\",\\n  \\\"message\\\": \\\"Cannot retrieve <resource> 'resource-name': <resource> 'resource-name' not found\\\"\\n }\"\n }"
```

```output
{\n  \"code\": \"PreflightcheckError\",\n  \"message\": \"{\\n  \\\"code\\\": \\\"InvalidEntityError\\\",\\n  \\\"message\\\": \\\"The specified vSphere Datacenter '/VxRail-Datacenter' does not exist\\\"\\n }\"\n }"
```

To fix these errors, use one of these options:

- Move the appliance VM back to its original location and ensure RBAC credentials are updated for the location change.
- Create a resource with the same name, then move Arc resource bridge to that new resource, ensuring you recreate the original location path.
- For Arc-enabled VMware, [run the Arc-enabled VMware disaster recovery script](../vmware-vsphere/disaster-recovery.md). The script deletes the appliance, deploys a new appliance, and reconnects the appliance with the previously deployed custom location, cluster extension, and Arc-enabled VMs.

### vCenter account is locked out - Update credentials

Arc resource bridge uses the vCenter account you provide during initial deployment to connect to vCenter. If you update the vCenter account but don't update the corresponding account info in Arc resource bridge, the account might lock out. To immediately update the credentials without waiting for the lockout period to expire, run the following command with the `--skipWait` flag:

```azurecli
az arcappliance update-infracredentials vmware \
  --kubeconfig [REQUIRED] \
  --address [REQUIRED] \
  --username [REQUIRED] \
  --password [REQUIRED] \
  --skipWait
```

If you need to retrieve the kubeconfig, you can run the following command:

```azurecli
az arcappliance get-credentials \
  --resource-group [REQUIRED] \
  --name [REQUIRED] \
  --credentials-dir [OPTIONAL]
```

> [!NOTE]
> You might also need to update the vCenter credentials for the Arc-enabled VMware cluster extension installed on the Arc resource bridge. For more information, see [Update the vSphere account credentials](/azure/azure-arc/vmware-vsphere/administer-arc-vmware?tabs=account-for-arc-resource-bridge#update-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding).

### Insufficient privileges

When you deploy or upgrade the resource bridge on VMware vCenter, you might see an error similar to:

```output
{  ""code"": ""PreflightcheckError"", ""message"": ""{\n  \""code\"": \""InsufficientPrivilegesError\"",\n  \""message\"": \""The provided vCenter account is missing required vSphere privileges on the resource 'root folder (MoRefId: Folder:group-d1)'. Missing privileges: [Sessions.ValidateSession].  add the privileges to the vCenter account and try again. To review the full list of required privileges, go to https://aka.ms/ARB-vsphere-privilege.\""\n }
```

When you deploy Arc resource bridge, you provide vCenter credentials. Arc resource bridge stores these vCenter credentials locally to interact with vCenter. To resolve the missing privileges issue, the vCenter account used by the resource bridge needs the following privileges in VMware vCenter:

**Datastore**:

- Allocate space
- Browse datastore
- Low level file operations

**Folder**:

- Create folder

**vSphere Tagging**:

- Assign or Unassign vSphere Tag

**Network**:

- Assign network

**Resource**:

- Assign virtual machine to resource pool
- Migrate powered off virtual machine
- Migrate powered on virtual machine

**Sessions**:

- Validate session

**vApp**:

- Assign resource pool
- Import

**Virtual machine**:

- Change Configuration
  - Acquire disk lease
  - Add existing disk
  - Add new disk
  - Add or remove device
  - Advanced configuration
  - Change CPU count
  - Change Memory
  - Change Settings
  - Change resource
  - Configure managedBy
  - Display connection settings
  - Extend virtual disk
  - Modify device settings
  - Query Fault Tolerance compatibility
  - Query unowned files
  - Reload from path
  - Remove disk
  - Rename
  - Reset guest information
  - Set annotation
  - Toggle disk change tracking
  - Toggle fork parent
  - Upgrade virtual machine compatibility
- Edit Inventory
  - Create from existing
  - Create new
  - Register
  - Remove
  - Unregister
- Guest operations
  - Guest operation alias modification
  - Guest operation modifications
  - Guest operation program execution
  - Guest operation queries
- Interaction
  - Connect devices
  - Console interaction
  - Guest operating system management by VIX API
  - Install VMware Tools
  - Power off
  - Power on
  - Reset
  - Suspend
- Provisioning
  - Allow disk access
  - Allow file access
  - Allow read-only disk access
  - Allow virtual machine download
  - Allow virtual machine files upload
  - Clone virtual machine
  - Deploy template
  - Mark as template
  - Mark as virtual machine
  - Customize guest
- Snapshot management
  - Create snapshot
  - Remove snapshot
  - Revert to snapshot

## Next steps

[Understand recovery operations for resource bridge in Azure Arc-enabled VMware vSphere disaster scenarios](../vmware-vsphere/disaster-recovery.md)

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Connect with [@AzureSupport](https://x.com/azuresupport), the official Microsoft Azure account for improving customer experience. Azure Support connects the Azure community to answers, support, and experts.
- [Open an Azure support request](../../azure-portal/supportability/how-to-create-azure-support-request.md)
