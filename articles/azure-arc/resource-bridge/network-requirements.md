---
title: Azure Arc resource bridge network requirements
description: Learn about network requirements for Azure Arc resource bridge including URLs that must be allowlisted.
ms.topic: concept-article
ms.date: 09/20/2024
---

# Azure Arc resource bridge network requirements

This article describes the networking requirements for deploying Azure Arc resource bridge in your enterprise.

## General network requirements

The lowest network bandwidth validated for deployment of Arc resource bridge is 100 mbps. If your network bandwidth is slower, you may experience problems with deployment. 

Arc resource bridge communicates outbound securely to Azure Arc over TCP port 443. If the appliance needs to connect through a firewall or proxy server to communicate over the internet, it communicates outbound using the HTTPS protocol.

[!INCLUDE [network-requirement-principles](../includes/network-requirement-principles.md)]

[!INCLUDE [network-requirements](includes/network-requirements.md)]

> [!NOTE]
> The URLs listed here are required for Arc resource bridge only. Other Arc products (such as Arc-enabled VMware vSphere) may have additional required URLs. For details, see [Azure Arc network requirements](../network-requirements-consolidated.md#azure-arc-enabled-vmware-vsphere).

## Designated IP ranges for Arc resource bridge

When deploying Arc resource bridge, specific IP ranges are reserved exclusively for the Kubernetes pods and services within the appliance VM. These internal IP ranges must not overlap with any configuration inputs for the resource bridge, such as IP address prefix, control plane IP, appliance VM IPs, DNS servers, proxy servers, or vSphere ESXi hosts. For details on the Arc resource bridge configuration, refer to the [system requirements](system-requirements.md#ip-address-prefix-subnet-requirements).

> [!NOTE]
> These designated IP ranges are only used internally within the Arc resource bridge. They don't affect Azure resources or networks.

|      **Service**       |    **Designated IP range**    |  
| ----------------------- | ---------------------------- |
| Arc resource bridge Kubernetes pods   | 10.244.0.0/16 |
| Arc resource bridge Kubernetes services  | 10.96.0.0/12  |


## SSL proxy configuration

> [!IMPORTANT]
> Arc Resource Bridge supports only direct (explicit) proxies, including unauthenticated proxies, proxies with basic authentication, SSL terminating proxies, and SSL passthrough proxies.

If using a proxy, the Arc Resource Bridge must be configured to use the proxy in order to connect to Azure services.

- To configure the Arc resource bridge with proxy, provide the proxy certificate file path during creation of the configuration files.

- The format of the certificate file is *Base-64 encoded X.509 (.CER)*.

- Only pass the single proxy certificate. If a certificate bundle is passed, the deployment will fail.

- The proxy server endpoint can't be a `.local` domain.

- The proxy server has to be reachable from all IPs within the IP address prefix, including the control plane and appliance VM IPs.

There are only two certificates that should be relevant when deploying the Arc resource bridge behind an SSL proxy:

- SSL certificate for your SSL proxy (so that the management machine and appliance VM trust your proxy FQDN and can establish an SSL connection to it)

- SSL certificate of the Microsoft download servers. This certificate must be trusted by your proxy server itself, as the proxy is the one establishing the final connection and needs to trust the endpoint. Non-Windows machines may not trust this second certificate by default, so you may need to ensure that it's trusted.

In order to deploy Arc resource bridge, images need to be downloaded to the management machine and then uploaded to the on-premises private cloud gallery. If your proxy server throttles download speed, you may not be able to download the required images (~3.5 GB) within the allotted time (90 min).

## Exclusion list for no proxy

If a proxy server is being used, the following table contains the list of addresses that should be excluded from proxy by configuring the `noProxy` settings.

|      **IP Address**       |    **Reason for exclusion**    |  
| ----------------------- | ------------------------------------ |
| localhost, 127.0.0.1  | Localhost traffic  |
| .svc | Internal Kubernetes service traffic (.svc) where *.svc* represents a wildcard name. This is similar to saying \*.svc, but none is used in this schema. |
| 10.0.0.0/8 | private network address space |
| 172.16.0.0/12 |Private network address space - Kubernetes Service CIDR |
| 192.168.0.0/16 | Private network address space - Kubernetes Pod CIDR |
| .contoso.com | You may want to exempt your enterprise namespace (.contoso.com) from being directed through the proxy. To exclude all addresses in a domain, you must add the domain to the `noProxy` list. Use a leading period rather than a wildcard (\*) character. In the sample, the addresses `.contoso.com` excludes addresses `prefix1.contoso.com`, `prefix2.contoso.com`, and so on. |

The default value for `noProxy` is `localhost,127.0.0.1,.svc,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16`. While these default values will work for many networks, you may need to add more subnet ranges and/or names to the exemption list. For example, you may want to exempt your enterprise namespace (.contoso.com) from being directed through the proxy. You can achieve that by specifying the values in the `noProxy` list.

> [!IMPORTANT]
> When listing multiple addresses for the `noProxy` settings, don't add a space after each comma to separate the addresses. The addresses must immediately follow the commas.

## Internal port listening

Be aware that the appliance VM is configured to listen on the following ports. These ports are used exclusively for internal processes and do not require external access:

- 8443 – Endpoint for Microsoft Entra Authentication Webhook
- 10257 – Endpoint for Arc resource bridge metrics
- 10250 – Endpoint for Arc resource bridge metrics
- 2382 – Endpoint for Arc resource bridge metrics

## Next steps

- Review the [Azure Arc resource bridge overview](overview.md) to understand more about requirements and technical details.
- Learn about [security configuration and considerations for Azure Arc resource bridge](security-overview.md).
- View [troubleshooting tips for networking issues](troubleshoot-resource-bridge.md#networking-issues).

