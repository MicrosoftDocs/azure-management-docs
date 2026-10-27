---
title: "Manage and maintain Azure Arc-enabled Kubernetes agent proxy settings"
description: "Learn how to set up proxy bypass so Azure Arc and Container insights traffic from Azure Arc-enabled Kubernetes clusters skips the proxy server."
ms.date: 10/26/2026
ms.topic: how-to
# Customer intent: "As a system administrator, I want to manage the proxy settings of the Azure Arc agents on my cluster, so that Azure Arc and Container insights traffic can reach private endpoints without going through my proxy server."
---

# Manage and maintain Azure Arc-enabled Kubernetes agent proxy settings

Use the steps in this article to choose which services skip the outbound proxy server that's configured for your Azure Arc-enabled Kubernetes cluster. To connect a cluster through a proxy server, see [Connect using an outbound proxy server](quickstart-connect-cluster.md#connect-using-an-outbound-proxy-server).

## Proxy bypass for private endpoints

Starting with version 1.11.4 of the `connectedk8s` Azure CLI extension, you can specify services that shouldn't use the outbound proxy server configured for your cluster. This configuration helps with split-network designs and private endpoint scenarios where you want Microsoft Entra ID and Azure Resource Manager traffic to go through your proxy server to public endpoints, but you want Azure Arc and Container insights traffic to skip the proxy and communicate with a private IP address on your network.

The proxy bypass feature doesn't require you to enter specific URLs to bypass. Instead, provide the name of any services that shouldn't use the proxy server.

| Proxy bypass value | Affected endpoints |
| --- | --- |
| `Arc` | `*.his.arc.azure.com`<br>`*.dp.kubernetesconfiguration.azure.com`<br>`*.guestconfiguration.azure.com` |
| `Microsoft.AzureMonitor.Containers` | `*.oms.opinsights.azure.com`<br>`*.ods.opinsights.azure.com`<br>`global.handler.control.monitor.azure.com`<br>`dc.services.visualstudio.com` |

The endpoints in the table are for Azure public cloud. In Azure Government and Microsoft Azure operated by 21Vianet, the `Arc` endpoints use each cloud's own domain, for example `*.his.arc.azure.us` in Azure Government. For the Container insights endpoints in those clouds and for private data collection endpoints, see [Network firewall requirements for monitoring Kubernetes clusters](/azure/azure-monitor/containers/kubernetes-monitoring-firewall).

The two values work in different ways:

- `Arc` makes the Azure Arc agents skip the proxy only for the Azure Arc endpoints. You don't need to add these endpoints to `--proxy-skip-range` manually.
- `Microsoft.AzureMonitor.Containers` sets `ignore_proxy_settings` to `"true"` in the `container-azm-ms-agentconfig` ConfigMap in the `kube-system` namespace. If the ConfigMap doesn't exist, the CLI creates it. The Container insights agent (`ama-logs` pods) then ignores the proxy for all of its traffic, so make sure the agent can reach its endpoints without the proxy. The change can take a few minutes to take effect while the agent pods restart. For more information, see [ConfigMap settings](/azure/azure-monitor/containers/kubernetes-data-collection-configmap#configmap-settings).


To send Microsoft Entra ID and Azure Resource Manager traffic through the configured proxy while allowing supported Azure Arc traffic to bypass the proxy, specify `--add-proxy-bypass Arc` when you connect your cluster.

```azurecli
az connectedk8s connect --name <cluster-name> --resource-group <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --add-proxy-bypass Arc
```

If your cluster is already connected, use `az connectedk8s update` instead. To provide a list of services, separate the service names by commas:

```azurecli
az connectedk8s update --name <cluster-name> --resource-group <resource-group> --add-proxy-bypass Arc,Microsoft.AzureMonitor.Containers
```

To clear the proxy bypass, run the following command:

```azurecli
az connectedk8s update --name <cluster-name> --resource-group <resource-group> --clear-proxy-bypass Arc,Microsoft.AzureMonitor.Containers
```

Clearing `Arc` removes only the Azure Arc endpoints that the CLI added. Entries that you set with `--proxy-skip-range` stay in place. Clearing `Microsoft.AzureMonitor.Containers` sets `ignore_proxy_settings` back to `"false"` only if the CLI set it to `"true"`. The CLI doesn't delete the ConfigMap. Removing the Container insights extension doesn't clear this bypass.

When you add `Arc`, the command output lists the Azure Arc endpoints that bypass the proxy. To view the Container insights setting, run the following command:

```bash
kubectl get configmap container-azm-ms-agentconfig --namespace kube-system --output yaml
```

## Related content

- Learn how to [connect an existing Kubernetes cluster to Azure Arc](quickstart-connect-cluster.md).
- Learn how to [use private connectivity for Azure Arc-enabled Kubernetes clusters with private link](private-link.md).
- Learn how to [configure container log collection with a ConfigMap](/azure/azure-monitor/containers/kubernetes-data-collection-configmap).
