---
title: Monitor delegation changes in your managing tenant
description: Learn how to monitor all Azure Lighthouse delegation activity to your managing tenant. 
ms.date: 01/21/2026
ms.topic: how-to 
ms.devlang: azurecli
ms.custom:
  - devx-track-azurepowershell
  - devx-track-azurecli
  - sfi-ga-nochange
# Customer intent: "As a service provider, I want to monitor delegation changes to my managing tenant, so that I can maintain oversight of customer subscriptions and resource groups and respond to any modifications effectively."
---

# Monitor delegation changes in your managing tenant

As a service provider, you might want to keep track of customer subscriptions or resource groups that are delegated to your tenant through [Azure Lighthouse](../overview.md), or when previously delegated resources are removed.

In the managing tenant, the [Azure activity log](/azure/azure-monitor/platform/activity-log) tracks delegation activity at the tenant level. This logged activity includes any added or removed delegations from customer tenants.

This article explains the permissions needed to monitor delegation activity to your tenant across all of your customers. It also includes a sample script that shows one method for querying and reporting on this data.

> [!IMPORTANT]
> Perform all these steps in your managing tenant, rather than in any customer tenants.
>
> Though this article refers to service providers and customers, [enterprises managing multiple tenants](../concepts/enterprise.md) can use the same processes.

## Enable access to tenant-level data

To access tenant-level Activity Log data, an account must have the Monitoring Reader Azure built-in role at root scope (/). This assignment must be performed by a Global Administrator with additional elevated access.

### Elevate access for a Global Administrator account

To assign a role at root scope (/), you must have the Global Administrator role with elevated access. Enable this elevated access only when you need to make the role assignment, and remove it when you're done.

For detailed instructions on adding and removing elevation, see [Elevate access to manage all Azure subscriptions and management groups](/azure/role-based-access-control/elevate-access-global-admin).

After you elevate your access, your account has the User Access Administrator role in Azure at root scope. This role assignment allows you to view all resources and assign access in any subscription or management group in the directory. It also allows you to make role assignments at root scope.

### Assign the Monitoring Reader role at root scope

After you elevate your access, you can assign the appropriate permissions to an account so it can query tenant-level activity log data. Assign the [Monitoring Reader](/azure/role-based-access-control/built-in-roles#monitoring-reader) Azure built-in role at the root scope of your managing tenant to this account.

> [!IMPORTANT]
> Granting a role assignment at root scope means that the same permissions apply to every resource in the tenant. Because this access is very broad, we recommend [assigning this role to a service principal account and using that account to query data](#use-a-service-principal-account-to-query-the-activity-log).
>
> You can also assign the Monitoring Reader role at root scope to individual users or to user groups so they can [view delegation information directly in the Azure portal](#view-delegation-changes-in-the-azure-portal). If you choose this option, limit this broad access to the fewest number of users possible.

Use one of the following methods to make the root scope assignment.

#### PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

New-AzRoleAssignment -SignInName <yourLoginName> -Scope "/" -RoleDefinitionName "Monitoring Reader"  -ObjectId <objectId> 
```

#### Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az role assignment create --assignee 00000000-0000-0000-0000-000000000000 --role "Monitoring Reader" --scope "/"
```

### Remove elevated access for the Global Administrator account

After you assign the Monitoring Reader role at root scope to the desired account, be sure to [remove the elevated access](/azure/role-based-access-control/elevate-access-global-admin#remove-elevated-access-for-users) for the Global Administrator account, since this high level of access is no longer needed.

## View delegation changes in the Azure portal

Users who are assigned the Monitoring Reader role at the root scope can view delegation changes directly in the Azure portal.

1. Go to the **My customers** page, and select **Activity log** from the left-hand navigation menu.
1. Make sure **Directory Activity** is selected in the filter near the top of the screen.
1. Select the timespan for which you want to view delegation changes.

:::image type="content" source="../media/delegation-activity-portal.jpg" alt-text="Screenshot of delegation changes in the Azure portal.":::

## Use a service principal account to query the activity log

Because the Monitoring Reader role at root scope is such a broad level of access, assign the role to a service principal account and use that account to query data via a script.

> [!IMPORTANT]
> Currently, tenants with a large amount of delegation activity might run into errors when querying this data.

When using a service principal account to query the activity log, follow these best practices:

- [Create a new service principal account](/entra/identity-platform/howto-create-service-principal-portal) to use only for this function, rather than assigning this role to an existing service principal used for other automation.
- Ensure that this service principal doesn't have access to any delegated customer resources.
- [Use a certificate to authenticate](/entra/identity-platform/howto-create-service-principal-portal#set-up-authentication) and [store it securely in Azure Key Vault](/azure/key-vault/general/security-features).
- Limit the users who have access to act on behalf of the service principal.

After you create a service principal account that has Monitoring Reader access at the root scope of your managing tenant, use it to query and report on delegation activity.

[This Azure PowerShell script](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/monitor-delegation-changes) can query the past day of activity and report any added or removed delegations (or attempts that weren't successful). It queries the [Tenant Activity Log](/rest/api/monitor/TenantActivityLogs/List) data, then constructs the following values to report on delegations that are added or removed:

- **DelegatedResourceId**: The ID of the delegated subscription or resource group
- **CustomerTenantId**: The customer tenant ID
- **CustomerSubscriptionId**: The subscription ID that was delegated or that contains the resource group that was delegated
- **CustomerDelegationStatus**: The status change for the delegated resource (succeeded or failed)
- **EventTimeStamp**: The date and time at which the delegation change was logged

When querying this data, keep in mind:

- If multiple resource groups are delegated in a single deployment, separate entries are returned for each resource group.
- Changes made to a previous delegation (such as updating the permission structure) are logged as an added delegation.
- As noted earlier, an account must have the Monitoring Reader Azure built-in role at root scope (/) to access this tenant-level data.
- You can use this data in your own workflows and reporting. For example, you can use the [Logs Ingestion API](/azure/azure-monitor/logs/logs-ingestion-api-overview) to log data to Azure Monitor from a REST API client, then use [action groups](/azure/azure-monitor/alerts/action-groups) to create notifications or alerts.

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# Azure Lighthouse: Query Tenant Activity Log for registered/unregistered delegations for the last 1 day

$GetDate = (Get-Date).AddDays((-1))

$dateFormatForQuery = $GetDate.ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")

# Getting Azure context for the API call
$currentContext = Get-AzContext

# Fetching new token
$azureRmProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = [Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient]::new($azureRmProfile)
$token = $profileClient.AcquireAccessToken($currentContext.Tenant.Id)

$listOperations = @{
    Uri     = "https://management.azure.com/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&`$filter=eventTimestamp ge '$($dateFormatForQuery)'"
    Headers = @{
        Authorization  = "Bearer $($token.AccessToken)"
        'Content-Type' = 'application/json'
    }
    Method  = 'GET'
}
$list = Invoke-RestMethod @listOperations

# First link can be empty - and point to a next link (or potentially multiple pages)
# While you get more data - continue fetching and add result
while($list.nextLink){
    $list2 = Invoke-RestMethod $list.nextLink -Headers $listOperations.Headers -Method Get
    $data+=$list2.value;
    $list.nextLink = $list2.nextlink;
}

$showOperations = $data;

if ($showOperations.operationName.value -eq "Microsoft.Resources/tenants/register/action") {
    $registerOutputs = $showOperations | Where-Object -FilterScript { $_.eventName.value -eq "EndRequest" -and $_.resourceType.value -and $_.operationName.value -eq "Microsoft.Resources/tenants/register/action" }
    foreach ($registerOutput in $registerOutputs) {
        $eventDescription = $registerOutput.description | ConvertFrom-Json;
    $registerOutputdata = [pscustomobject]@{
        Event                    = "An Azure customer has registered delegated resources to your Azure tenant";
        DelegatedResourceId      = $eventDescription.delegationResourceId; 
        CustomerTenantId         = $eventDescription.subscriptionTenantId;
        CustomerSubscriptionId   = $eventDescription.subscriptionId;
        CustomerDelegationStatus = $registerOutput.status.value;
        EventTimeStamp           = $registerOutput.eventTimestamp;
        }
        $registerOutputdata | Format-List
    }
}
if ($showOperations.operationName.value -eq "Microsoft.Resources/tenants/unregister/action") {
    $unregisterOutputs = $showOperations | Where-Object -FilterScript { $_.eventName.value -eq "EndRequest" -and $_.resourceType.value -and $_.operationName.value -eq "Microsoft.Resources/tenants/unregister/action" }
    foreach ($unregisterOutput in $unregisterOutputs) {
        $eventDescription = $unregisterOutput.description | ConvertFrom-Json;
    $unregisterOutputdata = [pscustomobject]@{
        Event                    = "An Azure customer has unregistered delegated resources from your Azure tenant";
        DelegatedResourceId      = $eventDescription.delegationResourceId;
        CustomerTenantId         = $eventDescription.subscriptionTenantId;
        CustomerSubscriptionId   = $eventDescription.subscriptionId;
        CustomerDelegationStatus = $unregisterOutput.status.value;
        EventTimeStamp           = $unregisterOutput.eventTimestamp;
        }
        $unregisterOutputdata | Format-List
    }
}
else {
    Write-Output "No new delegation events for tenant: $($currentContext.Tenant.TenantId)"
}
```

## Next steps

- Learn how to [onboard customers to Azure Lighthouse](onboard-customer.md).
- Learn about [Azure Monitor](/azure/azure-monitor/) and the [Azure activity log](/azure/azure-monitor/essentials/activity-log).
- Review the [Activity Logs by Domain](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain) sample workbook to learn how to display Azure activity logs across subscriptions with an option to filter them by domain name.
