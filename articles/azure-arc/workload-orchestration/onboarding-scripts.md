---
title: Onboarding Scripts for Workload Orchestration
description: The onboarding scripts are designed to help you set up the necessary infrastructure and resources for workload orchestration in Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: install-set-up-deploy
ms.date: 06/10/2025
ms.custom:
  - build-2025
# Customer intent: "As a system administrator, I want to automate the setup of infrastructure and resources for workload orchestration, so that I can efficiently provision a Kubernetes cluster and related services without manual intervention."
---

# Set up workload orchestration using onboarding scripts

The onboarding scripts automate the end-to-end setup of infrastructure and resources for workload orchestration in Azure Arc. Rather than running individual CLI commands, the scripts handle the entire process of [setting up workload orchestration](set-up-workload-orchestration.md) to deploy your first application. The scripts are available in three variants—PowerShell, Python, and Bash—all of which are functionally equivalent.

> [!TIP]
> If you prefer full control over each step and want to run commands individually, follow the instructions in [Set up using CLI](set-up-workload-orchestration.md). For a Git-based declarative approach with Bicep templates, follow [Set up using Git](workload-orchestration-multicluster-git.md)

## How the scripts work

The [onboarding scripts](https://github.com/Azure/workload-orchestration/tree/main/Onboarding%20script) for workload orchestration execute the following processes: 

1. Installs the required Azure CLI extensions.
1. Initializes a new or existing Azure Arc-enabled cluster and prepares it for deployments.
1. Creates the workload orchestration environment, Site hierarchy, and deployment targets within your cluster.
1. Creates schema and solution template that work as blueprints for application deployments.

The scripts enable the reuse of previously provisioned infrastructure and configuration. To start using the scripts or learn more, refer to the [README](https://github.com/Azure/workload-orchestration/blob/main/Onboarding%20script/README.md) section within the [workload orchestration GitHub repository](https://github.com/Azure/workload-orchestration-quickstart).

## Next steps

> [!div class="nextstepaction"]
> [Deploy a basic solution](solution-without-common-configuration.md)