---
title: How to deploy and configure Azure Monitor Agent using Azure Policy
description: Learn how to deploy and configure Azure Monitor Agent using Azure Policy.
ms.date: 05/01/2025
ms.topic: how-to
# Customer intent: "As a cloud administrator, I want to deploy and configure the Azure Monitor Agent on Arc-enabled servers using Azure Policy, so that I can ensure consistent monitoring and data collection practices across my hybrid environment."
---

# Deploy and configure Azure Monitor Agent using Azure Policy

This article covers how to deploy and configure the Azure Monitor Agent (AMA) to Arc-enabled servers through Azure Policy using a custom policy definition. Using Azure Policy ensures that Azure Monitor is running on your selected Arc-enabled servers and that the AMA is automatically installed on newly added Arc resources.

Deploying the Azure Monitor Agent through a custom Policy definition involves two main steps:

- Selecting an existing or creating a new Data Collection Rule (DCR)
- Creating and deploying the policy definition

In order for Azure Monitor to work on a machine, it needs to be associated with a Data Collection Rule (DCR). You include the resource ID of the DCR when you create your policy definition.

## Select a Data Collection Rule

Data Collection Rules define the data collection process in Azure Monitor. They specify what data should be collected and where that data should be sent. You'll need to select or create a DCR to be associated with your Policy definition.

1. From your browser, go to the [Azure portal](https://portal.azure.com).

1. Navigate to the **Monitor | Overview** page. Under **Settings**, select **Data Collection Rules** to show the list of existing DCRs.

1. Select the DCR that you want to use.

1. Select **Overview**, then select **JSON View** to view the JSON code for the DCR:

    :::image type="content" source="media/deploy-ama-policy/dcr-overview.png" alt-text="Screenshot of the Overview window for a data collection rule highlighting the JSON view button." lightbox="media/deploy-ama-policy/dcr-overview.png":::

1. Locate the **Resource ID** field at the top of the **Resource JSON** pane, then select the button to copy the resource ID. You'll need to use this resource ID when creating your policy definition.

## Create and deploy the Policy definition

In order for Azure Policy to check if the Azure Monitor Agent is installed on your Arc-enabled servers, you need to create a custom policy definition that does the following:

- Evaluates if new VMs have the agent installed and are associated with the DCR.

- Enforces a remediation task to install the Azure Monitor Agent and create the association with the DCR on any VMs that aren't compliant with the policy.

1. Select one of the following policy definition templates, depending on the operating system of the machine:
    - [Configure Windows machines](https://portal.azure.com/#view/Microsoft_Azure_Policy/InitiativeDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F9575b8b7-78ab-4281-b53b-d3c1ace2260b/scopes/undefined)
    - [Configure Linux machines](https://portal.azure.com/#view/Microsoft_Azure_Policy/InitiativeDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F118f04da-0375-44d1-84e3-0fd9e1849403/scopes/undefined)

    These templates are used to create a policy to configure machines to run Azure Monitor Agent and associate those machines to a DCR.

1. Select **Assign initiative** to begin creating the policy definition. Enter the applicable information for each tab. For more information about these options, see [Create a policy assignment ](/azure/governance/policy/assign-policy-portal#assign-a-policy-initiative).

1. On the **Parameters** tab, paste the **Data Collection Rule Resource ID** that you copied during the previous procedure:

    :::image type="content" source="media/deploy-ama-policy/resource-id-field.png" alt-text="Screenshot of the Parameters tab highlighting the Data Collection Rule Resource ID field.":::

1. Select **Review + create** to complete policy creation and deploy it to the applicable machines. Once Azure Monitor Agent is deployed, your Azure Arc-enabled servers can apply its services and use it for log collection.

## Additional resources

- [Azure Monitor overview](/azure/azure-monitor/overview)
- [Tutorial: Monitor a hybrid machine with VM insights](learn/tutorial-enable-vm-insights.md)
