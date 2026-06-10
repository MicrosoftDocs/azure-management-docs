---
title: Workload orchestration configuration model
description: Understand the configuration model of workload orchestration, including schema, solution templates, and hierarchy configuration templates, and how configuration is applied across environments.
author: nathmanish
ms.author: nathmanish
ms.topic: conceptual
ms.date: 05/10/2026
# Customer intent: As a platform engineer, I want to understand the configuration model of workload orchestration, so that I can define, reuse, and apply configurations consistently across deployments.
---

# Workload orchestration configuration model

The configuration model operates on top of the resource model and focuses on how applications and their configurations are defined and applied. It is composed of three core components:

- **Solution template**: Defines the application and its deployment artifacts
- **Hierarchy configuration template**: Defines how configuration is applied to different hierarchy components
- **Schema**: Defines configurable attributes and validation rules

These components work together to enable consistent, reusable, and scalable configuration across distributed deployments.


## Solution template

A **solution template** defines the application to be deployed along with its configuration model. Once defined, a single solution template can be used to deploy multiple replicas of an application or solution across distributed clusters, with custom dynamic configurations. Solution templates combine:

- Helm charts that define the application to be deployed
- Configurable parameters for the application
- References to one or more schemas (optional)
- Support for application versioning
- Associates capabilities required for deployment

Solution templates are typically defined using:

- A YAML configuration template file defining the application parameters, template details and schema references.
- A JSON specification file specifying deployment artifacts like Helm chart details

<details>
<summary>Learn more</summary>
The building blocks of a solution template help you manage configurations dynamically using different expressions:

- **Variables**: Serve as placeholders for values that can be dynamically configured while deploying the solution.
- **Conditionals**: Execute code based on specified conditions, enabling decision-making in templates.
- **Functions**: Reusable code blocks that perform specific tasks, enhancing modularity and provide a mechanism to incorporate configs from external config files or template promoting reusability.

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

You can also include only a subsection from `LogSettingsConfig`:

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


## Hierarchy configuration template

A **hierarchy configuration template** follows the same design and rules as that of a solution template, except that instead of defining the configuration parameters for a particular solution, it applies them to any target or Site in the hierarchy it is linked to, which can be used as common configurations for any solutions deployed on those targets.

This ensures that:
- Global settings can be defined once and reused
- Site-specific adjustments can be applied where needed


## Schema

A **schema** defines the configurable attributes of a solution and the rules that govern their usage. It is written in YAML and specifies the properties, types, and validation rules for the configuration values that users can provide when deploying a solution or configuring the hierarchy. Schemas are optional and can be reused across multiple solution deployments and hierarchy configurations.


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
      defaultValue: # default value applicable for primitive types i.e. int, string, boolean, float (optional) 
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
- The `allowedValues` property is applicable only for `string`, `int`, and `float` types.
- The `disallowedValues` property is applicable only for `string`, `int`, and `float` types.
- The `defaultValue` property is not applicable for the `object` type.
- The `editableBy` property accepts either IT, OT or both values. If IT is set, this parameter isn't shown on the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com) and has to be configured via CLI. If OT is set, this parameter is visible on the workload orchestration portal and can be set via CLI also.

### Including and referencing rules from another schema

Workload orchestration allows you to reuse a particular set of rules from another schema. For example, if you want to include full content from *LogSettingsSchema* in your current schema, use the `include` function as shown below.

```yaml
rules:
    configs:
      ${{$include(LogSettingsSchema/1.0.0)}}:
      ErrorThreshold:
        type: int
        required: true
```

You can also choose to include only a specific section or specific nested object and its children.

```yaml
rules:
    configs:
      ${{$include(LogSettingsSchema/1.0.0, LogSettings)}}:
      ErrorThreshold:
        type: int
        required: true
```

### Custom validation using expressions

Custom validation is supported using user defined expressions at two levels:

- **Per Rule Validation:** Uses `expression` (optional property) inside each rule. Suitable for validations that are constrained to a single configuration key.
- **Per Schema Validation:** Uses `validations` property that accepts a list of expressions. Suitable for validations that span across multiple configuration keys or the whole schema.

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


## Configuration resolution

At deployment time, workload orchestration resolves configuration values by combining inputs from multiple levels:

- Schema definitions
- Solution template defaults
- Hierarchy configuration values

Configuration is applied hierarchically, with lower levels overriding higher-level values where applicable.


## How the configuration model works together

The configuration model operates as a layered system to achieve these objectives:
- **Declarative configuration**: Define once and apply consistently
- **Separation of definition and application**: Schemas define structure, templates define deployment
- **Hierarchy-aware configuration**: Configuration is resolved based on organizational structure
- **Reusability**: Shared schemas and templates reduce duplication
- **Version control**: Templates support versioned deployments
