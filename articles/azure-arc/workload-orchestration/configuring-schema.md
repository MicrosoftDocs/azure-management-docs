---
title: Configuration Schemas for Workload Orchestration
description: Learn the rules on how to create configuration schemas for workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: concept-article
ms.date: 06/24/2025
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to create a YAML configuration schema for workload orchestration, so that I can define properties, validation rules, and options for a solution template, ensuring proper structure and consistency during deployment.
---

# Configuration schema for workload orchestration

The configuration schema is a YAML file that defines the structure and rules for the configuration of a solution template. It specifies the properties, types, and validation rules for the configuration values that users can provide when deploying the solution template. A configuration template can refer to a single schema or none.

The schema is used to validate the configuration values provided by users and ensure that they conform to the expected format and constraints. It also provides a way to document the configuration options available for the solution template.

This article describes the structure and rules for creating a configuration schema for workload orchestration.

## Configuration schema structure

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
      editableAt: #array of levels ex: Line, Factory where value can be edited at (optional)
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

## Constraints on rules

The following constraints apply to the rules defined in the configuration schema:

- The `config` key is optional. If a key has a rule, the `type` property is mandatory.
- The `minValue` and `maxValue` properties are only supported for numeric types (`int`, `float`).
- The `minValue` must be greater than the `maxValue`.
- The `allowedValues` property is applicable only for `string`, `int`, and `float` types.
- The `disallowedValues` property is applicable only for `string`, `int`, and `float` types.
- The `defaultValue` property is not applicable for the `object` type.
- The `editableBy` property accepts either IT, OT or both values. If IT is set, this parameter isn't shown on the workload orchestration portal and has to be configured via CLI. If OT is set, this parameter is visible on the workload orchestration portal and can be set via CLI also.
- The `editableAt` parameter accepts any hierarchy level value as defined during context creation, for example, `Line`, `Factory`, or `City`.

## Including and referencing rules from another schema

If you prefer to re-use a particular set of schema rules, this feature enables you to achieve it.

For example, you want to define a *LogSettingsSchema* with version 1.0.0 as below:

```yaml
rules:
    configs:
      LogSettings:
        type: object
        required: true
      LogSettings.LogOutput:
        type: object
        required: true
      LogSettings.LogOutput.EnableConsole:
        type: boolean
        required: true
        editableAt:
        - Line
      SupportedHttpMethods:
        type: array[string]
        required: true
        editableAt:
        - Factory

```

**Scenario 1**

If you prefer to include full content from *LogSettingsSchema*, use the `include` function as defined. Make sure you end with ":" to make the YAML file valid. 
Name the below schema as *SampleSchema* with version 1.0.0:

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
        editableAt:
          - Line

```  

The final schema is generated as follows. The content of *LogSettingsSchema* is included in the final schema:

```yaml
rules:
    configs:
      LogSettings:
        type: object
        required: true
      LogSettings.LogOutput:
        type: object
        required: true
      LogSettings.LogOutput.EnableConsole:
        type: boolean
        required: true
        editableAt:
        - Line
      SupportedHttpMethods:
        type: array[string]
        required: true
        editableAt:
        - Factory
      TempSettings:
        type: object
        required: true
      TempSettings.Threshold:
        type: int
        required: true
        editableAt:
          - Line
```

**Scenario 2**

If you want to include only a specific section or specific nested object and its children, you can do:

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
        editableAt:
          - Line
```

The final schema is generated as follows. Only `LogSettings` and its children are included. *SupportedHttpMethods* is skipped as the syntax included specific object - `LogSettings`.

```yaml
rules:
    configs:
      LogSettings:
        type: object
        required: true
      LogSettings.LogOutput:
        type: object
        required: true
      LogSettings.LogOutput.EnableConsole:
        type: boolean
        required: true
        editableAt:
        - Line
      TempSettings:
        type: object
        required: true
      TempSettings.Threshold:
        type: int
        required: true
        editableAt:
          - Line
```

## Custom validation using expressions

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

**Example 2: Per-Rule Validation with Cross Referencing to other Configs**

Enforce the `frequency` configuration key to have an integer value less than or equal to `frequencyLimit`.

```yaml
rules:
    configs:
      frequency:
        type: int
        expression: ${{ $le($val(frequency), $val(frequencyLimit))}}
      frequencyLimit:
         type: int
```

**Example 3: Per-Schema Validation**

Enforce the `service_ip` configuration key, which is an IP Address, to be in a CIDR range.

```yaml
rules:
    configs:
      service_ip:
        type: string
        expression: ${{ $cidr_contains("192.168.1.0/24", $val(service_ip))}}
```

**Example 4: Per-Schema Validation**

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

## Related content

- [Configuration template](configuring-template.md)
- [RBAC guide](rbac-guide.md)
- [Service groups with workload orchestration](service-group.md)