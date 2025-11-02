---
ms.service: azure-arc
ms.topic: include
ms.date: 04/15/2025
# Customer intent: As a network administrator, I want to configure outbound network access for Azure Arc agents in the Azure public cloud, so that they can connect to necessary endpoints for proper functionality and management of connected clusters.
---

> [!IMPORTANT]
> Azure Arc agents require the following outbound URLs on `https://:443` to function.
> For `*.servicebus.windows.net`, websockets need to be enabled for outbound access on firewall and proxy.

| Endpoint (DNS) | Description |
| ----------------- | ------------- |
| `https://management.azure.com` | Required for the agent to connect to Azure and register the cluster. |
| `https://<region>.dp.kubernetesconfiguration.azure.com` | Data plane endpoint for the agent to push status and fetch configuration information. |
| `https://login.microsoftonline.com`<br/>`https://<region>.login.microsoft.com`<br/>`login.windows.net`| Required to fetch and update Azure Resource Manager tokens. |
| `https://mcr.microsoft.com`<br/>`https://*.data.mcr.microsoft.com` | Required to pull container images for Azure Arc agents.        |
| `dl.k8s.io` | Required to download kubectl binaries during Azure Arc onboarding by Azure CLI [connectedk8s extension](/cli/azure/connectedk8s). |
| `https://gbl.his.arc.azure.com` |  Required to get the regional endpoint for pulling system-assigned Managed Identity certificates. |
| `https://*.his.arc.azure.com` |  Required to pull system-assigned Managed Identity certificates. |
|`guestnotificationservice.azure.com`<br/>`*.guestnotificationservice.azure.com`<br/>`sts.windows.net` | For [Cluster Connect](../cluster-connect.md) and for [Custom Location](../custom-locations.md) based scenarios. |
|`*.servicebus.windows.net` | For [Cluster Connect](../cluster-connect.md) and for [Custom Location](../custom-locations.md) based scenarios. |
|`https://graph.microsoft.com/` | Required when [Azure RBAC](../azure-rbac.md) is configured. |
| `*.arc.azure.net`| Required to manage connected clusters in Azure portal. |
|`https://<region>.obo.arc.azure.com:8084/` | Required when [Cluster Connect](../cluster-connect.md) and [Azure RBAC](../azure-rbac.md) is configured. |
| `https://linuxgeneva-microsoft.azurecr.io` | Required if using [Azure Arc-enabled Kubernetes extensions](../conceptual-extensions.md).

To translate the `*.servicebus.windows.net` wildcard into specific endpoints, use the command:

```rest
GET https://guestnotificationservice.azure.com/urls/allowlist?api-version=2020-01-01&location=<region>
```

[!INCLUDE [arc-region-note](../../includes/arc-region-note.md)]
