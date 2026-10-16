---
title: Increase Azure Storage account quotas
description: Learn how to request an increase in the quota limit for Azure Storage accounts within a subscription from 250 to 500 for a given region. Quota increases apply to both standard and premium account types.
ms.date: 03/13/2024
ms.topic: how-to
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to request an increase in the Azure Storage account quota for our subscription, so that we can accommodate our growing data storage needs effectively.
---

# Increase Azure Storage account quotas

This article shows how to view your storage account quotas and request quota increases by using the [Azure portal](https://portal.azure.com), [Azure PowerShell](/powershell/module/az.storage/get-azstorageusage), or [Azure CLI](/cli/azure/storage/account#az-storage-account-show-usage). You can also view quota usage and limits by using the [REST API](/rest/api/storagerp/usages/list-by-location).

In the [Azure portal](https://portal.azure.com), **My quotas** is a centralized location where you can view your quota usage and request quota increases. To quickly request an increase, select **Quotas** on the Home page in the Azure portal.

:::image type="content" source="media/storage-account-quota-requests/quotas-icon.png" alt-text="Screenshot of the Quotas icon in the Azure portal.":::

If you don't see **Quotas** in the Azure portal, type *quotas* in the search box, then select **Quotas**. The **Quotas** icon then appears on your Home page the next time you visit.

You can request an increase from 250 to up to 1000 storage accounts per region for your subscription. This quota increase applies to storage accounts with standard endpoints.

## View current quotas for a region

To view your current storage account quotas for a subscription in a given region, use the Azure portal, PowerShell, or Azure CLI.

# [Portal](#tab/azure-portal)

1. From the [Azure portal](https://portal.azure.com), select **Quotas** and then select **Storage**.

1. Select your subscription from the drop-down.

1. Use the **Region** filter to specify the regions you're interested in. You can then see your storage account quotas for each of those regions.

    :::image type="content" source="media/storage-account-quota-requests/view-quotas-region-portal.png" alt-text="Screenshow showing how to filter on regions to show quotas for specific regions" lightbox="media/storage-account-quota-requests/view-quotas-region-portal.png":::

# [PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Get-AzStorageUsage -Location <region>
```

For more information, see [Get-AzStorageUsage](/powershell/module/az.storage/get-azstorageusage) or [Get-AzQuota](/powershell/module/az.quota/get-azquota).


# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az storage account show-usage --location <region>
```

For more information, see [az storage account show-usage](/cli/azure/storage/account#az-storage-account-show-usage) or [az quota list](/cli/azure/quota#az-quota-list).

---

## Request storage account quota increases

To request quota increases, you must have an Azure account with the Contributor role (or another role that includes Contributor access).

# [Portal](#tab/azure-portal)

Follow these steps to request a storage account quota increase from Azure Home.

1. From the [Azure portal](https://portal.azure.com), select **Quotas** and then select **Storage**.

1. Select the subscription for which you want to increase your storage account quota.

1. Locate the region where you want to increase your storage account quota, then select the pencil icon in the **Request adjustment** column.

1. In the **New Quota Request** pane, enter a number up to 1000.

    :::image type="content" source="media/storage-account-quota-requests/request-quota-increase-portal.png" alt-text="Screenshot showing how to increase your storage account quota":::

1. Select **Submit**. It might take a few minutes to process your request.

# [PowerShell](#tab/azure-powershell)

To increase the storage account quota to 500:

```azurepowershell-interactive
Update-AzQuota -ResourceName 'StorageAccounts' -Name 'StorageAccounts' -Scope '/subscriptions/<subscriptionId>/providers/Microsoft.Storage/locations/<region>' -Limit (New-AzQuotaLimitObject -Value 500 -LimitType 'Independent')
```

For more information, see [Update-AzQuota](/powershell/module/az.quota/update-azquota).

# [Azure CLI](#tab/azure-cli)

To increase the storage account quota to 500:

```azurecli-interactive
az quota update --resource-name StorageAccounts --scope /subscriptions/<subscriptionId>/providers/Microsoft.Storage/locations/<region> --limit-object value=500
```

For more information, see [az quota update](/cli/azure/quota#az-quota-update).

---

## See also

- [Scalability and performance targets for standard storage accounts](/azure/storage/common/scalability-targets-standard-account)
- [Scalability targets for premium block blob storage accounts](/azure/storage/blobs/scalability-targets-premium-block-blobs)
- [Scalability and performance targets for premium page blob storage accounts](/azure/storage/blobs/scalability-targets-premium-page-blobs)
- [Azure subscription and service limits, quotas, and constraints](/azure/azure-resource-manager/management/azure-subscription-service-limits)
