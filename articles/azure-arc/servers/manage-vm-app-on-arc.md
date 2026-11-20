---
title: Manage VM Applications on Arc-enabled servers (preview)
description: Learn how to manage VM Applications on Azure Arc-enabled servers.
ms.date: 10/26/2026
ms.topic: how-to
ai-usage: ai-generated
---

# Manage VM applications on Azure Arc-enabled servers (preview)

Manage your VM Applications throughout their lifecycle, from published versions in Azure Compute Gallery to applications deployed to your Arc-enabled servers. This article explains how to manage gallery resources, how to view deployment status, review logs, update application versions, and remove deployed applications.

## Manage VM applications published in Azure Compute Gallery

Azure Compute Gallery stores your VM Application definitions and versions. Each version references an application package and the commands used to install, update, and remove it. Managing these published resources is separate from managing the applications deployed to your Arc-enabled servers.

Use the following guidance to manage your gallery resources:

- **View applications and versions.** Review application definitions, available versions, and publishing details. See [View published VM Applications](/azure/virtual-machines/vm-applications-manage#view-published-vm-applications).
- **Query applications across galleries.** Use Azure Resource Graph to inventory published applications and versions across your subscriptions. See [View published VM applications using Azure Resource Graph](/azure/virtual-machines/vm-applications-manage#view-published-vm-applications-using-azure-resource-graph).
- **Update published resources.** Change supported properties of an application or version. Not all properties can be updated. See [Update published VM Application](/azure/virtual-machines/vm-applications-manage#update-published-vm-application).
- **Delete applications and versions.** Delete all versions before deleting an application definition. Deleting a version removes its gallery replicas but doesn't delete the source application blob in your storage account. See [Delete published VM Application from Azure Compute Gallery](/azure/virtual-machines/vm-applications-manage#delete-published-vm-application-from-azure-compute-gallery).

## Manage VM applications deployed to Azure Arc-enabled servers

Refer to the following sections to manage VM Applications deployed to your Arc-enabled servers, including viewing installed applications, monitoring their status, updating versions, and removing applications.

### View deployed VM applications and their state

Retrieve details for a specific application or list all deployed applications on an Arc-enabled server.

#### Get details of a single application

Use one of the following methods to view a specific VM Application deployed to your Arc-enabled server.

# [Azure portal](#tab/view-portal)

1. In the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**.
1. Select **Machines**, and then select your Arc-enabled server.
1. In the resource menu, select **VM applications**.
1. Find your application in the list to review its version, install order, gallery, and provisioning state.

# [Azure CLI](#tab/view-azure-cli)

```azurecli
az connectedmachine application get-instance-view \
  --name "{application-resource-name}" \
  --machine-name "{arc-machine-name}" \
  --resource-group "{resource-group}"
```

# [REST](#tab/view-rest)

```rest
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.HybridCompute/machines/{arc-machine-name}/applications/{application-resource-name}/instanceView?api-version=2026-09-01
```

# [Azure Resource Graph](#tab/view-arg)


```kusto
resources
| where type =~ "microsoft.hybridcompute/machines/applications"
| where resourceGroup =~ "{resource-group}"
| extend machineName = tostring(split(id, "/")[8])
| where machineName =~ "{arc-machine-name}" and name =~ "{application-resource-name}"
| project subscriptionId, resourceGroup, location, machineName,
          applicationName = name,
          provisioningState = tostring(properties.provisioningState),
          packageReferenceId = tostring(properties.packageReferenceId),
          id
```

> [!NOTE]
> Azure Resource Graph doesn't provide an instance view. To retrieve an instance view, use Azure CLI or REST.

---

#### List all applications on your Arc-enabled server

Use one of the following methods to view all VM applications deployed to an Arc-enabled server.

# [Azure portal](#tab/list-portal)

1. In the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**.
1. Select **Machines**, and then select your Arc-enabled server.
1. In the resource menu, select **VM applications**.
1. Review the list of deployed applications.

# [Azure CLI](#tab/list-azure-cli)

```azurecli
az connectedmachine application list \
  --machine-name "{arc-machine-name}" \
  --resource-group "{resource-group}"
```

To retrieve instance view for a listed application, use `az connectedmachine application get-instance-view` with its name, as shown in [Get details of a single application](#get-details-of-a-single-application).

# [REST](#tab/list-rest)

```rest
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.HybridCompute/machines/{arc-machine-name}/applications?api-version=2026-09-01
```

To retrieve the instance view for a listed application, use its `/instanceView` endpoint, as shown in [Get details of a single application](#get-details-of-a-single-application).

# [Azure Resource Graph](#tab/list-arg)

```kusto
resources
| where type =~ "microsoft.hybridcompute/machines/applications"
| where resourceGroup =~ "{resource-group}"
| extend machineName = tostring(split(id, "/")[8])
| where machineName =~ "{arc-machine-name}"
| project subscriptionId, resourceGroup, location, machineName,
          applicationName = name,
          provisioningState = tostring(properties.provisioningState),
          packageReferenceId = tostring(properties.packageReferenceId),
          id
```

> [!NOTE]
> Azure Resource Graph doesn't provide instance view. To retrieve instance view, use Azure CLI or REST.

---


### Update a deployed VM application

Update a deployed VM application on your Arc-enabled server to a newer version by using one of the following methods.

Before you begin, [create a new application version in Azure Compute Gallery](/azure/virtual-machines/vm-applications-how-to#create-the-vm-application) and replicate it to the region where your Arc-enabled server is connected.

# [Azure portal](#tab/update-portal)

1. In the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**.
1. Select **Machines**, and then select your Arc-enabled server.
1. In the resource menu, select **VM applications**.
1. Confirm that the **Update available** column shows an update for your application.
1. Select the application's checkbox, and then on the toolbar, select **Manual update**.

# [Azure CLI](#tab/update-azure-cli)

```azurecli
az connectedmachine application update \
  --name "{application-resource-name}" \
  --machine-name "{arc-machine-name}" \
  --resource-group "{resource-group}" \
  --package-reference-id "{package-reference-id}"
```

# [REST](#tab/update-rest)


```rest
PATCH 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.HybridCompute/machines/{arc-machine-name}/applications/{application-resource-name}?api-version=2026-09-01
Content-Type: application/json

{
  "properties": {
    "packageReferenceId": "/subscriptions/{subscription-id}/resourceGroups/{gallery-resource-group}/providers/Microsoft.Compute/galleries/{gallery-name}/applications/{application-name}/versions/{version}",
    "order": {order},
    "treatFailureAsDeploymentFailure": false
  }
}
```

---

### Remove a deployed VM application

Remove a deployed VM Application from your Arc-enabled server by using one of the following methods. 

# [Azure portal](#tab/remove-portal)

1. In the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**.
1. Select **Machines**, and then select your Arc-enabled server.
1. In the resource menu, select **VM applications**.
1. Select the checkbox for the application you want to remove.
1. On the toolbar, select **Delete**, and then confirm the deletion.

# [Azure CLI](#tab/remove-azure-cli)

```azurecli
az connectedmachine application delete \
  --name "{application-resource-name}" \
  --machine-name "{arc-machine-name}" \
  --resource-group "{resource-group}"
```

# [REST](#tab/remove-rest)

```rest
DELETE 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.HybridCompute/machines/{arc-machine-name}/applications/{application-resource-name}?api-version=2026-09-01
```

---

## Next steps

- [Monitor VM Applications on Arc-enabled servers](monitor-vm-app-on-arc.md)