---
title: Tutorial - New policy assignment with Azure portal
description: In this tutorial, you use Azure portal to create an Azure Policy assignment to identify non-compliant resources.
ms.topic: tutorial
ms.date: 05/05/2025
# Customer intent: "As a cloud administrator, I want to create and assign an Azure Policy to identify non-compliant Azure Arc-enabled servers, so that I can ensure compliance with machine configuration standards and improve the security of my environment."
---

# Tutorial: Create a policy assignment to identify non-compliant resources

You can use [Azure Policy](/azure/governance/policy/overview) to audit the state of your Azure Arc-enabled servers to ensure they comply with machine configuration policies.

This tutorial steps you through the process of creating and assigning a policy that identifies which of your Azure Arc-enabled servers don't have [Microsoft Defender for Servers](/azure/defender-for-cloud/defender-for-servers-overview) enabled.

In this tutorial, you'll learn how to:

> [!div class="checklist"]
> * Create policy assignment and assign a definition to it
> * Identify resources that aren't compliant with the new policy
> * Remove the policy from non-compliant resources

## Prerequisites

If you don't have an Azure subscription, create a [free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) account
before you begin.

## Create a policy assignment

Use the following procedure to create a policy assignment and assign the policy definition *Azure Defender for servers should be enabled*.

1. Launch the Azure Policy service in the Azure portal by searching for and selecting **Policy**.

1. In the service menu, under **Authoring**, select **Assignments**. An assignment is a policy assigned to take place within a specific scope.

1. Select **Assign Policy** from the top of the **Assignments** pane.

    :::image type="content" source="./media/tutorial-assign-policy-portal/assignments-tab.png" alt-text="Screenshot of the Policy Assignments page in the Azure portal." border="true":::

1. On the **Assign Policy** page, select the **Scope** by selecting the ellipsis and selecting either a management group or subscription. Optionally, select a resource group. The scope determines which resources or grouping of resources the policy assignment gets enforced on. Then, choose **Select** at the bottom of the **Scope** pane.

1. Resources can be excluded based on the **Scope**. **Exclusions** start at one level lower than the level of the **Scope**. **Exclusions** are optional, so leave it blank for now.

1. Select the **Policy definition** ellipsis to open the list of available definitions. Azure Policy comes with many built-in policy definitions you can use, such as:

   * Enforce tag and its value
   * Apply tag and its value
   * Inherit a tag from the resource group if missing

   For a partial list of available built-in policies, see [Azure Policy samples](/azure/governance/policy/samples/).

1. Search through the policy definitions list to find the *Azure Defender for servers should be enabled* definition. Choose that policy and select **Add**.

1. The **Assignment name** is automatically populated with the policy name you selected, but you can change it. For this example, leave the policy name as is, and don't change any of the remaining options on the page.

1. For this example, we don't need to change any settings on the other tabs. Select **Review + Create** to review your new policy assignment, then select **Create**.

You're now ready to identify noncompliant resources to understand the compliance state of your environment.

## Identify noncompliant resources

In the service menu for Azure Policy, select **Compliance**. Then locate the **Azure Defender for servers should be enabled** policy assignment you created.

:::image type="content" source="./media/tutorial-assign-policy-portal/compliance-policy.png" alt-text="Screenshot of Policy Compliance page showing policy compliance for the selected scope." border="true":::

Any existing resources that aren't compliant with the new assignment will show **Non-compliant** under **Compliance state**.

When a condition is evaluated against your existing resources and found true, those resources are marked as noncompliant with the policy. The following table shows how different policy effects work with the condition evaluation for the resulting compliance state. Although you don't see the evaluation logic in the Azure portal, the compliance state results are shown. The compliance state result is either **Compliant** or **Non-compliant**.

| **Resource state** | **Effect** | **Policy evaluation** | **Compliance state** |
| --- | --- | --- | --- |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | True | Non-compliant |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | False | Compliant |
| New | Audit, AuditIfNotExist\* | True | Non-compliant |
| New | Audit, AuditIfNotExist\* | False | Compliant |

\* The Append, DeployIfNotExist, and AuditIfNotExist effects require the IF statement to be TRUE.
The effects also require the existence condition to be FALSE to be noncompliant. When TRUE, the IF
condition triggers evaluation of the existence condition for the related resources.

To learn more, see [Azure Policy compliance states](/azure/governance/policy/concepts/compliance-states).

## Clean up resources

To remove the assignment that you created, follow these steps:

1. In the service menu, select **Compliance** (or select  **Assignments** under **Authoring**).
1. Locate the **Azure Defender for servers should be enabled** policy assignment you created.
1. Right-click the policy assignment, then select **Delete assignment**.

## Next steps

In this tutorial, you assigned a policy definition to a scope and evaluated its compliance report. The policy definition validates that all the resources in the scope are compliant and identifies which ones aren't. Now you're ready to monitor your Azure Arc-enabled servers machine by enabling [VM insights](/azure/azure-monitor/vm/vminsights-overview).

To learn how to monitor and view the performance, running process, and dependencies from your machine, continue to the tutorial:

> [!div class="nextstepaction"]
> [Enable VM insights](tutorial-enable-vm-insights.md)
