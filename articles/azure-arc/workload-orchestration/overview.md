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
 
Workload orchestration for Azure Arc is a cloud‑native, cross‑platform service that enables centralized deployment, configuration, and lifecycle management of application workloads across distributed edge environments. It helps organizations deploy applications consistently across multiple fleets with site‑specific configurations, and natively supports Kubernetes workloads.

## Why use workload orchestration?

Imagine a manufacturing company, Contoso, where workers use a simple on‑premises app to monitor factory machines such as motors, pumps, and mixers. The app runs locally inside each factory—not in the cloud. Contoso operates multiple factories, and while the same app runs everywhere, each site needs it configured differently. One factory may have 10 machines, another 30. Safety thresholds may vary, and alerts might need to be delivered in different languages.

When the app is updated—whether for a new feature or a bug fix—it must be deployed carefully to every factory, ensuring each site’s unique settings remain intact. And this challenge isn’t limited to a single application. Most factories run many workloads, from monitoring and predictive maintenance to legacy systems and AI‑powered apps. These same challenges apply beyond manufacturing, across industries like retail, quick‑service restaurants, energy, and healthcare, where distributed operations rely on consistent yet localized applications.

## What are the key features of workload orchestration?

- **Fast onboarding and setup:** Guided workflows help you quickly configure your [organizational hierarchy](setup-wo.md#step-3-create-the-hierarchy), user roles, and access policies, so teams and edge infrastructure are ready for orchestration in minutes.
- **Application lifecycle management:** Rapidly scale application rollouts and centrally manage them across distributed edge environments with custom site‑specific configurations and version control, enabling collaboration across IT, DevOps, and operations teams.
- **Template framework and schema inheritance:** Define [solution configurations](solution-without-common-configuration.md#create-the-solution-template) and [schemas](solution-without-common-configuration.md#create-a-configuration-schema) once, then reuse or extend them for multiple deployments. Central IT teams can create a single source of truth for app configurations, which sites can inherit and customize as needed. This ensures consistency through pre‑deployment checks and validation rules, and reduces duplicate work.
- **Integrated monitoring and unified control:** Monitor deployments and workload health from a [centralized dashboard](https://portal.digitaloperations.configmanager.azure.com). Pause, retry, or roll back deployments as needed, with full logging and compliance visibility.
- **Interface versatility:** IT admins and DevOps engineers can use the CLI for scripted deployments, automation, and CI/CD integration, enabling bulk management of application lifecycles across sites. The [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/), on the other hand, offers a no-code UI for configuring and deploying solutions at scale.
- **Git integration:** In addition to the CLI and portal experience, workload orchestration provides the option to manage your resources and automate deployments from a [central Git repository](workload-orchestration-multicluster-git.md).
- **Dependent application management:** Deploy and manage interdependent applications using orchestrated workflows.
- **Custom validation rules:** Define pre-deployment validation rules to check parameter inputs and prevent misconfigurations. For advanced scenarios, [external validation](external-validation.md) lets you verify templates through services like Azure Functions or webhooks.
- **Security and governance:** Enforce Role‑Based Access Control (RBAC) to ensure users can only access and manage resources within their assigned scope.

## How does workload orchestration work?

Workload orchestration uses both cloud and edge components to deliver a unified management experience. At its core, the cloud-based control plane leverages a dedicated Azure resource provider, allowing you to centrally define deployment templates. These templates are then consumed by workload orchestration agents running at edge locations, which automatically adapt and apply the necessary customizations for each site.

### Architecture overview

:::image type="content" source="./media/workload-orchestration-architecture.png" alt-text="Diagram of the workload orchestration architecture" lightbox="./media/workload-orchestration-architecture.png":::

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

