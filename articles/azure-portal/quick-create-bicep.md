---
title: Create an Azure portal dashboard by using a Bicep file
description: Learn how to create an Azure portal dashboard by using a Bicep file.
ms.topic: quickstart
ms.date: 03/04/2025
ms.custom:
  - subject-bicepqs
  - devx-track-bicep
  - sfi-ropc-nochange
# Customer intent: As an Azure developer, I want to deploy a dashboard in the Azure portal using a Bicep file, so that I can visualize the performance of my virtual machine and manage resources efficiently.
---

# Quickstart: Create a dashboard in the Azure portal by using a Bicep file

A [dashboard](azure-portal-dashboards.md) in the Azure portal is a focused and organized view of your cloud resources. This quickstart shows how to deploy a Bicep file to create a dashboard. The example dashboard shows the performance of a virtual machine (VM), along with some static information and links.

[!INCLUDE [About Bicep](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-bicep-introduction.md)]

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- [Azure PowerShell](/powershell/azure/install-azure-powershell) or [Azure CLI](/cli/azure/install-azure-cli).

## Review the Bicep file

The Bicep file used in this quickstart is from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/azure-portal-dashboard/). This Bicep file is too long to show here. To view the Bicep file, see [main.bicep](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.portal/azure-portal-dashboard/main.bicep).

The Bicep file defines one Azure resource, a [Microsoft.Portal dashboards resource](/azure/templates/microsoft.portal/dashboards?pivots=deployment-language-bicep) that displays data about the VM that you'll create as part of the deployment.

The example dashboard created by deploying this Bicep file requires an existing virtual machine. Before deploying the Bicep file, the script deploys an ARM template called [prereq.azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.portal/azure-portal-dashboard/prereqs/prereq.azuredeploy.json) that creates a virtual machine to be used in the dashboard.

The virtual machine name is hard-coded as **SimpleWinVM** in the ARM template, to match what's used in the `main.bicep` file that creates the dashboard. You'll need to create your own administration username and password for this example VM. This is a new username and password (not the account you use to sign in to Azure). The password must be complex. For more information, see [username requirements](/azure/virtual-machines/windows/faq#what-are-the-username-requirements-when-creating-a-vm-)
and [password requirements](/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

> [!IMPORTANT]
> The steps outlined in this quickstart are solely for education purposes and aren't intended for deployments to a production environment. For information about best practices for production virtual machines, see the [security best practices for VMs and operating systems](/azure/security/fundamentals/iaas).

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file using either Azure CLI or Azure PowerShell, using the script shown here. Replace the following values in the script:

    - &lt;admin-user-name>: specify an administrator username.
    - &lt;admin-password>: specify an administrator password.
    - &lt;dns-label-prefix>: specify a DNS prefix.

    # [CLI](#tab/CLI)

    ```azurecli
    $resourceGroupName = 'SimpleWinVmResourceGroup'
    $location = 'eastus'
    $adminUserName = '<admin-user-name>'
    $adminPassword = '<admin-password>'
    $dnsLabelPrefix = '<dns-label-prefix>'
    $virtualMachineName = 'SimpleWinVM'
    $vmTemplateUri = 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.portal/azure-portal-dashboard/prereqs/prereq.azuredeploy.json'

    az group create --name $resourceGroupName --location $location
    az deployment group create --resource-group $resourceGroupName --template-uri $vmTemplateUri --parameters adminUsername=$adminUserName adminPassword=$adminPassword dnsLabelPrefix=$dnsLabelPrefix
    az deployment group create --resource-group $resourceGroupName --template-file main.bicep --parameters virtualMachineName=$virtualMachineName virtualMachineResourceGroup=$resourceGroupName
    ```

    # [PowerShell](#tab/PowerShell)

    ```azurepowershell
    $resourceGroupName = 'SimpleWinVmResourceGroup'
    $location = 'eastus'
    $adminUserName = '<admin-user-name>'
    $adminPassword = '<admin-password>'
    $dnsLabelPrefix = '<dns-label-prefix>'
    $virtualMachineName = 'SimpleWinVM'
    $vmTemplateUri = 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.portal/azure-portal-dashboard/prereqs/prereq.azuredeploy.json'

    $encrypted = ConvertTo-SecureString -string $adminPassword -AsPlainText

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $vmTemplateUri -adminUsername $adminUserName -adminPassword $encrypted -dnsLabelPrefix $dnsLabelPrefix
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile ./main.bicep -virtualMachineName $virtualMachineName -virtualMachineResourceGroup $resourceGroupName
    ```

    ---

After the deployment finishes, you should see a message indicating the deployment succeeded.

## Review deployed resources

[!INCLUDE [azure-portal-review-deployed-resources](./includes/azure-portal-review-deployed-resources.md)]

## Clean up resources

To remove the example VM and associated dashboard, delete the resource group that contains them.

1. In the Azure portal, search for **SimpleWinVmResourceGroup**, then select it in the search results.

1. On the **SimpleWinVmResourceGroup** page, select **Delete resource group**, enter the resource group name to confirm, then select **Delete**.

> [!CAUTION]
> Deleting a resource group will delete all of the resources contained within it. If the resource group contains additional resources aside from your virtual machine and dashboard, those resources will also be deleted.

## Next steps

For more information about dashboards in the Azure portal, see:

> [!div class="nextstepaction"]
> [Create and share dashboards in the Azure portal](azure-portal-dashboards.md)
