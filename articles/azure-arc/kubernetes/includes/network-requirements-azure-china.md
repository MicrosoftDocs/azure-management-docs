---
ms.service: azure-arc
ms.topic: include
ms.date: 02/18/2025
# Customer intent: "As a system administrator, I want to configure outbound access for Azure Arc agents in Microsoft Azure operated by 21Vianet, so that I can ensure proper connectivity for registration, management, and operation of connected clusters in my environment."
---

> [!IMPORTANT]
> Azure Arc agents require the following outbound URLs on `https://:443` to function.
> For `*.servicebus.chinacloudapi.cn`, websockets need to be enabled for outbound access on firewall and proxy.

| Endpoint (DNS) | Description |
| ----------------- | ------------- |
| `https://management.chinacloudapi.cn` | Required for the agent to connect to Azure and register the cluster. |
| `https://<region>.dp.kubernetesconfiguration.azure.cn` | Data plane endpoint for the agent to push status and fetch configuration information. |
| `https://login.chinacloudapi.cn`<br/>`https://<region>.login.chinacloudapi.cn`<br/>`login.partner.microsoftonline.cn`| Required to fetch and update Azure Resource Manager tokens. |
| `mcr.azk8s.cn` | Required to pull container images for Azure Arc agents.          |
| `https://gbl.his.arc.azure.cn` |  Required to get the regional endpoint for pulling system-assigned Managed Identity certificates. |
| `https://*.his.arc.azure.cn` |  Required to pull system-assigned Managed Identity certificates. |
|`guestnotificationservice.azure.cn`<br/>`*.guestnotificationservice.azure.cn`<br/>`sts.chinacloudapi.cn` | For [Cluster Connect](../cluster-connect.md) and for [Custom Location](../custom-locations.md) based scenarios. |
|`*.servicebus.chinacloudapi.cn` | For [Cluster Connect](../cluster-connect.md) and for [Custom Location](../custom-locations.md) based scenarios. |
|`https://graph.chinacloudapi.cn/` | Required when [Azure RBAC](../azure-rbac.md) is configured. |
|`*.arc.azure.cn` | Required to manage connected clusters in Azure portal.|
|`https://<region>.obo.arc.azure.cn:8084/` | Required when [Cluster Connect](../cluster-connect.md) is configured. |
|`quay.azk8s.cn`<br/>`registryk8s.azk8s.cn`<br/>`k8sgcr.azk8s.cn`<br/>`usgcr.azk8s.cn`<br/>`dockerhub.azk8s.cn/<repo-name>/<image-name>:<version>`|Container registry proxy servers for Azure China VMs.|