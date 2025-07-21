---
title: External Validation for Workload Orchestration
description: Learn how to use Event Grid for workload orchestration external validation and how to create a solution template with external validation enabled.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 05/10/2025
ms.custom:
  - build-2025
---

# Enable external validation for workload orchestration

External validation allows you to validate the solution template using an external service, such as an Azure Function or a webhook. The external validation service receives events from the workload orchestration service and can perform custom validation logic. 

This article describes how to set up an Event Grid subscription for workload orchestration and how to create and publish a solution template with external validation enabled.

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
    - Ensure that you have the latest version of the workload orchestration CLI installed. 

## Event Grid subscription for workload orchestration

Azure Event Grid is a fully managed, intelligent event routing service that enables reliable event delivery at massive scale. It facilitates reactive programming by allowing applications to respond to events in near real time, reducing the need for constant polling and enabling loosely coupled architectures. 

You can create an Event Grid subscription to receive notifications from the workload orchestration service. The subscription is used to validate the external workload orchestration solution. 

Event Grid works as a facilitator, connecting different components of a system. It acts as a bridge between event publishers and subscribers, allowing them to communicate without being tightly coupled. For example, on one side you have the workload orchestration service and on the other side, you have the customer’s plugin service. The workload orchestration service is the publisher of events, while the customer’s plugin service is the subscriber that receives and processes those events.

Event Grid supports event-driven models by routing events from various sources—such as Azure services, custom applications, or handlers like Azure Functions, Logic Apps, Event Hubs, or custom webhooks. For more information, see [Azure Event Grid overview](/azure/event-grid/overview).

> [!IMPORTANT]
> Event Grid subscriptions are associated to one topic type (product) at a time. If you already have an Event Grid subscription for another topic type, you need to create a new subscription for workload orchestration.

### Register Event Grid 

Since currently Event Grid first party can't acquire Service-to-Service (S2S) permissions due to service federation isolation (SFI), you need to execute the following steps once per context.

#### [Bash](#tab/bash)

```bash
az provider register --namespace Microsoft.EventGrid
    
providerAppId="4962773b-9cdb-44cf-a8bf-237846a00ab7" # App Id for Event Grid
providerOid=$(az ad sp show --id $providerAppId --query id -o tsv)
    
az role assignment create --assignee "$providerOid" \
    --role "Workload Orchestration IT Admin" \
    --scope "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$instanceName"
```

#### [PowerShell](#tab/powershell)

```powershell
az provider register --namespace Microsoft.EventGrid
    
$providerAppId = "4962773b-9cdb-44cf-a8bf-237846a00ab7" # App Id for Event Grid
$providerOid = $(az ad sp show --id $providerAppId --query id -o tsv)
    
az role assignment create --assignee "$providerOid" `
    --role "Workload Orchestration IT Admin" `
    --scope "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$instanceName"
```
***

### Create an Event Grid subscription using Azure CLI

To create an Event Grid subscription using the Azure CLI, follow these steps:

### [Bash](#tab/bash)

Create an Event Grid subscription for your Azure subscription and resource group. 

```bash
az eventgrid event-subscription create --name <subscription-name> \
    --source-resource-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$instanceName" \
    --endpoint <function-app-endpoint> \
    --endpoint-type azurefunction 
```

#### [PowerShell](#tab/powershell)

Create an Event Grid subscription for your Azure subscription and resource group. 

```powershell
az eventgrid event-subscription create --name <subscription-name> `
    --source-resource-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$instanceName" `
    --endpoint <function-app-endpoint> `
    --endpoint-type azurefunction 
```

***

### Create an Event Grid subscription using Azure portal

To create an Event Grid subscription using the Azure portal, follow these steps:

1. Go to [Azure portal](https://portal.azure.com/).
1. In the search box, type **Event Grid Subscriptions** and select it from the list.
1. Select **+ Event Subscription**.
1. On the **Create Event Subscription** page, follow these steps:
    1. In the **Basics** tab, in the **Topic Types** field, select the type of event source on which you want to subscribe. For workload orchestration, select **Workload Orchestration (Preview)**.
    
        :::image type="content" source="./media/event-grid-1.png" alt-text="Screenshot of the Azure portal showing how to create an Event Grid subscription for workload orchestration." lightbox="./media/event-grid-1.png":::

    1. Select the **Azure subscription** and **resource group** that contains the workload orchestration context.
    1. Select the context resource in the **Resource** field.
    1. On **Event types**, select the event types that you want to receive notifications for. 
    1. Select an **endpoint type** and the endpoint that receives the events. For example, you can select Azure Function as endpoint type and SolutionValidator is the name of the function that receives events from Event Grid.
    1. Click on **Create** to create the Event Grid subscription.

For more information, see [Event Grid subscription through portal](/azure/event-grid/subscribe-through-portal) and [Event Grid delivery](/azure/event-grid/delivery-and-retry).

### Provide access to workload orchestration

If you use a function app as the endpoint for the Event Grid subscription, you need to assign the **Workload Orchestration Solution External Validator** role to the function app managed identity. This role allows the function app to validate the solution template and send events back to the workload orchestration service.

#### [Bash](#tab/bash)

1. Run the following command to get the managed identity object ID of the function app. Replace `<functionAppName>` with the name of your function app.

    ```bash
    az functionapp identity show \
        --name "<functionAppName>" \
        --resource-group "$rg"
    ```

1. Copy the `principalId` from the output, which is the managed identity object ID of the function app.

    ```bash
    functionAppMSIObjectId="<principalId>"
    ```

1. Assign the "Workload Orchestration Solution External Validator" role to the function app managed identity. For more information, see [Assign Azure roles using the Azure CLI](/azure/role-based-access-control/role-assignments-cli).

    ```bash
    az role assignment create \
        --assignee $functionAppMSIObjectId \
        --role "Workload Orchestration Solution External Validator" \
        --scope "/subscriptions/$subId/resourceGroups/$rg"
    ```

#### [PowerShell](#tab/powershell)

1. Run the following command to get the managed identity object ID of the function app. Replace `<functionAppName>` with the name of your function app.

    ```powershell
    az functionapp identity show `
      --name "<functionAppName>" `
      --resource-group "$rg"
    ```

1. Copy the `principalId` from the output, which is the managed identity object ID of the function app.

    ```powershell
    $functionAppMSIObjectId = "<principalId>"
    ```

1. Assign the "Workload Orchestration Solution External Validator" role to the function app managed identity. For more information, see [Assign Azure roles using the Azure CLI](/azure/role-based-access-control/role-assignments-cli).

    ```powershell
    az role assignment create `
      --assignee $functionAppMSIObjectId `
      --role "Workload Orchestration Solution External Validator" `
      --scope "/subscriptions/$subId/resourceGroups/$rg" 
    ```
***

## Create a solution template with external validation enabled

When you create a solution template, you can enable external validation by setting the `--enable-external-validation` parameter to `true`. This allows you to validate the solution template using an external service, such as an Azure Function or a webhook. The external validation service receives events from the workload orchestration service and can perform custom validation logic.

### [Bash](#tab/bash)

For example, the following command creates a solution template with external validation enabled.

```bash
az workload-orchestration solution-template create \
    --solution-template-name $appName1 \
    --resource-group $rg \
    --location $l \
    --capabilities $appCapList1 \
    --description $desc \
    --config-template-file $appConfig \
    --specification "@specs.json" \
    --version $appVersion \
    --enable-ext-validation "true"
```

### [PowerShell](#tab/powershell)

For example, the following command creates a solution template with external validation enabled.

```powershell
az workload-orchestration solution-template create `
  --solution-template-name $appName1 `
  --resource-group $rg `
  --location $l `
  --capabilities $appCapList1 `
  --description $desc `
  --config-template-file $appConfig `
  --specification "@specs.json" `
  --version $appVersion `
  --enable-ext-validation "true"
```
***

> [!NOTE]
> When a solution template version is created with the external validation flag set (true or false), the flag is stored at the solution template level. As a result, all solution versions—both new and existing—inherit the same external validation setting. Thus, it isn't possible to have multiple versions under the same solution template with different external validation configurations.

For more information about solution templates and publishing a solution, see [Quickstart: Create a basic solution without common configurations](quickstart-solution-without-common-configuration.md#define-the-variables-for-solution-templating).

## Publish and validate the solution 

### [Bash](#tab/bash)

1. When you publish the solution version, the publish command triggers the external validation process. The workload orchestration service sends an event to the Event Grid subscription, which invokes the external validation service. The external validation service can then perform custom validation logic and send a response back to the workload orchestration service.

    ```bash
    az workload-orchestration target publish \
      --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName1/versions/1.0.0 \
      --resource-group "$rg" \
      --target-name "$childName"
    ```

1. Set `solutionVersionId` and `externalValidationId` as variables which you get as part of the publish response.

    ```bash
    solutionVersionId="<solutionVersionId>"
    externalValidationId="<externalValidationId>"
    ```

### [PowerShell](#tab/powershell)

1. When you publish the solution version, he publish command triggers the external validation process. The workload orchestration service sends an event to the Event Grid subscription, which invokes the external validation service. The external validation service can then perform custom validation logic and send a response back to the workload orchestration service.

    ```powershell
    az workload-orchestration target publish `
      --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName1/versions/1.0.0 `
      --resource-group $rg `
      --target-name $childName
    ```
1. Set ``solutionVersionId` and `externalValidationId` as variables which you get a part of publish response.

    ```powershell	
    $solutionVersionId = "<solutionVersionId>"
    $externalValidationId = "<externalValidationId>"
    ```
***

Event Grid uses the event subscription to determine the final delivery endpoint and applies necessary filtering, such as matching the subject name, to ensure only relevant events are forwarded.

## Monitor the validation status with workload orchestration portal

After publishing, the solution should instantly move to *Publish In Progress* state in the [workload orchestration portal](monitor.md#monitor-solutions-with-external-validation-enabled), meaning that the data has been successfully pushed to Event Grid for external validation.

- If the solution is in **Ready to deploy** state, the validation completed successfully.

- If the solution is in **Publish failed** state, the validation failed due to some errors. In the [Configure tab](configure.md#configure-a-solution-with-external-validation-enabled) of the workload orchestration portal, go to the *Published Solutions* tab and click on the alert for the solution to view the error details.


## Check the status of solution version via CLI

You can check the state of the solution version using the CLI. The state of the solution version is stored in the `properties.state` field of the solution version object. 

### Status is ReadyToDeploy

If the state changes from `PendingExternalValidation` to `ReadyToDeploy`, it means that the external validation was successful and the solution is ready to be deployed. You can proceed with the installation of the solution.

#### [Bash](#tab/bash)

Change `solutionVersion` to the new version you created in the previous step.

```bash
subId="<subscription-id>"
solutionVersion="<new-solution-version>"
solutionName="<solution-template-name>"

az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$solutionName$/versions/$solutionVersion
```

#### [PowerShell](#tab/powershell)

Change `solutionVersion` to the new version you created in the previous step.

```powershell
$subId = "<subscription-id>"
$solutionVersion = "<new-solution-version>"
$solutionName = "<solution-template-name>"

az workload-orchestration target install --resource-group $rg --target-name $childName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$solutionName/versions/$solutionVersion
```

***

### Status is ExternalValidationFailed

If the state changes from `PendingExternalValidation` to `ExternalValidationFailed`, it means that the external validation failed due to some wrong configurations, and the solution can't be deployed. To solve this, you need to create a new version/revision of the solution template with valid configurations.

### Status is PendingExternalValidation

If the state remains in `PendingExternalValidation` state, it's possible that the status doesn't proceed further due to some error in Function App. To solve this, you can manually update the status of the solution version to either `Valid` or `Invalid` using the CLI command below.

#### [Bash](#tab/bash)

1. In GET response, check the state in properties.state

    ```bash
    az rest --method GET --url "$solutionVersionId?api-version=2025-01-01-preview"
    ```

1. To set solution version configurations as **valid**:

    1. Update the status of the solution version to `Valid` using the following command:

        ```bash
        az workload-orchestration target update-external-validation-status \
            --resource-group $rg \
            --target-name $childName \
            --external-validation-id $externalValidationId \
            --solution-version-id $solutionVersionId \
            --validation-status "Valid"
        ```

    1. In the command response, the solution version object is displayed where the state is changed to `ReadyToDeploy`.
    1. Proceed further with installation.

1. To set solution version configurations as **invalid**:

    1. Update the status of the solution version to `Invalid` using the following command:
    
        ```bash
        az workload-orchestration target update-external-validation-status \
         --resource-group $rg \
         --target-name $childName \
         --external-validation-id $externalValidationId \
         --solution-version-id $solutionVersionId \
         --validation-status "Invalid" \
         --error-details "@error.json"
        ```

    1. In the command response, solution version object is displayed where the state is changed to `ExternalValidationFailed`.
    1. Errors mentioned in *error.json* file are stored in the `properties.errorDetails` field in the response solution version object. The errors are visible on [workload orchestration portal](monitor.md).
    1. As this is the terminal state, you can't proceed with installation as there are some invalid configurations in solution version. You need to create new version/revision with valid configurations to proceed for install.


#### [PowerShell](#tab/powershell)

1. In GET response, check the state in properties.state

    ```powershell
    az rest --method GET --url "$solutionVersionId`?api-version=2025-01-01-preview"
    ```

1. To set solution version configurations as **valid**:

    1. Update the status of the solution version to `Valid` using the following command:

        ```powershell
        az workload-orchestration target update-external-validation-status `
            --resource-group $rg `
            --target-name $childName `
            --external-validation-id $externalValidationId `
            --solution-version-id $solutionVersionId `
            --validation-status "Valid"
        ```

    1. In the command response, solution version object is displayed where the state is changed to `ReadToDeploy`.
    1. Proceed further with install.

1. To set solution version configurations as **invalid**:

    1. Update the status of the solution version to `Invalid` using the following command:
    
        ```powershell
        az workload-orchestration target update-external-validation-status `
         --resource-group $rg `
         --target-name $childName `
         --external-validation-id $externalValidationId `
         --solution-version-id $solutionVersionId `
         --validation-status "Invalid" `
         --error-details "@error.json"
        ```

    1. In the command response, solution version object is displayed where the state is changed to `ExternalValidationFailed`.
    1. Errors mentioned in *error.json* file are stored in the `properties.errorDetails` field in the response solution version object. The errors are visible on [workload orchestration portal](monitor.md).
    1. As this is the terminal state, you can't proceed with installation as there are some invalid configurations in solution version. You need to create new version/revision with valid configurations to proceed for install.

***



