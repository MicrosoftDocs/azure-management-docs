---
title: Upgrade a Shared Application with Workload Orchestration
description: Learn how to upgrade a shared application using workload orchestration via CLI.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: quickstart
ms.date: 05/10/2025
ms.custom:
  - build-2025
# Customer intent: As a solutions engineer, I want to upgrade shared applications using workload orchestration via command line, so that I can efficiently deploy updates and manage dependencies between interconnected solutions.
---

# Quickstart: Upgrade a shared solution

In this quickstart, you upgrade a shared Factory Sensor Anomaly Detector (FSAD) solution and deploy a dependent Shared Sync Adapter (SSA) solution using workload orchestration and the Azure CLI. 

The FSAD solution is deployed on a child target, while the SSA solution is deployed on a parent target. The FSAD solution uses the SSA solution to synchronize data between devices and servers. For more information about the FSAD and SSA solutions, see [Quickstart: Create a solution with multiple shared adapter dependencies](quickstart-solution-multiple-shared-adapter-dependency.md).

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/microsoft/AEP/blob/main/content/en/docs/Configuration%20Manager%20(Public%20Preview)/Scripts%20for%20Onboarding/Configuration%20manager%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Configure the resources of workload orchestration](initial-setup-configuration.md#configure-the-resources-of-workload-orchestration).


## Prepare the setup

### [Bash](#tab/bash)

1. Before proceeding, ensure that the targets for the shared application (FSAD) and the dependent solution (SSA) are created. Additionally, create solution templates for both FSAD and SSA, ensuring all dependencies are properly configured.

    > [!IMPORTANT]
    > When configuring the SSA solution template, include only the solution template ID of FSAD as a dependency. Don't include the solution template version in the SSA configuration template.
    
    ```yaml
    dependencies:
      - solutionTemplateId: <arm id FSAD solution template id>
        configsToBeInjected:
          - from: DependencyConfig
            to: DependencyConfigs
    ```

1. Set the variables for the FSAD and SSA solution templates, their instances, and the target names. The target names should be unique for each solution template instance.

    ```bash
    fsad="<fsad name>"
    fsad_instance_name="<fsad instance name>"
    ssa="<ssa name>"
    t1="<target for fsad>"
    t2="<target for ssa>"
    rg="rg"
    ```

1. Review the FSAD solution template using the `az workload-orchestration target review` command.

    ```bash 
    az workload-orchestration target review --solution-template-name "$fsad" --solution-template-version 1.0.0 --resource-group "$rg" --target-name "$t1" --solution-instance-name "$fsad_instance_name"
    ```

1. Copy the ID from the output of the FSAD review command and add it to `"solutionVersionId":  "<solution version id>"` in the *dependencies.json* file. 
1. Review the SSA solution template.

    ```bash 
    az workload-orchestration target review --solution-template-name "$ssa" --solution-template-version 1.0.0 --resource-group "$rg" --target-name "$t2" --solution-dependencies "@dependencies.json" 
    ```

1. Publish and install the SSA solution instance.

    ```bash 
    az workload-orchestration target publish \
        --solution-name "$ssa" \
        --solution-version "<copy the name from review output>" \
        --review-id "<copy the reviewId from review output>" \
        --resource-group "$rg" \
        --target-name "$t2"

    az workload-orchestration target install \
        --solution-name "$ssa" \
        --solution-version "<copy the name from review output>" \
        --resource-group "$rg" \
        --target-name "$t2"
    ```

### [PowerShell](#tab/powershell)

1. Before proceeding, ensure that the targets for the shared application (FSAD) and the dependent solution (SSA) are created. Additionally, create solution templates for both FSAD and SSA, ensuring all dependencies are properly configured.

    > [!IMPORTANT]
    > When configuring the SSA solution template, include only the solution template ID of FSAD as a dependency. Don't include the solution template version in the SSA configuration template.
    
    ```yaml
    dependencies:
      - solutionTemplateId: <arm id fsad solution template id>
        configsToBeInjected:
          - from: DependencyConfig
            to: DependencyConfigs
    ```

1. Set the variables for the FSAD and SSA solution templates, their instances, and the target names. The target names should be unique for each solution template instance.

    ```powershell
    $fsad = "<fsad name>"
    $fsad_instance_name = "<fsad instance name>"
    $ssa = "<ssa name>"
    $t1 = "<target for fsad>"
    $t2 = "<target for ssa>"
    $rg = "rg"
    ```

1. Review the FSAD solution template using the `az workload-orchestration target review` command.

    ```powershell
    az workload-orchestration target review --solution-template-name $fsad --solution-template-version 1.0.0 --resource-group $rg --target-name $t1 --solution-instance-name $fsad_instance_name
    ```

1. Copy the ID from the output of the FSAD review command and add it to `"solutionVersionId":  "<solution version id>"` in the *dependencies.json* file. 
1. Review the SSA solution template.

    ```powershell
    az workload-orchestration target review --solution-template-name $ssa --solution-template-version 1.0.0 --resource-group $rg --target-name $t2 --solution-dependencies "@dependencies.json"
    ```

1. Publish and install the SSA solution instance.

    ```powershell
    az workload-orchestration target publish `
        --solution-name $ssa `
        --solution-version "<copy the name from review output>" `
        --review-id "<copy the reviewId from review output>" `
        --resource-group $rg `
        --target-name $t2

    az workload-orchestration target install `
        --solution-name $ssa `
        --solution-version "<copy the name from review output>" `
        --resource-group $rg `
        --target-name $t2
    ```

***

Once installation succeeds, you see both SSA and FSAD solutions deployed on edge and FSAD solution has the configurations injected from SSA.

## Upgrade the FSAD solution instance

### [Bash](#tab/bash)

1. Create a new version of the FSAD solution template. For example, the new version can have new configurations or new helm chart version.
1. Run the review command for FSAD instance with new solution template version.

    ```bash
    az workload-orchestration target review --solution-template-name "$fsad" --solution-template-version "<new version>" --resource-group "$rg" --target-name "$t1" --solution-instance-name "$fsad_instance_name"
    ```

1. Publish and install the FSAD solution instance.

    ```bash
    az workload-orchestration target publish \
            --solution-name "$fsad" \
            --solution-version "<copy the name from review output>" \
            --review-id "<copy the reviewId from review output>" \
            --resource-group "$rg" \
            --target-name "$t1"
    
    az workload-orchestration target install \
            --solution-name "$fsad" \
            --solution-version "<copy the name from review output>" \
            --resource-group "$rg" \
            --target-name "$t1"
    ```

### [PowerShell](#tab/powershell)

1. Create a new version of the FSAD solution template. For example, the new version can have new configurations or new helm chart version.
1. Run the review command for FSAD instance with new solution template version.

    ```powershell
    az workload-orchestration target review --solution-template-name $fsad --solution-template-version "<new version>" --resource-group $rg --target-name $t1 --solution-instance-name $fsad_instance_name
    ```

1. Publish and install the FSAD solution instance.

    ```powershell
    az workload-orchestration target publish `
        --solution-name $fsad `
        --solution-version "<copy the name from review output>" `
        --review-id "<copy the reviewId from review output>" `
        --resource-group $rg `
        --target-name $t1

    az workload-orchestration target install `
        --solution-name $fsad `
        --solution-version "<copy the name from review output>" `
        --resource-group $rg `
        --target-name $t1
    ```

***

Once installation succeeds, the FSAD solution is upgraded with latest changes along with the SSA configurations.