---
title: Create an Azure portal dashboard with PowerShell
description: Learn how to create a dashboard in the Azure portal using Azure PowerShell.
ms.topic: quickstart
ms.custom: devx-track-azurepowershell, mode-api
ms.date: 03/04/2025
# Customer intent: As an Azure developer, I want to create a customized dashboard in the Azure portal using PowerShell, so that I can monitor and manage my cloud resources efficiently.
---

# Quickstart: Create an Azure portal dashboard with PowerShell

A [dashboard](azure-portal-dashboards.md) in the Azure portal is a focused and organized view of your cloud resources. This article focuses on the process of using the [Az.Portal PowerShell module](/powershell/module/az.portal) to create a dashboard. The example dashboard shows the performance of a virtual machine (VM) that you create, along with some static information and links.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).

- If you choose to use PowerShell locally, this article requires that you install the Az PowerShell module and connect to your Azure account using the [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) cmdlet. For more information about installing the Az PowerShell module, see [Install Azure PowerShell](/powershell/azure/install-azure-powershell).

[!INCLUDE [cloud-shell-try-it](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Choose a specific Azure subscription

If you have multiple Azure subscriptions, choose the appropriate subscription in which the resources should be billed. Select a specific subscription using the
[Set-AzContext](/powershell/module/az.accounts/set-azcontext) cmdlet.

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## Define variables

The example dashboard uses several pieces of information repeatedly. Create variables to store this information.

```azurepowershell-interactive
# Name of resource group used throughout this article
$resourceGroupName = 'myResourceGroup'

# Azure region
$location = 'centralus'

# Dashboard Title
$dashboardTitle = 'Simple VM Dashboard'

# Dashboard Name
$dashboardName = $dashboardTitle -replace '\s'

# Your Azure Subscription ID
$subscriptionID = (Get-AzContext).Subscription.Id

# Name of test VM
$vmName = 'myVM1'
```

## Create a resource group

Create an [Azure resource group](/azure/azure-resource-manager/management/overview) using the [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)
cmdlet. A resource group is a logical container in which Azure resources are deployed and managed as a group.

The following example creates a resource group based on the name in the `$resourceGroupName`
variable in the region specified in the `$location` variable.

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

## Create a virtual machine

> [!IMPORTANT]
> The steps outlined in this quickstart are solely for education purposes and aren't intended for deployments to a production environment. For information about best practices for production virtual machines, see the [security best practices for VMs and operating systems](/azure/security/fundamentals/iaas).

The example dashboard requires a virtual machine. If you have a VM already, you can update your template to use that VM. Otherwise, you can create an example VM by following these steps.

Store login credentials for the VM in a variable. The password must be complex. This is a new username and password (not the account you use to sign in to Azure). For more information, see [username requirements](/azure/virtual-machines/windows/faq#what-are-the-username-requirements-when-creating-a-vm-)
and [password requirements](/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

```azurepowershell-interactive
$Cred = Get-Credential
```

Create the VM.

```azurepowershell-interactive
$AzVmParams = @{
  ResourceGroupName = $resourceGroupName
  Name = $vmName
  Location = $location
  Credential = $Cred
}
New-AzVm @AzVmParams
```

The VM deployment now starts and typically takes a few minutes to complete. After deployment completes, move on to the next section.

## Download the dashboard template

Since Azure dashboards are resources, they can be represented as JSON. The following code downloads a JSON representation of a sample dashboard. For more information, see [The structure of Azure Dashboards](./azure-portal-dashboards-structure.md).

```azurepowershell-interactive
$myPortalDashboardTemplateUrl = 'https://raw.githubusercontent.com/Azure-Samples/azure-docs-powershell-samples/refs/heads/main/azure-portal/portal-dashboard-template-testvm.json'

$myPortalDashboardTemplatePath = "$HOME\portal-dashboard-template-testvm.json"

Invoke-WebRequest -Uri $myPortalDashboardTemplateUrl -OutFile $myPortalDashboardTemplatePath -UseBasicParsing
```

## Customize the template

Customize the downloaded template by running the following code.

```azurepowershell
$Content = Get-Content -Path $myPortalDashboardTemplatePath -Raw
$Content = $Content -replace '<subscriptionID>', $subscriptionID
$Content = $Content -replace '<rgName>', $resourceGroupName
$Content = $Content -replace '<vmName>', $vmName
$Content = $Content -replace '<dashboardTitle>', $dashboardTitle
$Content = $Content -replace '<location>', $location
$Content | Out-File -FilePath $myPortalDashboardTemplatePath -Force
```

For more information about the dashboard template structure, see [Microsoft portal dashboards template reference](/azure/templates/microsoft.portal/dashboards).

## Deploy the dashboard template

You can use the `New-AzPortalDashboard` cmdlet that's part of the Az.Portal module to deploy the template directly from PowerShell.

```azurepowershell
$DashboardParams = @{
  DashboardPath = $myPortalDashboardTemplatePath
  ResourceGroupName = $resourceGroupName
  DashboardName = $dashboardName
}
New-AzPortalDashboard @DashboardParams
```

## Review the deployed resources

Check that the dashboard was created successfully.

```azurepowershell
Get-AzPortalDashboard -Name $dashboardName -ResourceGroupName $resourceGroupName
```

[!INCLUDE [azure-portal-review-deployed-resources](./includes/azure-portal-review-deployed-resources.md)]

## Clean up resources

To remove the VM and associated dashboard, delete the resource group that contains them.

> [!CAUTION]
> Deleting the resource group will delete all of the resources contained within it. If the resource group contains additional resources aside from your virtual machine and dashboard, those resources will also be deleted.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
Remove-Item -Path "$HOME\portal-dashboard-template-testvm.json"
```

## Next steps

For more information about the cmdlets contained in the Az.Portal PowerShell module, see:

> [!div class="nextstepaction"]
> [Microsoft Azure PowerShell: Portal Dashboard cmdlets](/powershell/module/Az.Portal/#portal)
