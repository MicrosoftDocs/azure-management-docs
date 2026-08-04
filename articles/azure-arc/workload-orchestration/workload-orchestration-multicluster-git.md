---
title: Set up workload orchestration using Git
description: Learn how to set up and manage Workload Orchestration resources as Bicep templates in a Git repository, with automated validation and deployment via Azure Deployment Stacks.
author: nathmanish
ms.author: nathmanish
ms.topic: how-to
ms.date: 04/24/2026
ms.custom:
  - build-2025
# Customer intent: As a platform engineer, I want to manage workload orchestration resources as Bicep templates in Git, so that I can apply GitOps practices, enforce review through pull requests, and protect deployed resources from out-of-band changes.
---

# Set up workload orchestration using Git

This approach lets you manage all workload orchestration resources — environment, hierarchy, targets, schemas and templates — in the form of Bicep templates in a Git [repository](https://github.com/Azure/workload-orchestration-quickstart), and trigger single-click solution deployments. Leveraging preconfigured GitHub Actions and [Azure Deployment Stacks](/azure/azure-resource-manager/bicep/deployment-stacks), you can validate changes through pull requests, deploy resources automatically on merge, and protect Git-managed resources from out-of-band changes. This is ideal for teams that want to apply GitOps practices with PR-based review and automated deployment.

> [!TIP]
> For alternative setup methods, see [Set up using CLI](set-up-workload-orchestration.md) for a manual controlled approach, or [Set up using scripts](onboarding-scripts.md) for scripted automation.


## How it works

Your GitHub repository uses a Bicep template `main.bicep` to provision your workload orchestration resources, supported by other Bicep modules that declare those resources. Two GitHub Actions workflows enforce validation and synchronization between your repository and Azure account to manage the declared resources, and another workflow helps you deploy applications to selected targets. 

This is how you manage your workload orchestration resources from your Git repository:

1. **Edit** `workload-orchestration/main.bicep` in a feature branch to add, update, or remove resource declarations.
1. **Open a pull request** to the `main` branch. The validation workflow runs automatically and:
   - Validates the Bicep templates against Azure.
   - Posts a validation result comment on the Pull Request.
1. **Merge** the pull request into `main`. The sync workflow deploys the resources to Azure by using a deployment stack with configurable deny settings.

Additional details on how to set up your Git repository can be found in the [workload orchestration GitHub repository](https://github.com/Azure/workload-orchestration-quickstart). To quickly deploy a sample application using preauthored Bicep templates, refer to this [README](https://github.com/Azure/workload-orchestration-quickstart/blob/main/samples/quickstart-basic/README.md) section within the repository.

## Related content

- [Use GitHub Actions to connect to Azure](/azure/developer/github/connect-from-azure)
- [Azure Deployment Stacks overview](/azure/azure-resource-manager/bicep/deployment-stacks)
- [Set up workload orchestration](set-up-workload-orchestration.md)
