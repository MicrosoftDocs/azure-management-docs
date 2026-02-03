---
title: Tutorial - New policy assignment with Azure portal
description: In this tutorial, you use Azure portal to create an Azure Policy assignment to identify non-compliant resources.
ms.topic: how-to
ms.date: 02/03/2026
# Customer intent: "As a cloud administrator, I want to create and assign an Azure Policy to identify non-compliant Azure Arc-enabled servers, so that I can ensure compliance with machine configuration standards and improve the security of my environment."
---

# Tutorial: Create a policy assignment and identify non-compliant resources

You can use [Azure Policy](/azure/governance/policy/overview) to audit the state of your Azure Arc-enabled servers to ensure they comply with machine configuration policies.

This tutorial steps you through the process of creating and assigning a policy that identifies which of your Azure Arc-enabled servers don't have [Microsoft Defender for Servers](/azure/defender-for-cloud/defender-for-servers-overview) enabled.

This tutorial teaches you how to:

> [!div class="checklist"]
> * Create policy assignment and assign a definition to it
> * Identify resources that aren't compliant with the new policy
> * Remove the policy from non-compliant resources

## Prerequisites

* If you don't have an Azure subscription, create a [free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) account before you begin.

* The Azure PowerShell module must be installed on your device if you choose to use this command-line method. To learn more, see [How to install Azure PowerShell
](/powershell/azure/install-azure-powershell).

## Create a policy assignment

# [Portal](#tab/portal)

Use the following procedure to create a policy assignment and assign the policy definition *Azure Defender for servers should be enabled*.

1. At the top of the Azure portal, search for and select **Policy**.

1. In the service menu, expand **Authoring**, then select **Assignments**. An assignment is a policy assigned to take place within a specific scope.

1. Select **Assign policy** from the top of the **Assignments** pane.

   :::image type="content" source="./media/tutorial-assign-policy-portal/assignments-tab.png" alt-text="Screenshot of the Policy Assignments page in the Azure portal." border="true":::

1. On the **Assign Policy** page, under **Scope**, select the ellipsis (**...**) and select either a management group or subscription. Optionally, select a resource group. The scope determines which resources or grouping of resources the policy assignment gets enforced on. Then, choose **Select** at the bottom of the **Scope** pane.

1. Resources can be excluded based on the **Scope**. **Exclusions** start at one level lower than the level of the **Scope**. **Exclusions** are optional, so leave it blank for now.

1. Under **Basics**, select the **Policy definition** ellipsis (**...**) to open the list of available definitions. Azure Policy comes with many built-in policy definitions you can use, such as:

   * Enforce tag and its value
   * Apply tag and its value
   * Inherit a tag from the resource group if missing

   For a partial list of available built-in policies, see [Azure Policy samples](/azure/governance/policy/samples/).

1. Search through the policy definitions list to find the **Azure Defender for servers should be enabled** definition. Choose that policy and select **Add**.

1. The **Assignment name** is automatically populated with the policy name you selected, but you can change it. For this example, leave the policy name as is, and don't change any of the remaining options on the page.

1. For this example, we don't need to change any settings on the other tabs. Select **Review + create** to review your new policy assignment, then select **Create**.

# [Azure PowerShell](#tab/az-powershell)

Set up your variables for the policy you want to assign. The following command retrieves the policy definition ID and assigns your desired policy. Replace the information with your actual values.

```azurepowershell
# Variables
$assignmentName = "myPolicyAssignment"
$policyDefinitionName = "Azure Defender for servers should be enabled"
$scope = "/subscriptions/<your-subscription-id>/resourceGroups/<resource-group-name>"

# Get the policy definition ID
$policy = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq $policyDefinitionName }

# Assign the policy
New-AzPolicyAssignment -Name $assignmentName -PolicyDefinitionId $policy.PolicyDefinitionId -Scope $scope
```

You can also run the following command to display a list of the available policies by the policy name.

```azurepowershell
Get-AzPolicyDefinition | Sort-Object DisplayName
```

# [Azure CLI](#tab/azurecli)

Set up your variables for the policy you want to assign. The following command retrieves the policy definition ID and assigns your desired policy. Replace the information with your actual values.

```azurecli-interactive
# Variables
assignmentName="myPolicyAssignment" \
policyDefinitionName="Azure Defender for servers should be
enabled" \
scope="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"

# Get the policy definition ID (filter by displayName)
policyDefinitionId=$(az policy definition list \
    --query "[?displayName=='$policyDefinitionName'].id | [0]" -o tsv)

# Assign the policy
az policy assignment create \
    --name $assignmentName \
    --policy $policyDefinitionId \
    --scope $scope
```

You can also run the following command to display a list of the available policies by the policy name.

```azurecli-interactive
az policy definition list --query "sort_by([], &displayName)[].{DisplayName:displayName, Name:name}" -o table
```

---

You're now ready to identify noncompliant resources to understand the compliance state of your environment.

## Identify noncompliant resources

1. In the service menu for Azure Policy, select **Compliance**.
1. Locate the **Azure Defender for servers should be enabled** policy assignment you created.

   :::image type="content" source="./media/tutorial-assign-policy-portal/compliance-policy.png" alt-text="Screenshot of Policy Compliance page showing policy compliance for the selected scope." border="true":::

   Any existing resources that aren't compliant with the new assignment display as **Non-compliant** under **Compliance state**.

When a condition is evaluated against your existing resources and found true, those resources are marked as noncompliant with the policy. The following table shows how different policy effects work with the condition evaluation for the resulting compliance state. Although you don't see the evaluation logic in the Azure portal, the compliance state results are shown. The compliance state result is either **Compliant** or **Non-compliant**.

| **Resource state** | **Effect** | **Policy evaluation** | **Compliance state** |
| --- | --- | --- | --- |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | True | Non-compliant |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | False | Compliant |
| New | Audit, AuditIfNotExist\* | True | Non-compliant |
| New | Audit, AuditIfNotExist\* | False | Compliant |

\* The *Append*, *DeployIfNotExist*, and *AuditIfNotExist* effects require the `IF` statement to be `TRUE`. The effects also require the existence condition to be `FALSE` to be noncompliant. When `TRUE`, the `IF` condition triggers evaluation of the existence condition for the related resources.

To learn more, see [Azure Policy compliance states](/azure/governance/policy/concepts/compliance-states).

## Clean up resources

To remove the assignment that you created, follow these steps.

# [Portal](#tab/portal)

1. In the service menu, expand **Authoring**, then select **Assignments**

   *Alternatively*, in the service menu, select **Compliance**.

1. Locate the **Azure Defender for servers should be enabled** policy assignment you created.
1. Right-click the policy assignment, select **Delete assignment**, then select **Yes**.

# [Azure PowerShell](#tab/az-powershell)

1. If you know the assignment name and scope, run the following command:

   ```azurepowershell
   Remove-AzPolicyAssignment -Name "myPolicyAssignment" -Scope
   "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"
   ```

1. If you don’t know the assignment name, run the following command to find it first:

   ```azurepowershell
   Get-AzPolicyAssignment | Where-Object { $_.Properties.DisplayName -eq "Azure Defender for servers should be enabled" }
   ```

   Once you discover the assignment name, remove it with the following command:

   ```azurepowershell
   Remove-AzPolicyAssignment -Name "<myPolicyAssignment>"
   ```

# [Azure CLI](#tab/azurecli)

1. If you know the assignment name and scope, run the following command:

   ```azurecli-interactive
   az policy assignment delete \
       --name "myPolicyAssignment" \
       --scope "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"
   ```

1. If you don’t know the assignment name, run the following command to find it first:

   ```azurecli-interactive
   az policy assignment list \
       --query "[?displayName=='Azure Defender for servers should be enabled']"
   ```

   Once you discover the assignment name, remove it with the following command:

   ```azurecli-interactive
   az policy assignment delete --name "<myPolicyAssignment>"
   ```

---

## Next steps

In this tutorial, you assigned a policy definition to a scope and evaluated its compliance report. The policy definition validates that all the resources in the scope are compliant and identifies which ones aren't. Now you're ready to monitor your Azure Arc-enabled servers machine by enabling [virtual machine (VM) insights](/azure/azure-monitor/vm/vminsights-overview).

To learn how to monitor and view the performance, running process, and dependencies from your machine, continue to the tutorial:

> [!div class="nextstepaction"]
> [Enable VM insights](tutorial-enable-vm-insights.md)
