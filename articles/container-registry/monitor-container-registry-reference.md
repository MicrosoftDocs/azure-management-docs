---
title: Monitoring Data Reference for Azure Container Registry
description: Access essential reference material for monitoring Azure Container Registry, including metrics, logs, and activity logs for comprehensive insights.
ms.date: 10/09/2026
ms.custom: horz-monitor, subject-monitoring
ms.topic: reference
author: jeburke
ms.author: jeburke
ms.service: azure-container-registry
ai-usage: ai-assisted
# Customer intent: As a cloud operations manager, I want to access monitoring metrics and logs for Azure Container Registry, so that I can gain insights into performance and operational activities to ensure efficient management and troubleshooting of container resources.
---

# Azure Container Registry monitoring data reference

[!INCLUDE [horz-monitor-ref-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-intro.md)]

See [Monitor Azure Container Registry](monitor-container-registry.md) for details on the data you can collect for Azure Container Registry and how to use it.

[!INCLUDE [horz-monitor-ref-metrics-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-intro.md)]

### Supported metrics for Microsoft.ContainerRegistry/registries

The following table lists the metrics available for the Microsoft.ContainerRegistry/registries resource type.

[!INCLUDE [horz-monitor-ref-metrics-tableheader](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-tableheader.md)]

[!INCLUDE [Microsoft.ContainerRegistry/registries](~/reusable-content/ce-skilling/azure/includes/azure-monitor/reference/metrics/microsoft-containerregistry-registries-metrics-include.md)]

> [!NOTE]
> Because of layer sharing, registry **Storage used** might be less than the sum of storage for individual repositories. When you [delete](container-registry-delete.md) a repository or tag, you recover only the storage used by manifest files and the unique layers referenced.

[!INCLUDE [horz-monitor-ref-metrics-dimensions-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-dimensions-intro.md)]

[!INCLUDE [horz-monitor-ref-metrics-dimensions](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-dimensions.md)]

| Dimension name | Description |
|---|---|
| `ApiName` | The registry operation represented by the metric. Available values depend on the metric. |
| `ResponseType` | The outcome of a transaction, classified by its HTTP status code. |
| `Region` | The Azure region that handled the transaction or from which data was transferred. |

`Region` values depend on the registry's home region and [geo-replicas](container-registry-geo-replication.md).

> [!NOTE]
> The `StorageUsed` metric uses the legacy `Geolocation` dimension to represent the Azure region where content is
> stored. Use `Geolocation` when filtering or splitting `StorageUsed`. Newer metrics use the `Region` dimension.

### API name values

The `Transactions` metric supports the following `ApiName` values:

- `HeadManifest`: Checks whether a manifest exists and retrieves its response headers.
- `GetManifest`: Retrieves a manifest.
- `GetManifestProperties`: Retrieves properties for a specific manifest.
- `GetListManifests`: Lists the manifests in a repository.
- `GetTag`: Retrieves information about a tag.
- `GetListTags`: Lists the tags in a repository.
- `HeadBlob`: Checks whether a blob or layer exists and retrieves its response headers.
- `GetBlobRedirection`: Redirects a blob or layer download request to its content location.
- `GetBlob`: Retrieves blob or layer content.
- `GetListRepositories`: Lists repositories in the registry.
- `GetListReferrers`: Lists artifacts that refer to a specified manifest digest.
- `GetRepositoryProperties`: Retrieves repository properties.
- `GetToken`: Requests an ACR access token for the requested scope.
- `PostToken`: Exchanges an ACR refresh token for an ACR access token.
- `PostExchange`: Exchanges a Microsoft Entra access token for an ACR refresh token.

For the corresponding API definitions, see
[Registry data plane REST operation groups](/rest/api/registry-dataplane/operation-groups).

The `DataTransfer` metric supports the following `ApiName` values:

- `GetBlob`: Data transferred when a client downloads a blob or layer.
- `GetManifest`: Data transferred when a client retrieves a manifest.
- `GeoReplication`: Data transferred during service-initiated geo-replication, including initial synchronization and
  replication after a push.
- `ImportSource`: Data transferred when another Azure container registry uses this registry as the source of an import
  operation.

### Response type values

The `Transactions` metric supports the following `ResponseType` values:

- `Success`: The operation succeeded with HTTP status code 200, 201, or 202.
- `Redirect`: The operation returned HTTP status code 307. For example, a blob download can redirect the client to the
  blob's content location.
- `ClientErrorUnauthorized`: Authentication or authorization failed with HTTP status code 401 or 403.
- `ClientErrorNotFound`: The requested resource wasn't found and returned HTTP status code 404.
- `Throttled`: The request was rate limited and returned HTTP status code 429.
- `ClientErrorOther`: The operation returned a 4xx client error that doesn't have a dedicated response type.
- `ServerErrorOther`: The operation returned a 5xx server error.

[!INCLUDE [horz-monitor-ref-resource-logs](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-resource-logs.md)]

### Supported resource logs for Microsoft.ContainerRegistry/registries

[!INCLUDE [Microsoft.ContainerRegistry/registries](~/reusable-content/ce-skilling/azure/includes/azure-monitor/reference/logs/microsoft-containerregistry-registries-logs-include.md)]

[!INCLUDE [horz-monitor-ref-logs-tables](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-logs-tables.md)]

For a reference of all Azure Monitor Logs and Log Analytics tables, see the [Azure Monitor Log Table Reference](/azure/azure-monitor/reference/tables-index).

### Container Registry Microsoft.ContainerRegistry/registries

- [AzureActivity](/azure/azure-monitor/reference/tables/azureactivity#columns). Entries from the Azure Activity log that provide insight into any subscription-level or management group level events that occurred in Azure.
- [AzureMetrics](/azure/azure-monitor/reference/tables/azuremetrics#columns). Metric data emitted by Azure services that measure their health and performance.
- [ContainerRegistryLoginEvents](/azure/azure-monitor/reference/tables/containerregistryloginevents#columns). Registry authentication events and status, including the incoming identity and IP address.
- [ContainerRegistryRepositoryEvents](/azure/azure-monitor/reference/tables/containerregistryrepositoryevents#columns). Operations on images and other artifacts in registry repositories. The following operations are logged: push, pull, untag, delete (including repository delete), purge tag, and purge manifest.

  Purge events are logged only if a registry [retention policy](container-registry-retention-policy.md) is configured.

[!INCLUDE [horz-monitor-ref-activity-log](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-activity-log.md)]

- [Microsoft.ContainerRegistry resource provider operations](/azure/role-based-access-control/resource-provider-operations#microsoftcontainerregistry)

The following table lists operations related to Azure Container Registry that can be created in the Activity log. This list isn't exhaustive.

| Operation | Description |
|:---|:---|
| Create or Update Container Registry | Create a container registry or update a registry property |
| Delete Container Registry | Delete a container registry |
| List Container Registry Login Credentials | Show credentials for registry's admin account |
| Import Image | Import an image or other artifact to a registry |
| Create Role Assignment | Assign an identity a Role-based access control (RBAC) role to access a resource  |

## Related content

- See [Monitor Azure Container Registry](monitor-container-registry.md) for a description of monitoring Container Registry.
- See [Monitor Azure resources with Azure Monitor](/azure/azure-monitor/essentials/monitor-azure-resource) for details on monitoring Azure resources.
