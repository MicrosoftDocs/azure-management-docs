---
title: Deploy Azure Policy to delegated subscriptions at scale
description: Azure Lighthouse lets you deploy a policy definition and policy assignment across multiple tenants.
ms.date: 01/20/2026
ms.topic: how-to 
ms.custom: devx-track-azurepowershell
# Customer intent: "As a service provider managing multiple customer tenants, I want to deploy Azure Policy across these tenants so that I can ensure compliance with organizational policies and streamline management tasks."
---

# Deploy Azure Policy to delegated subscriptions at scale

As a service provider, you can onboard multiple customer tenants to [Azure Lighthouse](../overview.md). Azure Lighthouse lets you perform operations at scale across several tenants at once, making management tasks more efficient.

This article explains how to use [Azure Policy](/azure/governance/policy/overview) to deploy a policy definition and policy assignment across multiple tenants by using PowerShell commands. In this example, the policy definition ensures that storage accounts are secured by allowing only HTTPS traffic. You can use the same general process for any policy that you want to deploy.

> [!TIP]
> Though this article refers to service providers and customers, [enterprises managing multiple tenants](../concepts/enterprise.md) can use the same processes.

## Use Azure Resource Graph to query across customer tenants

[Azure Resource Graph](/azure/governance/resource-graph/overview) lets you query across all subscriptions in customer tenants that you manage.

This example identifies any storage accounts in managed customer subscriptions that don't currently require HTTPS traffic.  

```powershell
$MspTenant = "insert your managing tenantId here"

$subs = Get-AzSubscription

$ManagedSubscriptions = Search-AzGraph -Query "ResourceContainers | where type == 'microsoft.resources/subscriptions' | where tenantId != '$($mspTenant)' | project name, subscriptionId, tenantId" -subscription $subs.subscriptionId

Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Storage/storageAccounts' | project name, location, subscriptionId, tenantId, properties.supportsHttpsTrafficOnly" -subscription $ManagedSubscriptions.subscriptionId | convertto-json
```

## Deploy a policy across multiple customer tenants

After identifying the storage accounts that don't comply with a requirement, you can deploy a policy to enforce that setting.

The following example shows how to use an [Azure Resource Manager template](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/policy-enforce-https-storage/enforceHttpsStorage.json) to deploy a policy definition and policy assignment across all delegated subscriptions in multiple customer tenants. This policy definition requires all storage accounts to use HTTPS traffic, preventing creation of new storage accounts that don't comply. Any existing storage accounts without that setting are marked as noncompliant.

```powershell
Write-Output "In total, there are $($ManagedSubscriptions.Count) delegated customer subscriptions to be managed"

foreach ($ManagedSub in $ManagedSubscriptions)
{
    Select-AzSubscription -SubscriptionId $ManagedSub.subscriptionId

    New-AzSubscriptionDeployment -Name mgmt `
                     -Location eastus `
                     -TemplateUri "https://raw.githubusercontent.com/Azure/Azure-Lighthouse-samples/master/templates/policy-enforce-https-storage/enforceHttpsStorage.json" `
                     -AsJob
}
```

> [!NOTE]
> While you can deploy policies across multiple tenants, you can't currently [view compliance details](/azure/governance/policy/how-to/determine-non-compliance#compliance-details) for noncompliant resources in these tenants through Azure Lighthouse.

## Validate the policy deployment

After deploying the Azure Resource Manager template, confirm that the policy definition was successfully applied by attempting to create a storage account with **EnableHttpsTrafficOnly** set to **false** in one of your delegated subscriptions. The policy assignment should prevent you from creating this storage account.    

```powershell
New-AzStorageAccount -ResourceGroupName (New-AzResourceGroup -name policy-test -Location eastus -Force).ResourceGroupName `
                     -Name (get-random) `
                     -Location eastus `
                     -EnableHttpsTrafficOnly $false `
                     -SkuName Standard_LRS `
                     -Verbose                  
```

## Clean up resources

When you're finished, remove the policy definition and assignment created by the deployment by running the following PowerShell script.

```powershell
foreach ($ManagedSub in $ManagedSubscriptions)
{
    select-azsubscription -subscriptionId $ManagedSub.subscriptionId

    Remove-AzSubscriptionDeployment -Name mgmt -AsJob

    $Assignment = Get-AzPolicyAssignment | where-object {$_.Name -like "enforce-https-storage-assignment"}

    if ([string]::IsNullOrEmpty($Assignment))
    {
        Write-Output "Nothing to clean up - we're done"
    }
    else
    {

    Remove-AzPolicyAssignment -Name 'enforce-https-storage-assignment' -Scope "/subscriptions/$($ManagedSub.subscriptionId)" -Verbose

    Write-Output "Deployment has been deleted - we're done"
    }
}
```

## Next steps

- Learn about [Azure Policy](/azure/governance/policy/).
- Learn about [cross-tenant management experiences](../concepts/cross-tenant-management-experience.md).
- Learn how to [deploy a policy that can be remediated](deploy-policy-remediation.md) within a delegated subscription.
