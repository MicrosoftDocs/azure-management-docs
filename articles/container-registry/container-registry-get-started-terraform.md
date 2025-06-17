---
title: 'Quickstart: Create an Azure container registry using Terraform'
description: In this quickstart, you create a unique resource group and an Azure container registry in a specified location using Terraform.
ms.topic: quickstart
ms.date: 11/19/2024
ms.custom: devx-track-terraform
ms.service: azure-container-registry
author: chasedmicrosoft
ms.author: doveychase
content_well_notification: 
  - AI-contribution
# Customer intent: As a Terraform user, I want to create an Azure container registry and resource group, so that I can manage and deploy private Docker container images in my Azure environment.
---

# Quickstart: Create an Azure container registry using Terraform

In this quickstart, you create an Azure container registry and a resource group using Terraform. Azure Container Registry is a managed Docker registry service used for storing private Docker container images. It's typically used with Azure Kubernetes Service (AKS), Azure App Service, and other Azure services to pull down container images. The registry is stored within a resource group, which is a logical container for resources deployed on Azure. These resources are created with unique names by combining a prefix with a random string, ensuring they are unique within your Azure subscription.

[!INCLUDE [About Terraform](~/azure-dev-docs-pr/articles/terraform/includes/abstract.md)]

> [!div class="checklist"]
> * Specify the required version of Terraform and the required providers.
> * Define the Azure provider with no additional features.
> * Define a variable for the location of the resource group and with a default value of "eastus".
> * Define a variable for the prefix of the resource group name and with a default value of "rg".
> * Generate a random pet name for the resource group.
> * Create an Azure resource group with the generated name at a specified location.
> * Generate a random string of five lowercase letters to be used as part of the container registry name.
> * Create a container registry with the generated string as part of its name and in the same location and resource group as above.
> * Output the names of the created resource group and container registry.
> * Output the login server of the created container registry.

## Prerequisites

- Create an Azure account with an active subscription. You can [create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- [Install and configure Terraform](/azure/developer/terraform/quickstart-configure).

## Implement the Terraform code

> [!NOTE]
> The sample code for this article is located in the [Azure Terraform GitHub repo](https://github.com/Azure/terraform/tree/master/quickstart/101-azure-container-registry). You can view the log file containing the [test results from current and previous versions of Terraform](https://github.com/Azure/terraform/tree/master/quickstart/101-azure-container-registry/TestRecord.md).
> 
> See more [articles and sample code showing how to use Terraform to manage Azure resources](/azure/terraform).

1. Create a directory in which to test and run the sample Terraform code, and make it the current directory.

1. Create a file named `main.tf`, and insert the following code:
    :::code language="Terraform" source="~/terraform_samples/quickstart/101-azure-container-registry/main.tf":::

1. Create a file named `outputs.tf`, and insert the following code:
    :::code language="Terraform" source="~/terraform_samples/quickstart/101-azure-container-registry/outputs.tf":::

1. Create a file named `providers.tf`, and insert the following code:
    :::code language="Terraform" source="~/terraform_samples/quickstart/101-azure-container-registry/providers.tf":::

1. Create a file named `variables.tf`, and insert the following code:
    :::code language="Terraform" source="~/terraform_samples/quickstart/101-azure-container-registry/variables.tf":::

## Initialize Terraform

[!INCLUDE [terraform-init.md](~/azure-dev-docs-pr/articles/terraform/includes/terraform-init.md)]

## Create a Terraform execution plan

[!INCLUDE [terraform-plan.md](~/azure-dev-docs-pr/articles/terraform/includes/terraform-plan.md)]

## Apply a Terraform execution plan

[!INCLUDE [terraform-apply-plan.md](~/azure-dev-docs-pr/articles/terraform/includes/terraform-apply-plan.md)]

## Verify the results

### [Azure CLI](#tab/azure-cli)

Run [az acr show](/cli/azure/acr#az-acr-show) to view the container registry.

```azurecli
az acr show --name <registry_name> --resource-group <resource_group_name>
```

Replace `<registry_name>` with the name of your container registry and `<resource_group_name>` with the name of your resource group.

---

## Clean up resources

[!INCLUDE [terraform-plan-destroy.md](~/azure-dev-docs-pr/articles/terraform/includes/terraform-plan-destroy.md)]

## Troubleshoot Terraform on Azure

[Troubleshoot common problems when using Terraform on Azure](/azure/developer/terraform/troubleshoot).

## Next steps

> [!div class="nextstepaction"]
> [See more articles about Azure Container Registry](/search/?terms=Azure%20container%20registry%20and%20terraform).
