---
title: What Is Workload Orchestration?
description: Workload orchestration is a cross-platform orchestrator for managing edge workloads using an Azure control plane.
author: sethmanheim
ms.author: sethm
ms.topic: overview
ms.date: 06/22/2025
ms.custom:
  - build-2025
# Customer intent: "As an IT admin managing diverse edge environments, I want to use a centralized workload orchestration platform, so that I can efficiently deploy and monitor applications while minimizing manual errors and enhancing security across multiple locations."
---

# What is workload orchestration?
 
Workload orchestration is a centralized approach to deploying, configuring, and managing application workloads across distributed environments—including cloud, on-premises, and edge. It enables teams to define the desired state of their applications, such as the Helm chart, version, configuration parameters, and target environments, and ensures that state is consistently achieved and maintained at scale.

## What are the key features of workload orchestration?

- **Centralized deployment:** Deploy applications across multiple clusters from a single control plane, eliminating manual on-site operations.
- **Configuration management:** Apply environment-specific configurations while maintaining a consistent deployment blueprint.
- **Lifecycle management:** Handle upgrades, rollbacks, and version control seamlessly for all deployments.
- **Scalability:** Rapidly scale application rollouts across thousands of sites with minimal operational overhead.
- **Observability:** Monitor deployments status, workload health and compliance from a [centralized dashboard](https://portal.digitaloperations.configmanager.azure.com).

## How to use workload orchestration?

Workload orchestration operates in two key steps: 

- Setup: It is a one-time operation that prepares your environment for workload deployments. It involves initializing workload orchestration on existing ARC connected Kubernetes infrastructure, and provisioning targets that map to particular namespaces within the clusters where the applications are to be deployed.

- Deploy: With setup in place, you can begin deploying applications consistently across your targets. This involves creating a blueprint for the application to be deployed, and leveraging the same consistently across all relevant targets.

To get started with workload orchestration, refer to the [setup guide](setup-workload-orchestration.md).

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]