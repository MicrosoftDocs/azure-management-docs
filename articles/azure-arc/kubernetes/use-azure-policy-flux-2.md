---
title: "Deploy applications consistently at scale using Flux v2 configurations and Azure Policy"
ms.date: 02/14/2025
ms.topic: how-to
description: "Use Azure Policy to apply Flux v2 configurations at scale on Azure Arc-enabled Kubernetes or AKS clusters."
# Customer intent: As a Kubernetes administrator, I want to deploy Flux v2 configurations using Azure Policy so that I can ensure consistent application management across all my clusters at scale.
---

# Deploy applications consistently at scale using Flux v2 configurations and Azure Policy

You can use [Azure Policy](/azure/governance/policy/) to apply Flux v2 configurations (`Microsoft.KubernetesConfiguration/fluxConfigurations` resource type) at scale on Azure Arc-enabled Kubernetes (`Microsoft.Kubernetes/connectedClusters`) or Azure Kubernetes Service (AKS) (`Microsoft.ContainerService/managedClusters`) clusters. To use Azure Policy, you select a built-in policy definition and [create a policy assignment](/azure/governance/policy/tutorials/create-and-manage).

Before you assign the policy that creates Flux configurations, you must ensure that the Flux extension is deployed to your clusters. You can assign a policy to ensure the extension is deployed to all clusters in the selected scope (all resource groups in a subscription or management group, or to specific resource groups). Then, when creating the policy assignment to deploy configurations, you set parameters for the Flux configuration to be applied to the clusters in that scope.

To enable separation of concerns, you can create multiple policy assignments, each with a different Flux v2 configuration pointing to a different source. For example, cluster admins can use one Git repository, while application teams use another.

## Built-in policy definitions

The following [built-in policy definitions](policy-reference.md) provide support for these scenarios:

|Description  |Policy  |
|---------|---------|
|Flux extension install (required for all scenarios)     |  `Configure installation of Flux extension on Kubernetes cluster`       |
|Flux configuration using public Git repository (generally a test scenario)     | `Configure Kubernetes clusters with Flux v2 configuration using public Git repository`        |
|Flux configuration using private Git repository with SSH auth     | `Configure Kubernetes clusters with Flux v2 configuration using Git repository and SSH secrets`        |
|Flux configuration using private Git repository with HTTPS auth     | `Configure Kubernetes clusters with Flux v2 configuration using Git repository and HTTPS secrets`        |
|Flux configuration using private Git repository with HTTPS CA cert auth     | `Configure Kubernetes clusters with Flux v2 configuration using Git repository and HTTPS CA Certificate`        |
|Flux configuration using private Git repository with local K8s secret     |  `Configure Kubernetes clusters with Flux v2 configuration using Git repository and local secrets`       |
|Flux configuration using private Bucket source and KeyVault secrets     | `Configure Kubernetes clusters with Flux v2 configuration using Bucket source and secrets in KeyVault`      |
|Flux configuration using private Bucket source and local K8s secret     | `Configure Kubernetes clusters with specified Flux v2 Bucket source using local secrets`        |

To find all of the Flux v2 policy definitions, search for **flux**. For direct links to these policies, see [Azure policy built-in definitions for Azure Arc-enabled Kubernetes](policy-reference.md).

## Prerequisites

* One or more Arc-enabled Kubernetes clusters and/or AKS clusters.
* `Microsoft.Authorization/policyAssignments/write` permissions on the scope (subscription or resource group) to create the policy assignments.

## Create a policy assignment to install the Flux extension

In order for a policy to apply Flux v2 configurations to a cluster, the Flux extension must be installed on the cluster. To ensure that the extension is installed on each of your clusters, assign the **Configure installation of Flux extension on Kubernetes cluster** policy definition to the desired scope.

1. In the Azure portal, navigate to **Policy**.
1. In the service menu, under **Authoring**, select **Definitions**.
1. Find the **Configure installation of Flux extension on Kubernetes cluster** built-in policy definition, and select it.
1. Select **Assign policy**.
1. Set the **Scope** to the management group, subscription, or resource group to which the policy assignment will apply. If you want to exclude any resources from the policy assignment scope, set **Exclusions**.
1. Give the policy assignment an easily identifiable **Assignment name** and **Description**.
1. Ensure **Policy enforcement** is set to **Enabled**.
1. Select **Review + create**, then select **Create**.

## Create a policy assignment to apply Flux configurations

Next, return to the **Definitions** list (in the **Authoring** section of **Policy**) to apply the configuration policy definition to the same scope.

1. Find and select the **Configure Kubernetes clusters with Flux v2 configuration using public Git repository** built-in policy definition, or one of the other policy definitions that applies Flux configurations.
1. Select **Assign policy**.
1. Set the **Scope** to the same scope that you selected when assigning the first policy, including any exclusions.
1. Give the policy assignment an easily identifiable **Assignment name** and **Description**.
1. Ensure **Policy enforcement** is set to **Enabled**.
1. Select **Next** to open the **Parameters** tab.
1. Set the parameter values to be used, using the parameter names from the policy definition.
    * For more information about parameters, see [GitOps (Flux v2) supported parameters](./gitops-flux2-parameters.md).
    * When creating Flux configurations via policy, you must provide a value for one (and only one) of these parameters: `repositoryRefBranch`, `repositoryRefTag`, `repositoryRefSemver`, `repositoryRefCommit`.
1. Select **Next** to open the **Remediation** task.
1. Enable **Create a remediation task**.
1. Verify that **Create a Managed Identity** is checked, and that **Contributor** is listed in the **Permissions** section. For more information, see [Quickstart: Create a policy assignment to identify non-compliant resources](/azure/governance/policy/assign-policy-portal) and [Remediate non-compliant resources with Azure Policy](/azure/governance/policy/how-to/remediate-resources).

1. Select **Review + create**, then select **Create**.

The configuration is then applied to new clusters created within the scope of policy assignment.

For existing clusters, you might need to manually run a remediation task. The policy assignment will take effect after the remediation task finishes running (typically 10 to 20 minutes).

## Verify the policy assignment

1. In the Azure portal, navigate to an Azure Arc-enabled Kubernetes or AKS cluster that's within the scope of the policy assignment.
1. In the service menu, under **Settings**, select **GitOps**. In the **Configurations** list, you should see the configuration created by the policy assignment.
1. In the service menu, under **Kubernetes resources (preview)**, select **Namespaces**. You should see the namespace that was created by the Flux configuration.

## Customize a policy

The built-in policies cover the main scenarios for using GitOps with Flux v2 in your Kubernetes clusters. However, due to the limit of 20 parameters allowed in Azure Policy assignments, not all parameters are included in the built-in policies. Also, to fit within this 20-parameter limit, only a single kustomization can be created with the built-in policies.  

If you have a scenario that differs from the built-in policies, you can overcome these limitations by creating [custom policy definitions](/azure/governance/policy/tutorials/create-custom-policy-definition) using the built-in policies as templates. To work around the 20-parameter limit, create custom policies that contain only the parameters you need and hard-code the rest.

## Next steps

* Learn more about [deploying applications using GitOps with Flux v2](tutorial-use-gitops-flux2.md).
* [Set up Azure Monitor for Containers with Azure Arc-enabled Kubernetes clusters](/azure/azure-monitor/containers/container-insights-enable-arc-enabled-clusters).
