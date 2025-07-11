---
title: Deploy Azure Policy to delegated subscriptions at scale
description: Azure Lighthouse lets you deploy a policy definition and policy assignment across multiple tenants.
ms.date: 07/16/2024
ms.topic: how-to 
ms.custom: devx-track-azurepowershell
# Customer intent: "As a service provider managing multiple customer tenants, I want to deploy Azure Policy across these tenants so that I can ensure compliance with organizational policies and streamline management tasks."
---

# Deploy Azure Policy to delegated subscriptions at scale

As a service provider, you may have onboarded multiple customer tenants to [Azure Lighthouse](../overview.md). Azure Lighthouse allows service providers to perform operations at scale across several tenants at once, making management tasks more efficient.

This topic explains how to use [Azure Policy](/azure/governance/policy/) to deploy a policy definition and policy assignment across multiple tenants using PowerShell commands. In this example, the policy definition ensures that storage accounts are secured by allowing only HTTPS traffic. You can use the same general process for any policy that you want to deploy.

> [!TIP]
> Though we refer to service providers and customers in this topic, [enterprises managing multiple tenants](../concepts/enterprise.md) can use the same processes.

## Use Azure Resource Graph to query across customer tenants

You can use [Azure Resource Graph](/azure/governance/resource-graph/overview) to query across all subscriptions in customer tenants that you manage. In this example, we'll identify any storage accounts in these subscriptions that don't currently require HTTPS traffic.  

```powershell
$MspTenant = "insert your managing tenantId here"

$subs = Get-AzSubscription

$ManagedSubscriptions = Search-AzGraph -Query "ResourceContainers | where type == 'microsoft.resources/subscriptions' | where tenantId != '$($mspTenant)' | project name, subscriptionId, tenantId" -subscription $subs.subscriptionId

Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Storage/storageAccounts' | project name, location, subscriptionId, tenantId, properties.supportsHttpsTrafficOnly" -subscription $ManagedSubscriptions.subscriptionId | convertto-json
```

## Deploy a policy across multiple customer tenants

The following example shows how to use an [Azure Resource Manager template](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/policy-enforce-https-storage/enforceHttpsStorage.json) to deploy a policy definition and policy assignment across delegated subscriptions in multiple customer tenants. This policy definition requires all storage accounts to use HTTPS traffic. It prevents the creation of any new storage accounts that don't comply. Any existing storage accounts without the setting are marked as noncompliant.

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
> While you can deploy policies across multiple tenants, currently you can't [view compliance details](/azure/governance/policy/how-to/determine-non-compliance#compliance-details) for non-compliant resources in these tenants.

## Validate the policy deployment

After you've deployed the Azure Resource Manager template, confirm that the policy definition was successfully applied by attempting to create a storage account with **EnableHttpsTrafficOnly** set to **false** in one of your delegated subscriptions. Because of the policy assignment, you should be unable to create this storage account.  

```powershell
New-AzStorageAccount -ResourceGroupName (New-AzResourceGroup -name policy-test -Location eastus -Force).ResourceGroupName `
                     -Name (get-random) `
                     -Location eastus `
                     -EnableHttpsTrafficOnly $false `
                     -SkuName Standard_LRS `
                     -Verbose                  
```

## Clean up resources

When you're finished, you can remove the policy definition and assignment created by the deployment.

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
