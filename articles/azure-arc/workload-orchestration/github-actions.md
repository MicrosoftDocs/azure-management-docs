---
title: GitHub Actions for Workload Orchestration
description: Learn how to use GitHub actions to automate workload orchestration solution templates and schemas.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/24/2025
ms.custom:
  - build-2025
---

# Use GitHub actions to automate workload orchestration

GitHub actions facilitates creating workflows to enable managing workload orchestration solution templates and schemas. This workflow automates the process of deploying workload orchestration configurations for multiple applications and common resources. 

## Architecture overview

The following diagram illustrates the architecture of the GitHub action that automates workload orchestration. The action is triggered by a push to the .pg/apps directory in the repository. It then processes each application and common resource, generating the necessary configuration files and deploying them to Azure Arc.

:::image type="content" source="./media/github-actions-architecture.png" alt-text="Diagram illustrating the architecture of the GitHub action that automates workload orchestration." lightbox="./media/github-actions-architecture.png":::

## Environment setup

The workflow requires the following environment variables:

- `AZURE_RESOURCE_GROUP`: The name of the resource group for deployments with workload orchestration.
- `AZURE_LOCATION`: Azure region where the resource group is located, defaults to `eastus2`.
- `AZURE_SUBSCRIPTION_ID`: The ID of the Azure subscription where the resource group is located.
- `AZURE_TENANT_ID`: The ID of the Azure Active Directory tenant.    
- `AZURE_CLIENT_ID`: The client ID of the Azure Active Directory application.

## Required files

Each application under the `.pg/apps` directory requires these files in its workload-orchestration directory: 

- *schema.yaml: Defines the schema for workload orchestration.
- *-sol-template.yaml: Contains the solution template configuration.
- *specs.json: Specifies the deployment specifications.
- metadata.yaml: Contains metadata like capabilities and external validation settings. 

The common resources under `.pg/apps/common` include: 

- common-schema.yaml: Common schema shared across applications. 
- common-config-template.yaml: Common configuration template. 

> [!NOTE]
> The /* character in filenames indicates a variable prefix unique to each application. For example, if the application is named `testapp`, the files would be named `testapp-schema.yaml`, `testapp-sol-template.yaml`, `testapp-specs.json`, and `metadata.yaml`.

The following diagram illustrates the file structure. 

:::image type="content" source="./media/github-actions-file-structure.png" alt-text="Diagram illustrating the file structure of the GitHub action that automates workload orchestration." lightbox="./media/github-actions-file-structure.png":::


## Push and manual trigger comparison

The following table compares the features and behaviors of the push trigger and manual trigger for the GitHub action.

| Feature      | Push Trigger | Manual Trigger |
|--------------|--------------|---------------|
| When     | Activates on push to main branch | Manually triggered via GitHub Actions UI |
| Files    | - App changes: Processes changed files in `.pg/apps/<app>/workload-orchestration/**`<br> - Common changes: Processes changed files in `.pg/apps/common/*` | Processes all files for specified apps/resources |
| App Detection | Automatically detects apps from changed file paths (excludes common) | User provides comma-separated list (for example, `testapp1,testapp2`) |
| Actions* | - Creates/updates schema if schema file changed<br>- Creates/updates template if template, specs, or metadata file changed<br>- Creates/updates common resources if common files changed | - User chooses which common resources to deploy: none, schema-only, config-only, or all<br>- User chooses app actions: none, create-schema, create-sol-template, or all |
| Validation | Directory existence checked automatically | Validates each directory exists before processing |
| Parallel | Processes all detected apps concurrently (max 4) | Processes all specified apps concurrently (max 4) | 

## Job skipping conditions 

The deploy-apps job is skipped when no apps are detected or provided, or when `action="none"` is selected in manual trigger.

The deploy-common-resources job is skipped when `deploy_common="none"` is selected in manual trigger, or when no common resource files have changed in push trigger.


## Deployment Process 

### 1. Resource detection and validation 

The deployment process begins with detecting and validating the resources that need to be deployed. This is done through two main triggers: push and manual. 

#### Push Event:

1. Detects application names by analyzing changed file paths under `.pg/apps/$app/workload-orchestration/`.
1. Identifies updates to common resources by checking for changes in `.pg/apps/common/*`.
1. Automatically validates YAML files using `yq` to ensure correctness.
1. Compiles lists of applications and common resources that require deployment.
1. Determines required actions based on the specific files that were modified.
1. Logs details of changed and validated files for traceability.

#### Manual Event: 

1. Validates the list of applications provided by the user against the existing directory structure.
1. Determines which application components to deploy based on the selected app action type.
1. Determines which common resource components to deploy based on the selected common resource action.
1. Processes all files for the specified applications and common resources, regardless of whether changes have occurred.

### 2. Azure login 

The deployment process requires authentication with Azure to manage resources. This is done using the Azure CLI and OpenID Connect (OIDC) for secure access.

1. Authenticates with Azure using OIDC.
1. Requires these secrets in repository settings: 

    - AZURE_CLIENT_ID: Azure AD application (service principal) client ID.
    - AZURE_TENANT_ID: Azure AD tenant ID where the service principal is registered. 
    - AZURE_SUBSCRIPTION_ID: Target Azure subscription ID.

1. Uses Azure CLI for authentication and resource management 

### 3. File processing 

The file processing step is crucial for deploying workload orchestration configurations. It involves reading, validating, and creating schemas and solution templates based on the files found in the specified directories.

#### File processing in applications

For each app to deploy, the following steps are performed: 

1. Get files: 

    - Locates required files in `.pg/apps/$app/workload-orchestration/` directory. 
    - Required patterns: *schema.yaml, *-sol-template.yaml, *specs.json, and metadata.yaml 

1. Schema creation: 

    1. Checks for existing schema version first: 
    
        ```powershell
        az workload-orchestration schema version show -g <resource-group> --schema-name <name> -n <version> 
        ```
    
    1. Creates new schema only if version doesn't exist: 
    
        ```powershell
        az workload-orchestration schema create -g <resource-group> -l <location> --schema-file <path>
        ```

    1. Schema name and version are extracted from metadata section in schema file. 

1. Solution template creation: 

    1. Extracts metadata from files: 
    
        - Schema name and version from template file.
        - Template name and version from template file. 
        - Description from metadata file. 
        - Capabilities from metadata file or input. 
        - External validation settings from metadata file. 
    
    1. Verifies required schema version exists: 
    
        ```powershell
        az workload-orchestration schema version show -g <resource-group> --schema-name <name> -n <version>
        ```

    1. Checks if template version exists: 
    
        ```powershell
        az workload-orchestration solution-template version show -g <resource-group> --solution-template-name <name> -n <version>
        ```
    
    1. Creates new template only if version doesn't exist: 
    
    ```powershell
    az workload-orchestration solution-template create -g <resource-group> -l <location> --capabilities <from-metadata-or-input> --configuration-template-file <template-path> --specification <specs-path> --description <description> --enable-external-validation <true/false>
    ```

> [!NOTE]
> - Ensure capabilities are provided either in `metadata.yaml` file or as input.
> - Schema version must exist before template creation.
> - Templates are versioned - only creates if version doesn't exist.

#### File processing in common resources 

For common resources, the process is similar but focuses on the common schema and configuration template files:

1. Schema creation:

    1. Checks for existing schema version first: 
    
        ```powershell
        az workload-orchestration schema version show -g <resource-group> --schema-name <name> -n <version>
        ```

    1. Creates new schema only if version doesn't exist: 
    
        ```powershell
        az workload-orchestration schema create -g <resource-group> -l <location> --schema-file .pg/apps/common/common-schema.yaml 
        ```

    1. Schema name and version are extracted from metadata section in schema file. 

1. Config Template Creation: 

    1. Extracts metadata from template file: 

        - Schema name and version 
        - Template name (defaults to "common-config" if not found) 
        - Template version (must be provided in file or as input) 

    1. Verifies required schema version exists:
    
        ```powershell
        az workload-orchestration schema version show -g <resource-group> --schema-name <name> -n <version>
        ``` 

    1. Checks if config template version exists: 
    
        ```powershell
        az workload-orchestration config-template version show -g <resource-group> --config-template-name <name> -n <version>
        ```

    1. Creates new config template only if version doesn't exist: 
    
        ```powershell
        az workload-orchestration config-template create -g <resource-group> -l <location> --config-template-name <name> --config-template-file <path> --version <version> --description <description> 
        ```

## Error handling 

### Schema version check

- Checks for the existence of the required schema version before creating solution or configuration templates.
- Stops deployment and logs an error if the required schema version is not found.

### File validation

- Checks for the existence of required files before processing.
- Logs a warning if any expected file is missing.
- Continues processing with the available files to avoid blocking the workflow.

### Capability validation 

- Ensure that capabilities are specified either in the `metadata.yaml` file or provided as input.
- The deployment process fails if capabilities are missing.
    
## Best practices

The following best practices help maintain a clean and efficient workflow orchestration process.

### File organization 

- Keep all app files in `.pg/apps/<app>/workload-orchestration/` directory.
- Keep all common files in `.pg/apps/common/` directory.
- Use consistent naming patterns.
- Maintain `metadata.yaml` with current capabilities.

### Version management 

- Always update the schema version before updating the template version.
- Keep schema and template versions in sync.
- Consider dependencies between common and app resources. 

### Manual triggers 

- Use for selective deployment of specific components.
- Helpful for redeploying configurations without changes.
- Choose appropriate action types for both apps and common resources.


 