---
title: Create a Basic Solution with Workload Orchestration
description: Learn how to create a basic solution without common configurations using the workload orchestration via CLI. 
author: nathmanish
ms.author: nathmanish
ms.topic: quickstart
ms.date: 09/05/2025
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to create a basic solution using workload orchestration via CLI without common configurations, so that I can deploy applications efficiently with minimal setup.
---

# Create a basic solution 

In this guide, you create a basic solution using workload orchestration via CLI. 

## Prerequisites

- Set up the required resources for workload orchestration. If you haven't, refer to [Set up workload orchestration](setup-wo.md).
- Download the artifacts from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). 

  [![Download](https://img.shields.io/badge/Download%20zip%20file-0078D4?style=flat&labelColor=0078D4)](https://github.com/Azure/workload-orchestration/archive/refs/heads/main.zip)


## Define the variables

### [Bash](#tab/bash)

```bash
# Set environment variables
subId="<SUBSCRIPTION_ID>"
rg="<RESOURCE_GROUP_NAME>"
l="<LOCATION>"
contextName="<CONTEXT_NAME>"
contextId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/context/$contextName"
siteName="<SITE_NAME>"
siteId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$sitename"
childName="<TARGET_NAME>"

# Create variables for schema
schemaName="Sharedschema"
schemaFile="shared-schema.yaml"
# Enter value in x.x.x format
schemaVersion="1.0.0"

# Create variables for application/solution
# Enter name of application
appName="priceDetector"
# Enter value in x.x.x format
appVersion="1.0.0"
# Enter description for application
desc="To calculate total by detecting price of each item"
# Enter capabilities of application
appCapList1="[soap,conditioner]"
# Enter configuration template file name for the application 
appConfig="app-config-template.yaml"

helmurl="enter helm url of app e.g. contosocm.azurecr.io/helm/app"
chartVersion="enter in this format x.x.x e.g. 0.8.0"
```

### [PowerShell](#tab/powershell)

```powershell
# Set environment variables
$subId="<SUBSCRIPTION_ID>"
$rg="<RESOURCE_GROUP_NAME>"
$l="<LOCATION>"
$contextName="<CONTEXT_NAME>"
$contextId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/context/$contextName"
$siteName="<SITE_NAME>"
$siteId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$sitename"
$childName="<TARGET_NAME>"

# Create variables for schema
$schemaName = "Sharedschema"
$schemaFile = "shared-schema.yaml"
# Enter value in x.x.x format
$schemaVersion = "1.0.0"

# Create variables for application/solution
# Enter name of application
$appName = "priceDetector"
# Enter value in x.x.x format
$appVersion = "1.0.0"
# Enter description for application
$desc = "To calculate total by detecting price of each item"
# Enter capabilities of application
$appCapList1 = "[soap,conditioner]"
# Enter configuration template file name for the application 
$appConfig = "app-config-template.yaml"

$helmurl = "enter helm url of app e.g. contosocm.azurecr.io/helm/app"
$chartVersion = "enter in this format x.x.x e.g. 0.8.0"
```

***

## Create a configuration schema

Configuration schema is a YAML file that defines the structure and rules for the configuration of a solution template. It specifies the properties, types, and validation rules for the configuration values that users can provide when deploying a solution template or configuring the hierarchy. A template can refer to a single schema or none.

<details>
<summary>Learn more</summary>

### Configuration schema structure

The configuration schema consists of two main sections: `rules` and `validations`. The `rules` section defines the configuration keys and their properties, while the `validations` section defines any custom validation rules that apply to the entire schema.

Here is an example of a configuration schema:

```yaml

rules:
  configs:
    key: # name of the configuration key. this can be a string
      type: #string|boolean|int|float|object|array[int|string|float|object]  
      required: # boolean  
      pattern: # regex pattern (optional)
      allowedValues: #array applicable for string, int, float (optional)
      disallowedValues: #array applicable for string, int, float (optional)
      expression: # any valid expression. should be wrapped in expression template like: "${{ expression }}"
      editableBy: # array of personas Ex: IT, OT who can edit the value. (optional)
      minValue: # minimum number  (optional)
      maxValue: # maximum number (optional)
      defaultValue: # default value applicable for primitive types i.e. int, string, boolen, float (optional) 
      description: # string (optional)
      placeHolder: # place holder value applicable for string, int, float, boolean. (optional)
      label: # string to be displayed on the UI (optional)
      disabled: # boolean disable for edit (optional)
  validations:
    # one or more validation expressions are supported
    - expression: # any valid expression. should be wrapped in expression template like: "${{ expression }}"
    - expression: # any valid expression. should be wrapped in expression template like: "${{ expression }}"
    
```

### Constraints on rules

The following constraints apply to the rules defined in the configuration schema:

- The `config` key is optional. If a key has a rule, the `type` property is mandatory.
- The `minValue` and `maxValue` properties are only supported for numeric types (`int`, `float`).
- The `minValue` must be greater than the `maxValue`.
- The `allowedValues` property is applicable only for `string`, `int`, and `float` types.
- The `disallowedValues` property is applicable only for `string`, `int`, and `float` types.
- The `defaultValue` property is not applicable for the `object` type.
- The `editableBy` property accepts either IT, OT or both values. If IT is set, this parameter isn't shown on the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com) and has to be configured via CLI. If OT is set, this parameter is visible on the workload orchestration portal and can be set via CLI also.

### Including and referencing rules from another schema

Workload orchestration allows you to re-use a particular set of rules from another schema. For example, if you want to include full content from *LogSettingsSchema* in your current schema, use the `include` function as defined and make sure you end with ":" to make the YAML file valid.

```yaml
rules:
    configs:
      ${{$include(LogSettingsSchema/1.0.0)}}:
      TempSettings:
        type: object
        required: true
      TempSettings.Threshold:
        type: int
        required: true
```

You can also choose to include only a specific section or specific nested object and its children.

```yaml
rules:
    configs:
      ${{$include(LogSettingsSchema/1.0.0, LogSettings)}}:
      TempSettings:
        type: object
        required: true
      TempSettings.Threshold:
        type: int
        required: true
```

### Custom validation using expressions

Custom validation is supported using user defined expressions at two levels:

1. **Per Rule Validation:** Using expression (optional property) inside each rule.
   - Suitable for validations that are constrained to a single configuration key.
1. **Per Schema Validation:** Using "validations" property that accepts a list of expressions.
   - Suitable for validations that span across multiple configuration keys.
   - Suitable for validating the generated config as a single unit.

These expressions need to return true (as boolean or string) to indicate success in validation.

**Example 1: Per-Rule Validation**

Enforce the `frequency` configuration key to have an integer value less than or equal to 120.

```yaml
rules:
    configs:
      frequency:
        type: int
        expression: ${{ $le($val(frequency)), 120}}
```

**Example 2: Per-Schema Validation**

Enforce that at least one of `OptionA`, `OptionB`, or `OptionC` must be `false`.

```yaml
rules:
    configs:
      OptionA:
        type: boolean
        expression: ${{ $le($val(frequency), $val(frequencyLimit))}}
      OptionB:
        type: boolean
      OptionC:
        type: boolean

validations:
   expression: ${{ $any($not($val(OptionA)), $not($val(OptionB)), $not($val(OptionC)))}}
```

</details>


Create the schema file by referring to *shared-schema.yaml* from [GitHub repository](https://github.com/Azure/workload-orchestration).
   
```azurecli
az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --version "$schemaVersion" --schema-file "$schemaFile"
```

You can provide version on file instead of as a CLI argument. Add below section to the *shared-schema.yaml* file.

```yaml
metadata:
    name: <name> [optional]
    version: <version> [optional]
```

Run the same CLI command without `--version argument`. The service takes version input from file.

```azurecli
az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --schema-file "$schemaFile"
```

The name field is introduced for user to identify the resource name and its version the file refers to. If name is provided, then it should match `--schema-name` argument.

> [!TIP]
> You can view the created schema using `az workload-orchestration schema version show --resource-group "$rg" --schema-name "$schemaName" --version "$schemaVersion"`

## Create the solution template 

Solution template defines the Helm solution and configurable parameters for your application.

<details>
<summary>Learn more</summary>
The building blocks of a solution template help you manage configurations dynamically using different expressions:

- **Variables**: Serve as placeholders for values that can be dynamically configured while deploying the solution.
- **Conditionals**: Execute code based on specified conditions, enabling decision-making in templates.
- **Functions**: Reusable code blocks that perform specific tasks, enhancing modularity and provides mechanism to incorporate configs from external config files or template promoting reusability.

### Conditional Expressions

| Expression        | Example                | Behavior                                |
|-----------------|-----------------------------------------|--------------------------------------------------------|
| `$between(<value>, <value1>, <value2>)` | `$between(20, 10,100)`       | true if `<value>` is between `<value1>` and `<value2>`     |
| `$eq(<value1>, <value2>)`    | `$eq(20, 20)`                                 | true if `<value1>` equals `<value2>`                                     |
| `$ge(<value1>, <value2>)`               | `$ge(100, 90)`                                | true if `<value1>` is greater or equal to `<value2>`                     |
| `$gt(<value1>, <value2>)`               | `$gt(90, 60)`                                 | true if `<value1>` is greater than `<value2>`                            |
| `$le(<value1>, <value2>)`               | `$le(5, 10)`                                  | true if `<value1>` is less or equal to `<value2>`                        |
| `$lt(<value1>, <value2>)`               | `$lt(10, 20)`                                 | true if `<value1>` is less than `<value2>`                               |
| `$if(<condition>)`                      | `${{$if($eq(value,value), equal, notEqual)}}` | true if `<condition>` evaluates to `true` (boolean) or `"true"` (string) |

### Functions 

| Expression        | Example                     | Behavior       |
|----------------------------|---------------------|--------------------------------------------------|
| `$val(<jsonPath>)`                               | `${{$val(key1.key2.key3)}}`                                                    | Reads the value of given json path from given config path set. This allows user to set dynamic values for a configuration parameter defined in the template, at the time of solution deployment.                                                                                                                                                                                                                                                                                                                |
| `$config(<config path>, <json path (optional)>)` | `$config(subId/resourceGroupName/commonConfig/1.0.0, key1.key2.key3)` | Reads the value of given json path from given config path set. Config path can be full path, that is, subscription/resourceGroupName/configName/version or just configName/version, in which case the subscription and resourceGroup is taken from current resolveconfig request context. If json path isn't provided, then it reads the entire config from ARM object. |
| `$property(<json path>)`                         | `$property(key1.key2.key3)`                                                    | This is used to read config key within the config template                   |

### String concatenation

You can perform a concatenation using `<string>` + `<expression>`. For example,

- ApplicationName: Health Check Monitor For + ${{$val(FactoryName)}} + Factory
- Output: Health Check Monitor For Contoso Factory

### Handling Null Values

If key in the template is having the null value, parser checks whether schema rule has `defaultValue` defined for this key. If `defaultValue` is present, then it sets the `defaultValue` to the key. If `required` is set true in the schema rule, it throws the null value error. Else it sets the null value to key.

### Referencing ARM properties of target resource

Any ARM properties of target resource can be used in the configuration template, using the following syntax:

```yaml
TargetName= ${{$target(name)}}
TargetArmId = ${{$target(id)}}
```

### Nested Expression

Expression can be nested, for example ${{$if($eq($val(key1), value), equal, notEqual)}}. In this case, the expression is evaluated from inside out.

### Including and referencing other configurations

If you prefer to reuse same configuration across multiple solutions or other configuration templates, you can do it using the `include` function. Let's say you have a configuration template `LogSettingsConfig`. You can choose to include its full set of configurations in your template as:

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
    TempSettings:
        Threshold: ${{$val(TempSettings.Threshold)}}             
    ${{$include(LogSettingsConfig/1.0.0)}}:
```  

You can also include only a subsection from `LogsettingsConfig`:

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
    TempSettings:
        Threshold: ${{$val(TempSettings.Threshold)}}             
    ${{$include(LogSettingsConfig/1.0.0, LogSettings)}}:

```

</details>

Follow these steps to create a solution template for your application.

1. Create the *specs.json* and *app-config-template.yaml* files by referring to sample files from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). In *specs.json*, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*. Update the *app-config-template.yaml* file with proper reference to your schema which you created in the above step.

1. Create the solution template resource.

    ```azurecli
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName" --description "$desc" --capabilities "$appCapList1" --configuration-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

    Values for `--solution-template-name` and `--version` can be provided in the solution template file instead of as CLI arguments. If name or version are specified in both file and CLI, the values should match. Add the following section to the *app-config-template.yaml* file:

    ```yaml
    metadata:
      name: <name> [optional]
      version: <version> [optional]
    ```

    > [!NOTE]
    > The list of capabilities for a solution template should be a subset of that of the targets the solution is intended to be deployed to. To update the list of capabilities for an existing solution template, run `az workload-orchestration solution-template update-capabilities -n "$appName" --capabilities "<capability 1>" "<capability 2>" --description "$desc" --location $l -g $rg`.


## Deploy the solution

Run the following command to configure the solution template and deploy the corresponding solution/application to your target. The configuration values need to be stored in **config.yaml**.

```azurecli
az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-template-name "$appName" --solution-template-version $appVersion --configuration “config.yaml”   
```

> [!NOTE]
> If your template resides in a different resource group, you can use the `--solution-template-rg` argument to specify your template resource group.

You can also choose to configure and deploy the solution in separate individual steps:

1. Set the configuration values for the solution.

    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName" --template-name "$appName" --version $appVersion --solution
    ```

    > [!NOTE]
    > To view the configuration values you have set, use `az workload-orchestration configuration show` with the same set of arguments.

    > [!TIP]
    > You can use the `--template-subscription` argument to set or show configurations for a template residing in an Azure subscription other than the current subscription.

1. Review the configurations for a particular target. This step ensures the configured values obey all schema rules and generates a solution version based on the solution template.

    ```azurecli
    az workload-orchestration target review --resource-group "$rg" --target-name "$childName" --solution-template-version-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName/versions/$appVersion"
    ```

1. Run `target publish` to publish the solution. Enter `reviewId` from the previous command response.

    ```azurecli
    reviewId="<reviewId>"
    az workload-orchestration target publish --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName/versions/$appVersion
    ```

    Completion of this step generates the final configuration of the solution after it's validated and approved, created by combining the schema, configuration template, and solution Helm chart. It represents a fully rendered, a pre-deployment ready, targeted solution.

1. Run the `target install` command to deploy the solution.

    ```azurecli
    az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName/versions/$appVersion
    ```

  > [!TIP]
  > You can also deploy the solution using the [Deploy tab in Workload orchestration portal](deploy.md)

## Next steps

Once you know how to create a basic solution, you can explore more advanced scenarios. For example, check out how to [Create a basic solution with common configurations](solution-with-common-configuration.md), which is an extension of this guide.

