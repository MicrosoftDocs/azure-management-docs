---
title: Generate Terraform and Bicep configurations using Azure Copilot
description: Learn about how Azure Copilot can generate Terraform and Bicep configurations for you to use.
ms.date: 11/21/2025
ms.topic: concept-article
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
# Customer intent: As a cloud developer, I want to generate and customize Terraform and Bicep configurations using an AI tool, so that I can efficiently manage and deploy Azure infrastructure.
---

# Generate Terraform and Bicep configurations using Azure Copilot

Azure Copilot can generate Terraform and Bicep configurations that you can use to create and manage your Azure infrastructure.

When you tell Azure Copilot about some Azure infrastructure that you want to manage through Terraform, it provides a configuration using resources from the [AzureRM provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs). In addition to the primary resources, any dependent resources required to accomplish a successful deployment are included in the configuration.

You can also ask Azure Copilot about Azure infrastructure that you'd like to create using Bicep. Copilot provides a template that deploys the necessary resources to create this infrastructure. After generating the initial template, you can ask follow-up questions to further customize the template.

With either Terraform or Bicep, you can ask follow-up questions to further customize the results. When you're ready, copy or download the contents so you can deploy the configuration or template using your deployment method of choice. You can also use the **Select full view** option to see the entire configuration or template in a single view.

The requested Azure infrastructure should be limited to fewer than eight primary resource types. For example, you should see good results when asking for a configuration to manage a resource group that contains Azure Container App, Azure Functions, and Azure Cosmos DB resources. However, requesting configurations to fully address complex architectures may result in inaccurate results and truncated configurations.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Terraform sample prompts

Here are a few examples of the kinds of prompts you can use to generate Terraform configurations. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Create a Terraform config for a Cognitive Services instance with name 'mycognitiveservice' and S0 pricing tier."
- "Show me a Terraform configuration for a linux virtual machine with 8GB ram and an image of 'UbuntuServer 18.04-LTS'. The resource should be placed in the West US location and have a public IP address. Additionally, it should be part of a virtual network with a network security group."
- "Create Terraform configuration for a container app resource with name 'myApp' with quick start image. Add a log analytic space with PerGB2018 sku and set the retention days to 31. Enable single revision mode in the container app and set the CPU and memory limits to 2 and 4GB respectively. Also, set the name of the container app environment to 'awesomeAzureEnv' and set the name of the container to 'myQuickStartContainer'."
- "What is the Terraform code for a Databricks workspace in Azure with name 'myworkspace' and a premium SKU. The workspace should be created in the West US region."
- "Create an OpenAI deployment with gpt-3.5-turbo model using Terraform template. Set the version of the model to 0613."

:::image type="content" source="media/generate-terraform-bicep/generate-terraform.png" alt-text="Screenshot of Azure Copilot generating a Terraform configuration to create a web app." lightbox="media/generate-terraform-bicep/generate-terraform.png" :::

## Bicep sample prompts

Here are a few examples of the kinds of prompts you can use to generate Bicep templates. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "How to create a private endpoint resource using Bicep?"
- "Show me a Bicep template that creates an Azure Storage account with a blob container and a file share."
- "Give me a Bicep template that deploys a Container App Environment with a basic Container App. In addition, it should deploy a Log Analytics Workspace to store logs."
- "Give me a Bicep template for creating a key vault, a managed identity, and a role assignment for the managed identity to access the key vault."
- "How to use Bicep to create Azure OpenAI service?"

 :::image type="content" source="media/generate-terraform-bicep/generate-bicep.png" alt-text="Screenshot of Azure Copilot generating a Bicep template to create a storage account." lightbox="media/generate-terraform-bicep/generate-bicep.png":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Bicep](/azure/azure-resource-manager/bicep/overview?tabs=bicep).
- Learn more about [Terraform on Azure](/azure/developer/terraform/overview).
