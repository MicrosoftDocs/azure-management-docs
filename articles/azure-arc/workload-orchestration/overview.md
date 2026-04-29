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

## Key challenges addressed

Workload orchestration helps organizations by addressing the following common challenges:

- **Distributed configuration authoring:** Enables collaboration across teams while maintaining consistency in application configurations.
- **Edge contextualization:** Supports site‑specific customization for diverse devices and topologies.
- **Configuration validation:** Prevents misconfigurations through pre‑deployment checks and validation rules, avoiding costly downtime or productivity loss.
- **Version management:** Tracks application and configuration versions across deployments for auditing and rollback.
- **Observability:** Provides a unified view of deployment status and workload health across sites.
- **Security and governance:** Enforces Role‑Based Access Control (RBAC) to ensure users operate only within their designated scope.
- **Logging and traceability:** Captures comprehensive logs and errors for debugging, remediation, and compliance needs.

## What are the key features of workload orchestration?

Workload orchestration provides a centralized way to manage applications and their configurations across edge environments, enabling collaboration across IT, DevOps, and operations teams. Built‑in Role‑Based Access Control (RBAC) ensures that users can only access and manage resources within their assigned scope.

- **Fast onboarding and setup:** Guided workflows help you quickly configure your [organizational hierarchy](service-group.md#set-up-a-service-group-hierarchy-for-workload-orchestration), user roles, and access policies, so teams and edge infrastructure are ready for orchestration in minutes.
- **Template framework and schema inheritance:** Define [solution configurations](configuring-template.md) and [schemas](configuring-schema.md) once, then reuse or extend them for multiple deployments. Central IT teams can create a single source of truth for app configurations, which sites can inherit and customize as needed. This ensures consistency and reduces duplicate work.
- **Integrated monitoring and unified control:** Monitor deployments and workload health from a [centralized dashboard](deploy.md). Pause, retry, or roll back deployments as needed, with full logging and compliance visibility.
- **Dependent application management:** Deploy and manage interdependent applications using orchestrated workflows.
- **Custom and external validation rules:** Define pre-deployment validation rules to check parameter inputs and prevent misconfigurations. For advanced scenarios, [external validation](external-validation.md) lets you verify templates through services like Azure Functions or webhooks.
- **No-code authoring experience with RBAC:** The workload orchestration portal offers a no-code UI for [defining and updating application settings](configure.md), secured with role-based access control and audit logging.
- **CLI and automation support:** IT admins and DevOps engineers can use the CLI for scripted deployments, automation, and CI/CD integration, enabling bulk management of application lifecycles across sites.
- **Git integration:** In addition to the CLI and portal experience, workload orchestration provides the option to manage your resources and automate deployments from a [central Git repository](workload-orchestration-multicluster-git.md).

## How does workload orchestration work?

Workload orchestration uses both cloud and edge components to deliver a unified management experience. At its core, the cloud-based control plane leverages a dedicated Azure resource provider, allowing you to centrally define deployment templates. These templates are then consumed by workload orchestration agents running at edge locations, which automatically adapt and apply the necessary customizations for each site.

All workload orchestration resources are managed through Azure Resource Manager, enabling fine-grained [Role-Based Access Control (RBAC)](rbac-guide.md) and consistent governance. You can interact with workload orchestration using the [CLI](quickstart-solution-without-common-configuration.md) and [Azure portal](azure-portal-monitoring.md), while non-code onsite staff benefit from a [user-friendly interface](portal-user-guide.md) for authoring, monitoring, and deploying solutions with site-specific configurations.

### Architecture overview

:::image type="content" source="./media/workload-orchestration-architecture.png" alt-text="Diagram of the workload orchestration architecture" lightbox="./media/workload-orchestration-architecture.png":::

## Who can use workload orchestration?

Workload orchestration is designed for a wide range of users, who can be broadly categorized into two main groups: IT (Information Technology) and OT (Operational Technology) personas. 

### IT personas

IT personas usually have admin privileges that span across certain subcomponents. These users are responsible for managing the overall IT infrastructure and ensuring that applications are deployed and maintained correctly. Workload orchestration CLI provides a command-line interface for IT personas to manage the workload orchestration environment.

| Role | Responsibilities |
|------|------------------|
|IT Admin | Responsible for setting up and managing the physical hierarchy, user roles, and access control with RBACs. They also manage the overall IT infrastructure and monitor alerts. |
|IT DevOps | Responsible for writing configuration expressions and attributes, and submitting configuration changes. They also manage the deployment process and ensure that applications are running correctly. |
|Solution engineers | Responsible for solutions and their configurations. They onboard the existing apps, write configuration expressions and attributes, and define configuration schema. They also configure parameters for various applications, deploy the latest version of applications, and roll back when deployment fails. They also monitor application statuses and alerts. |

If you're an IT user and want to set up workload orchestration, follow the steps below:

- To manually set up workload orchestration, you can follow the instruction in [Prepare the environment for workload orchestration](initial-setup-environment.md) and [Setup workload orchestration](initial-setup-configuration.md). 
- To run the setup automatically, you can use the scripts in [Onboarding scripts](onboarding-scripts.md).


### OT personas

OT personas, also known as low-code/no-code personas, are users with limited privileges enabled for day-to-day business floor operations. OT personas can leverage the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview), which provides a user-friendly interface for no-code personas. For more information, see the [User guide of workload orchestration portal ](portal-user-guide.md).

The workload orchestration portal has three main tabs: **Monitor**, **Configure**, and **Deploy**. Each tab provides different functionalities and access levels based on the [RBACs assigned to the user](rbac-guide.md), typically defined by the IT admin. 

| Role | Responsibilities | Access required|
|------|------------------| ---------------|
|[Configuration of solutions](configure.md) | Responsible for configuring parameters at multiple hierarchical levels for various applications. They ensure that all values meet validation criteria before deployment.| Read-write access to factory, line, and solution levels.|       
|[Deployment](deploy.md) | Responsible for deploying latest version of applications and configurations on production, and roll back when deployment fails. | Deploy access on lines. |
|[Monitoring solutions](monitor.md) | Responsible for monitoring new versions of applications and hierarchical levels where new version of applications isn't yet deployed. | Read-only access to the solution and line levels.|

> [!NOTE]
> Line and factory levels are custom-defined by the IT admin. You can create up to four hierarchical levels and name them per your requirements.

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

