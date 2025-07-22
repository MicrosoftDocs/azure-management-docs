---
title: Create a Solution with Multiple Instances with Workload Orchestration
description: Learn how to create a solution with multiple instances in the same Kubernetes namespace using workload orchestration via CLI.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/18/2025
ms.custom:
  - build-2025
# Customer intent: As a DevOps engineer, I want to create a solution with multiple instances in a Kubernetes namespace using workload orchestration via CLI, so that I can efficiently manage and deploy applications with dynamic configurations.
---

# Create a solution with multiple instances in the same Kubernetes namespace

In this article, you create a solution with multiple instances in the same Kubernetes namespace using workload orchestration via CLI. 

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).


## Create the target

To create the targets, use the `solution-scope` argument in the CLI, which maps to the Kubernetes namespace. Make sure that all targets use the same `solution-scope` so they are mapped to the same namespace. When authoring the Helm chart, make the deployment name in the `deployment.yaml` dynamic. This ensures the deployment name is unique and can be specified at runtime.

For example, in the `deployment.yaml` file, you can use the following code to make the deployment name dynamic:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fsad-{{ .Values.DeploymentName }}
spec:
  replicas: 1
  selector:
    matchLabels:
      name: fsad
  template:
    metadata:
      labels:
        name: fsad
      annotations:
        my-config-checksum: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      nodeSelector:
        kubernetes.io/os: "linux"
      containers:
        - name: fsad
          image: contosocm.azurecr.io/sharedsyncadapter:latest
          volumeMounts:
            - mountPath: /app/appsettings.yaml
              subPath: appsettings.yaml
              name: config
              readOnly: true
          ports:
            - containerPort: 80
      volumes:
         - name: config
           configMap:
            name: fsad-config-{{ .Values.DeploymentName }}

```

Here, deployment name is being taken from *values.yaml* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip). 

Similarly, if `configmap` is used in the application, then the name of the `configmap` is also dynamic.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fsad-config-{{ .Values.DeploymentName }}
```

You define the `DeploymentName` in the configuration template and in schema as configurable, so that it can be set during the deployment. The configuration template and schema are defined in the *app-config-template.yaml* and *app-config-schema.yaml* files respectively. The configuration template file contains the configuration schema and template for the solution.

The following is the configuration template for the `DeploymentName` in the *app-config-template.yaml* file:

```yaml
schema:
  name: <schema-name>
  version: 1.0.0  
configs:
  DeploymentName: ${{$val(DeploymentName)}}

```

The following is the configuration schema for the `DeploymentName` in the *app-config-schema.yaml* file:

```yaml
rules:
  configs:
    DeploymentName:
      type: string
      required: true
      editableAt:
          - line
      editableBy:
          - ot
      pattern: "^[a-z0-9]([-a-z0-9]*[a-z0-9])?$"  #K8s object naming convention        

```

> [!NOTE]
> Check out [Configuration template](configuring-template.md) and [Configuration schema](configuring-schema.md) for the list of rules used to define the template and schema for configurations and details to write conditional or nested expressions in the schema for custom validations.

In the *fsad-specs.json* file, refer the component name to `DeploymentName`:

```json
{
    "components": [
        {
            "name": "${{$property(DeploymentName)}}",
            "type": "helm.v3",
            "properties": {
                "chart": {
                    "repo": "<repo uri>",
                    "version": "<version>",
                    "wait": true,
                    "timeout": "5m"
                }
            }
        }
    ]
}
```

Based on the hierarchy, set `DeploymentName` using the `config set` command as described in [Create a solution with multiple shared adapter dependencies](quickstart-solution-shared-adapter-dependency.md#set-the-configuration-values-for-the-solution), or via the [Configure tab in workload orchestration portal](configure.md).
