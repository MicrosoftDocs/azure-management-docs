---
title: What Is Workload Orchestration?
description: Workload orchestration is a cross-platform orchestrator for managing edge workloads using an Azure control plane.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: overview
ms.date: 06/22/2025
ms.custom:
  - build-2025
# Customer intent: "As an IT admin managing diverse edge environments, I want to use a centralized workload orchestration platform, so that I can efficiently deploy and monitor applications while minimizing manual errors and enhancing security across multiple locations."
---

# What is workload orchestration?

Workload orchestration for Azure Arc is a comprehensive, cloud-native, cross-platform service engine that simplifies the deployment, management, and update of application workloads across edge environments. Workload orchestration addresses typical application lifecycle management problems for customers who need to have application deployments across multiple fleets with site-specific configurations. It natively supports kubernetes workloads. 

## What problems does workload orchestration solve?

Imagine a company, Contoso manufacturing, where workers use a simple app to monitor the status of factory machines — like motors, pumps, or mixers. This app doesn’t run in the cloud. It runs on-premises, inside the factory, on a computer. Now imagine that Contoso has multiple locations, each with its own computer running the same app, but each location needs it set up a bit differently. Contoso factory A might have 10 machines while Contoso factory B has 30. Contoso factory A might have different safety thresholds than Contoso factory B. Contoso factory A might need alerts in English, while Contoso factory B needs them in Spanish.

So whenever there’s an update to this app — like a new feature or a bug fix — it has to be deployed carefully to each computer in every location, making sure it keeps the factory-specific settings intact. That’s already a big job for one app. But in reality, factories don’t run on just one app. They have many — some monitoring sensors, others doing predictive maintenance, some running on old Windows systems, others powered by AI.

These challenges aren't limited to manufacturing — workload orchestration is sector-agnostic and highly relevant for enterprise customers in industries like retail, quick service restaurants, energy, and healthcare, where distributed operations rely on consistent, localized applications.

Workload orchestration addresses several key challenges faced by organizations managing applications at the edge:

- **Distributed configuration authoring:** Managing configuration files for multiple applications often requires input from different stakeholders across various edge locations, making collaboration and consistency difficult.
- **Edge contextualization:** Edge environments typically include diverse devices and complex topologies, each requiring tailored configurations to meet site-specific needs.
- **Configuration validation:** Ensuring that configuration parameters are correct before deployment is critical to prevent misconfigurations and avoid costly downtime or productivity loss.
- **Version management:** Maintaining multiple versions of application code and configuration files can complicate auditing and tracking changes across deployments.
- **Lack of visibility:** Without a unified view of applications and deployment status, identifying failures and optimizing operations becomes a manual, resource-intensive process.
- **Role-Based Access Control (RBAC):** Enforcing role-based access ensures that only authorized users can manage and operate within their designated scope, improving security and governance.
- **Logging and traceability:** Comprehensive logging and error tracing are essential for effective debugging, remediation, and compliance.

## What are the key features of workload orchestration?

Workload orchestration provides a centralized platform for managing applications, their configurations and thus enabling better overall collaboration between the different personas who may interact with the system. The Role-Based Access Control (RBAC) feature ensures that only authorized users can access and manage the applications and devices.

- **Template framework and schema inheritance:** Define [solution configurations](configuring-template.md) and [schemas](configuring-schema.md) once, then reuse or extend them for multiple deployments. Central IT teams can create a single source of truth for app configurations, which sites can inherit and customize as needed. This ensures consistency and reduces duplicate work.
- **Dependent application management:** Deploy and manage interdependent applications using orchestrated workflows. Workload orchestration supports configuring and deploying apps with dependencies through the [CLI](quickstart-solution-without-common-configuration.md) or [workload orchestration portal](portal-user-guide.md), reducing errors and streamlining complex rollouts.
- **Custom and external validation rules:** Administrators can define pre-deployment validation rules to check parameter inputs and settings, preventing misconfigurations. For advanced scenarios, [external validation](external-validation.md) lets you verify templates through services like Azure Functions or webhooks, enabling business-specific logic and reducing runtime errors.
- **Integrated monitoring and unified control:** Monitor deployments and workload health from a [centralized dashboard](deploy.md). Pause, retry, or roll back deployments as needed, with full logging and compliance visibility.
- **No-code authoring experience with RBAC:** The workload orchestration portal offers a no-code UI for [defining and updating application settings](configure.md), secured with role-based access control and audit logging. Non-developers can safely make approved changes without compromising security.
- **CLI and automation support:** IT admins and DevOps engineers can use the CLI for scripted deployments, automation, and CI/CD integration, enabling bulk management of application lifecycles across sites.
- **Fast onboarding and setup:** Guided workflows help you quickly configure your [organizational hierarchy](service-group.md#set-up-a-service-group-hierarchy-for-workload-orchestration), user roles, and access policies, so you can onboard teams and prepare [edge infrastructure](initial-setup-environment.md) for orchestration in minutes.

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

If you're an IT user and want to set up workload orchestration, follow the steps below:

- To manually set up workload orchestration, you can follow the instruction in [Prepare the environment for workload orchestration](initial-setup-environment.md) and [Setup workload orchestration](initial-setup-configuration.md). 
- To run the setup automatically, you can use the scripts in [Onboarding scripts](onboarding-scripts.md).

#### Organization with central IT team

If your organization has a central IT team and no OT personas, workload orchestration covers the end-to-end journey with IT personas such as platform engineers and solution engineers. Both platform engineers and solution engineers have access to the Azure CLI and can use it to set up the environment, manage the overall IT infrastructure, and monitor alerts and infrastructure statuses.

| Role | Responsibilities |
|------|------------------|
|Platform engineers | Responsible for setting up initial infrastructure, profiles of all personas with RBACs and hierarchy level configurations. They also monitor alerts and infrastructure statuses. |
|Solution engineers | Responsible for solutions and their configurations. They onboard the existing apps, write configuration expressions and attributes, and define configuration schema. They also configure and publish parameters for various applications, deploy the latest version of applications, and roll back when deployment fails. They also monitor application statuses and alerts. |

### OT personas

OT personas, also known as low-code/no-code personas, are users with limited privileges enabled for day-to-day business floor operations. OT personas use the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview), which provides a user-friendly interface to allow no-code personas to easily manage various tasks. For more information, see the [User guide of workload orchestration portal ](portal-user-guide.md).

The workload orchestration portal has three main tabs: **Monitor**, **Configure**, and **Deploy**. Each tab provides different functionalities and access levels based on the RBACs assigned to the user. The access to these features is controlled by the [RBACs assigned to the user](rbac-guide.md), which are defined by the IT admin. 

| Role | Responsibilities | Access required|
|------|------------------| ---------------|
|[Configuration of solutions](configure.md) | Responsible for configuring parameters at multiple hierarchical levels for various applications. They ensure that all values meet validation criteria before deployment.| Read-write access to factory, line, and solution levels.|       
|[Deployment](deploy.md) | Responsible for deploying latest version of applications and configurations on production, and roll back when deployment fails. | Deploy access on lines. |
|[Monitoring solutions](monitor.md) | Responsible for monitoring new versions of applications and hierarchical levels where new version of applications isn't yet deployed. | Read-only access to the solution and line levels.|

> [!NOTE]
> Line and factory levels are custom-defined by the IT admin. You can create up to four hierarchical levels and name them per your requirements.

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

