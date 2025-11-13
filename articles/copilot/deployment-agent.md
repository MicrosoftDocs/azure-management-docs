---
title: Deployment agent capabilities in Agents (preview) in Azure Copilot
description: Agents (preview) in Azure Copilot lets you complete deployment tasks using intelligent agent capabilities.
author: JnHs
ms.author: jenhayes
ms.date: 11/18/2025
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use agents to help with deployment tasks, so that I can be more successful performing tasks in  my Azure tenant."
---

# Deployment capabilities in Agents (preview) in Azure Copilot

[Agents (preview) in Azure Copilot](agents-preview.md) intelligently surfaces the right agent to help with your tasks. The deployment capabilities of Agents (preview) in Azure Copilot serve as a virtual cloud solution architect, guiding you through the entire infrastructure planning and deployment process with simplicity and precision.

When you ask for help deploying workloads, Azure Copilot helps you translate high-level goals into actionable deployment plans by applying [Azure Well-Architected Framework](/azure/well-architected/) best practices. You can get help with tasks like creating workload plans, generating and reviewing Terraform configurations, and streamlining the automation of Infrastructure-as-Code (IaC) workflows by integrating with GitHub.

The agent capabilities support multi-turn conversations to clarify requirements, offering recommendations for optimal resource configurations and providing step-by-step guidance for deploying production-ready environments—whether you’re setting up analytics pipelines, web applications, or complex multi-tier architectures. These capabilities let you reduce manual effort, minimize errors, and accelerate time-to-value for your cloud deployments.

After generating a deployment plan, Azure Copilot can generate Terraform configurations that you can review, edit, and deploy. You can [open the generated files in Visual Studio Code for the Web](#open-in-vs-code) or have Azure Copilot [create a pull request to add the files to your GitHub repository](#github-pull-request-integration).

> [!IMPORTANT]
> The functionality described in this article is only available for tenants that have access to [Agents (preview) in Azure Copilot](agents-preview.md).

## Supported resource types

Currently, Agents (preview) in Azure Copilot supports deployment tasks for all Azure resource types. You can get assistance with many types of deployments, including:

- Compute services, such as Virtual Machines and containerized workloads.
- Networking components, such as Virtual Networks, Subnets, and Network Security Groups.
- Storage solutions, such as Blob Storage, with advanced resiliency options.
- Identity and access management scenarios.
- Monitoring and diagnostics solutions.
- Orchestration of multi-tier architectures for complex workloads.

## Deployment sample prompts

Here are a few examples of the kinds of prompts you can use to get help with deployment tasks. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. The more details you provide about the workload you want to deploy, the better Azure Copilot can assist you. When using these kinds of prompts, be sure to enable agent mode by selecting the icon in the chat window.

:::image type="content" source="media/agent-preview/azure-copilot-agent-mode-active.png" alt-text="Screenshot showing agent mode (preview) enabled in Azure Copilot.":::

- "Host a sentiment-analysis LLM with Azure Functions for serverless API endpoints, connect to an Azure SQL Database for logging user interactions, and set up alerting for failed requests."
- "Deploy a Python Flask web app on Azure App Service with a PostgreSQL Flexible Server backend, secure secrets in Azure Key Vault, and enable monitoring with Application Insights."
- "Launch a multilingual chatbot service using Azure OpenAI Service, integrate logging with Azure Monitor, and use Azure Key Vault for API credential management."
- "Set up a multitenant SaaS application on AKS using Kubernetes namespaces for isolation, integrate Microsoft Entra for authentication, and centralize logs in Azure Log Analytics."
- "Deploy a microservices workload on AKS where API Gateway routes traffic, integrate with Azure Key Vault for secrets, and roll out canary deployments for new service versions."

## Example workflow

Here’s an example workflow for using the deployment capabilities in Agents (preview) in Azure Copilot.

1. To get deployment help, start a conversation in Azure Copilot with agent mode enabled. Describe the workload you want to deploy using simple, natural language, such as "**I need a scalable web app with a SQL database.**"
1. Azure Copilot reviews your requirements and builds a detailed infrastructure plan tailored to your needs. If more information is needed, Azure Copilot asks questions about your scenario. The workload plan is a comprehensive, step-by-step blueprint that includes analysis of pros, cons, and trade-offs associated with each architectural decision. All recommendations are grounded in the [Azure Well-Architected Framework](/azure/well-architected/), ensuring that recommendations align with industry standards and Azure best practices.
1. After you approve the plan, Azure Copilot creates Terraform configurations with the necessary components to deploy the resources outlined in the plan. Comprehensive guidance on deployment strategies, including guidance for CI/CD pipeline configuration, is also provided. Select the maximize icon to view the scripts in the artifact pane.

   :::image type="content" source="media/deployment-agent/deployment-terraform-generated.png" alt-text="Screenshot of Azure Copilot providing Terraform configurations for a deployment.":::

1. Review the generated Terraform configurations in Azure Copilot's artifact pane. If desired, you can make changes to the configurations from this pane. Be sure to review the configurations carefully to make sure that they meet your requirements.
1. After validating the configurations, choose a deployment method. You can open the files in [VS Code for the Web](#open-in-vs-code), [create a GitHub pull request](#github-pull-request-integration), or use the Azure portal. You can also download the files for local deployment or further customization.

   :::image type="content" source="media/deployment-agent/deployment-artifact-pane.png" alt-text="Screenshot of Azure Copilot's artifact pane showing Terraform configurations and deployment options.":::

1. Complete the deployment, then use Azure's monitoring tools to track the performance, cost, and health of your resources.

## GitHub pull request integration

When Azure Copilot generates Terraform configurations, you can choose to automatically create a pull request to add the files to your GitHub repository. This option simplifies the process of integrating the generated files into your existing CI/CD workflows.

To use this feature, select **Create pull request** after reviewing your generated Terraform configurations in the artifact pane. After signing in, select an existing repository and branch, or create new ones. When you select **Create pull request**, the generated files are added to a new pull request.

:::image type="content" source="media/deployment-agent/deployment-github-pull-request.png" alt-text="Screenshot of Azure Copilot creating a pull request on GitHub to add generated files.":::

## Open in VS Code

You can choose to open the generated Terraform configurations in [Visual Studio Code for the Web](https://code.visualstudio.com/docs/setup/vscode-web). VS Code for the Web provides a free, zero-install Microsoft VS Code experience in your browser, allowing you to review and modify the files as needed.

To use this feature, select **Open in VS Code (Web)** after reviewing your generated Terraform configurations in the artifact pane. This action launches a VS Code web workspace with the generated files opened for review and editing.

:::image type="content" source="media/deployment-agent/deployment-open-visual-studio-code-web.png" alt-text="Screenshot of a Visual Studio for the Web workspace with files generated by Copilot in Azure." lightbox="media/deployment-agent/deployment-open-visual-studio-code-web.png" :::

## Current considerations and limitations

Keep in mind the following considerations and limitations when working with deployment in Agents (preview) in Azure Copilot.

- Currently, generated artifacts are only available as Terraform configurations.
- The deployment agent capabilities are designed to help you deploy brand new workloads and environments ("greenfield" scenarios). Agent capabilities don't currently support importing, analyzing, or modifying existing infrastructure. You can still ask Azure Copilot for guidance in these scenarios.
- While Azure Copilot provides guidance for secure deployment pipelines, it doesn't currently support automated integration of CI/CD workflows.
