---
title: What Is Workload Orchestration?
description: Workload orchestration is a cross-platform orchestrator for managing edge workloads using an Azure control plane.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: overview
ms.date: 05/10/2025
ms.custom:
  - build-2025
---

# What is workload orchestration (preview)?

Workload orchestration for Azure Arc is a comprehensive, cloud-native, cross-platform orchestrator that simplifies the deployment, management, and update of application workloads across edge environments. Workload orchestration solves the Kubernetes application lifecycle management problem for customers who need to have common applications installed across fleets with site specific variations. 

[!INCLUDE [public-preview-note](includes/public-preview-note.md)]

## What problems does workload orchestration solve?

Imagine a factory with different production lines, modernizing and adding smart tech for better efficiency, security, and safety. The factory has legacy apps on Windows servers, new smart apps with IoT and AI running on Kubernetes, devices gathering sensor data, and Android tablets for employees to check dashboards. Deploying updates across all these systems is a huge headache, especially when you’re doing it for multiple lines in one factory, let alone dozens of factories worldwide. 

You’d need a ton of resources just to keep up before even considering incorporating actions around approvals, maintenance windows, canary deployments. Workload orchestration simplifies this process by providing a single platform to manage all your solutions. 

- **Diversity and complexity of the edge ecosystem:** Your organization needs to manage different device types, operating systems, hardware configurations, and service providers, each with their own capabilities, limitations, and interfaces.
- **Incompatible toolchains:** Even for simple solutions deployed in plants and factories, your organization needs to manage a combination of cloud native applications and legacy applications. Each new component adds to the challenge of defining and managing the whole solution.
- **Manual effort and coordination during deployment cycles:** Your organization needs to distribute and install their edge applications on various devices, often in remote or disconnected locations, and ensure that they're running the latest and most secure versions. This process requires manual effort, coordination, and network bandwidth and exposes customers to potential errors, failures, and security risks.
- **Lack of framework to handle administrative tasks:** Production solutions require administrative tasks that apply across multiple tools and applications, such as managing approvals, scheduling maintenance windows, and handling canary deployments.
- **Access control and security:** Your organization needs to ensure that only authorized users can access and manage their edge applications and devices, and that sensitive data is protected from unauthorized access or tampering. This is especially important in industries such as manufacturing, retail, and healthcare, where data privacy and security are critical.

## What are the key features of workload orchestration?

Workload orchestration provides a centralized platform for managing applications, their configurations and thus enabling better overall collaboration between the different personas who may interact with the system. The Role-Based Access Control (RBAC) feature ensures that only authorized users can access and manage the applications and devices.

1. **Easy onboarding:** IT admins set up and manage their physical hierarchy, user roles, and access control.
1. **Custom parameters and rules:** DevOps users can use a default template and schema for writing configuration expressions and attributes respectively. Then, no-code OT personas can define required configuration parameters, ensuring all values meet validation criteria before deployment.
1. **User-friendly portal:** No-code personas have a user interface to easily deploy applications and configurations on production, including real-time status updates, error logs, and rollback options. 
1. **Azure Portal for monitoring:** Azure portal provides the technical details about the deployment process, aiding IT persona in debugging helm issues and tracking running applications across sites.

## How does workload orchestration work?

Workload orchestration uses both cloud and edge components to deliver a unified management experience. At its core, the cloud-based control plane leverages a dedicated Azure resource provider, allowing you to centrally define deployment templates. These templates are then consumed by workload orchestration agents running at edge locations, which automatically adapt and apply the necessary customizations for each site.

All workload orchestration resources are managed through Azure Resource Manager, enabling fine-grained Role-Based Access Control (RBAC) and consistent governance. You can interact with workload orchestration using an intuitive CLI and portal, while non-technical onsite staff benefit from a no-code interface for authoring, monitoring, and deploying solutions with site-specific configurations.

:::image type="content" source="./media/workload-orchestration-architecture.png" alt-text="Diagram of the workload orchestration architecture" lightbox="./media/workload-orchestration-architecture.png":::

## Who can use workload orchestration?

Workload orchestration is designed for a wide range of users, which can be broadly categorized into two main groups: IT(Information Technology) and OT (Operational Technology) personas. IT personas are responsible for setting up the environment, while OT personas are responsible for managing the applications. The workload orchestration portal provides a user-friendly interface for OT personas to manage their applications, while IT personas can use the Azure CLI to set up the environment.

### IT personas

IT personas usually have admin privileges that span across certain subcomponents. These users are responsible for managing the overall IT infrastructure and ensuring that applications are deployed and maintained correctly. Workload orchestration CLI provides a command-line interface for IT personas to manage the workload orchestration environment.

- **IT Admin:** Responsible for setting up and managing the physical hierarchy, user roles, and access control with RBACs. They also manage the overall IT infrastructure and monitor alters.
- **IT DevOps:** Responsible for writing configuration expressions and attributes, as well as submitting configuration changes. They also manage the deployment process and ensure that applications are running correctly.

If you're an IT user and want to set up workload orchestration, follow the steps below:

- To manually set up workload orchestration, you can follow the instruction in [Prepare the environment for workload orchestration](initial-setup-environment.md) and [Setup workload orchestration](initial-setup-configuration.md). 
- To run the setup automatically, you can use the scripts in [Onboarding scripts](onboarding-scripts.md).

#### Organization with central IT team

If your organization has a central IT team and no OT personas, workload orchestration covers the end-to-end journey with IT personas such as platform engineers and solution engineers. Both platform engineers and solution engineers have access to the Azure CLI and can use it to set up the environment, manage the overall IT infrastructure, and monitor alerts and infrastructure statuses.

- **Platform engineers:** Responsible for setting up initial infrastructure, profiles of all personas with RBACs and hierarchy level configurations. They also monitor alerts and infrastructure statuses.
- **Solution engineers:** Responsible for solutions and their configurations. They onboard the existing apps, write configuration expressions and attributes, and define configuration schema. They also configure and publish parameters for various applications, deploy the latest version of applications, and roll back when deployment fails. They also monitor application statuses and alerts.

### OT personas

OT personas, also known as no-code personas, are users with limited privileges enabled for day-to-day business floor operations. OT personas use the workload orchestration portal to manage workload orchestration.

- **OT responsible for configuration authoring:** Responsible for configuring parameters at multiple hierarchical levels for various applications. They ensure that all values meet validation criteria before deployment.
- **OT responsible for monitoring:** Responsible for monitoring new versions of applications and hierarchical levels where new version of applications isn't yet deployed
- **OT responsible for deployment:** Responsible for deploying latest version of applications and configurations on production, and roll back when deployment fails.

#### Workload orchestration portal for OT personas

Workload orchestration portal provides a user-friendly interface to allow no-code personas to easily manage various tasks such as fill in required configuration parameters, ensuring all values meet validation criteria before deployment and deploy applications with configurations on production, including real-time status updates, error logs, and rollback options. 

After logging in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview), you can view three main tabs: **Monitor**, **Configure**, and **Deploy**. Each tab provides different functionalities and access levels based on the RBACs assigned to the user. The access to these features is controlled by the [RBACs assigned to the user](rbac-guide.md).


|Tab|Actions| Access required|
|----|-------|----------------|
|[Monitor](monitor.md)|View the solutions, lines, status, and other details.|Read-only access to the solution and line levels.|
|[Configure](configure.md)|Configure parameters at factory, line and solution levels. View revision details and publish to lines.|Read-write access to factory, line, and solution levels.|
|[Deploy](deploy.md)|Deploy solutions and roll back to earlier versions if needed.|Deploy access on lines.|

> [!NOTE]
> Line and factory levels are custom-defined by the IT admin. You can create up to four hierarchical levels and name them per your requirements. 



