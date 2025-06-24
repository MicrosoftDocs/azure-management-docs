---
title: Workflow and Features in Workload Orchestration
description: Learn about the workflow in workload orchestration and the features available for authoring, deploying, and managing solutions.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: concept-article
ms.date: 06/24/2025
---

# Workflows and features in workload orchestration

This article provides an overview of the workflow in workload orchestration and the available features to easily author, deploy, and manage solutions.

For information about the prerequisites and how to set up workload orchestration, see [Prepare your environment for workload orchestration](initial-setup-environment.md) and [Set up workload orchestration](initial-setup-configuration.md).

## Solution authoring and deployment 

The process of authoring and deploying solutions using workload orchestration involves creating schemas, configuration templates, and deploying solutions to the target environments. This process is typically performed by IT DevOps.

### Hierarchical configuration authoring

Solution authoring requires seamless integration of workload orchestration-managed ARM assets with user-provided artifacts, such as Helm charts and container images. Workload orchestration supports both CLI and portal-based methods for authoring solution configurations across various use cases.

The diagram below represents an example hierarchical configuration of objects:

:::image type="content" source="./media/hierarchy-configuration-objects.png" alt-text="Diagram illustrating the hierarchical configuration of objects in the Solution Authoring and Deployments process." lightbox="./media/hierarchy-configuration-objects.png":::

A typical solution consists of:

- **Schema:** A schema is a JSON file that represents the declaration of configurable attributes of the solution and the associated permissions as it applies to hierarchies and personas. The schema is used to define the structure and format of the configuration data that is used in the solution. The schema is used to validate the configuration data before it's deployed to the target environment. For more information, see [Configuration Schema](configuring-schema.md).
- **Configuration template:** A configuration template is a JSON file that  represents associated configurations of the previously declared schema. These values can be modified as necessary. See [Configuration template](configuring-template.md) for the list of rules used to define the template and schema for configurations. This page also details the steps to write conditional or nested expressions.
- **Solution Helm chart:** A solution Helm chart is a package that contains all the necessary files and resources to deploy the solution to the target environment. The solution Helm chart integrates the configurable workload orchestration assets with the user provided solution artifacts. Solutions must be packaged as containers before uploading them to workload orchestration.
- **Published solution configuration:** A published solution configuration is a JSON file that represents the final configuration of the solution after it's validated and approved. The published solution configuration is created by combining the schema, configuration template, and solution Helm chart. The published solution represents a fully rendered, a pre-deployment ready, targeted solution. At this point, the solution is ready to be deployed.

### Helm deployment

When deploying solutions using Helm, a release name is required. The release name is a unique identifier for the deployed solution instance in the Kubernetes cluster. It helps in managing and tracking the deployed resources. 

You can provide the release name as part of specifications in the [solution template](configuring-template.md). The release name should follow the naming conventions for Kubernetes resources, which typically include lowercase alphanumeric characters, dashes, and dots.

```json
{
  "components": [
    {
      "name": "<name>",
      "type": "helm.v3",
      "properties": {
        "chart": {
          "repo": "acr",
          "version": "0.8.0",
          "wait": true,
          "timeout": "5m"
        },
        "releaseName": "test-release"
      }
    }    
  ]
}
```

If you choose not to specify a Helm release name, omit the `releaseName` property from the specifications. Workload orchestration automatically generates a unique hash to use as the release name.

### Solution authoring scenarios

There are different variants of schemas that can be used to author solutions. See the following tutorials for examples of different solution authoring scenarios:

- **Shared schema:** This schema comprises configurable attributes/properties that can be used across hierarchies and solutions. For more information, see [Create a basic solution](quickstart-solution-without-common-configuration.md).
- **Common schema:** This schema defines configurable attributes/properties at each hierarchical level that can be used for a particular solution. For more information, see [Create a basic solution with common configurations](quickstart-solution-with-common-configuration.md).
- **Schema with dependencies:** This schema defines configurable attributes/properties of an application dependent on another application. For more information, see [Create a solution with shared adapter dependencies](quickstart-solution-shared-adapter-dependency.md).
- **Multiple shared adapter dependencies:** This schema defines configurable attributes/properties of an application dependent on more than one other application. For more information, see [Create a solution with multiple shared adapter dependencies](quickstart-solution-multiple-shared-adapter-dependency.md).
- **Deploy multiple instances of the same application:** This schema defines configurable attributes/properties of an application that can be deployed multiple times in the same namespace. For more information, see [Create a solution with multiple instances](quickstart-solution-multiple-instances-k8s.md).
- **Upgrade a shared solution:** This schema shows how to upgrade shared solution along with dependent solutions. For more information, see [Upgrade a shared solution](quickstart-upgrade-shared-application.md).


## Application and solution versioning

Application and solution versions must be manually updated. It's recommended to follow Semantic Versioning, a widely adopted versioning scheme that uses a three-part version number format: `major.minor.patch`. Each part of the version number reflects the scope of changes:

- *Major:* Incremented for changes that are not backward-compatible.
- *Minor:* Incremented for new features that are backward-compatible.
- *Patch:* Incremented for backward-compatible bug fixes.

Version information can also be specified in a file instead of being passed as a CLI argument when creating solution templates, schemas, or configuration templates. To include version details, add the following section to the YAML file:

```yaml
metadata:
    name: <name> [optional]
    version: <version> [optional]
```

## Ways to author configurations

Once solution is uploaded to workload orchestration, IT DevOps author the configuration template and schema for validation rules. IT DevOps can provide technical configurations and set the default values and ranges for all configurations in the template and schema.

- **Azure CLI**: IT DevOps can provide values for these configurations via CLI. The CLI will validate the configurations and publish them to the target. For more information, see [Solution authoring scenarios](#solution-authoring-scenarios).
- **Workload orchestration portal**: The published solutions and configurations are reflected on the portal. Any no-code persona is able to view the configurations and provide values via workload orchestration portal based on RBAC. For more information, see [Configure your solutions](configure.md).

## Revisions of configurations

When user provides values for solution configurations and publishes them for certain targets, revisions of configurations are created for each target. These revisions are incremented with each new change made by user for respective target.

## Ways to monitor solutions



## Staging before deployment

Workload orchestration supports staging of solutions before deployment. Staging allows users to download the artifacts and validate the configurations before deploying them to the edge cluster. Staging is an optional step, but it's beneficial for some user scenarios with large-scale deployments or network latency issues. 

For more information, see [Staging before deployment](how-to-stage.md).

## Service groups 

Service groups are a new resource type in Azure Resource Manager (ARM) that allow you to organize selected resources into a unified logical grouping in a way that matches your real-world structure. With service groups you can organize resources in a way that reflects your organizational hierarchical structure, such as departments, teams, or projects. Relationships are created between resources to establish a hierarchical structure such as parent and child associations.

For more information, see [Service groups](service-group.md).

## Diagnostics 


## Bulk deployment 
