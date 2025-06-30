---
title: Configuration Templates for workload orchestration
description: Learn how to create configuration templates for workload orchestration using the templating language.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: concept-article
ms.date: 04/17/2025
ms.custom:
  - build-2025
---

# Configuration template for workload orchestration

A configuration template can refer to a single schema or none. Schema can be referred with its full path, that is, subscription, resource group name, schema name and version, or just with schema name and version. If subscription and resource group name aren't provided, then these details are taken from the request for solution or target creation.

The building blocks of a templating language help you manage configurations dynamically using different expressions:

- **Variables**: Serve as placeholders for values that can be dynamically replaced.
- **Conditionals**: Execute code based on specified conditions, enabling decision-making in templates.
- **Functions**: Reusable code blocks that perform specific tasks, enhancing modularity and provides mechanism to incorporate configs from external config files or template promoting reusability.

This article describes different expressions in the templating language and its usage with examples.

## Expressions

Expressions are defined using `${{<expression>}}` syntax.

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
| `$val(<jsonPath>)`                               | `${{$val(key1.key2.key3)}}`                                                    | Reads the value from current config value of given json path set in different hierarchy. If the value is set in multiple hierarchy levels, lowest level value is taken.                                                                                                                                                                                                                                                                                                                |
| `$config(<config path>, <json path (optional)>)` | `$config(subId/resourceGroupName/commonConfig/1.0.0, key1.key2.key3)` | Reads the value of given json path from given config path set. If the value is set in multiple hierarchy levels, lowest level value is taken.<br><br>Config path can be full path, that is, subscription/resourceGroupName/configName/version or just configName/version, in case of configName/Version. the subscription and resourceGroup is taken from current resolveconfig request context.<br><br>If json path isn't provided, then it reads the entire config from ARM object. |
| `$property(<json path>)`                         | `$property(key1.key2.key3)`                                                    | This is used to read config key within the config template                   |

### *$val*

*$val* is used to read the configurations from either hierarchy context or configuration template. Hierarchy context holds configurations set at different hierarchy levels, like line or factory. For example, let's say the configuration of temperature in Hotmelt solution needs to be set at line level by OT persona. In this case, IT first needs to define the rules in schema:

```yaml
Temperature:
  editableAt:
     - line
  editableBy:
    - OT
```

Then, expression needs to be defined in the configuration template:

```yaml
Temperature: ${{$val(Temperature)}}
```

The *$val* expression looks for value for `Temperature` set at line level. If `editableAt` has multiple levels, then top down search based on order defined in hierarchy list is done. The child level always overrides parent level value.

## String Operations

Concatenation: string + `<expression>`

### Example

- ApplicationName: Health Check Monitor For + ${{$val(FactoryName)}} + Factory
- Output: Health Check Monitor For Contoso Factory

## Handling Null Values

If key is having the null value, parser checks whether schema rule has `defaultValue` defined for this key. If `defaultValue` is present, then it sets the `defaultValue` to the key. If `required` is set true in the schema rule, it throws the null value error. Else it sets the null value to key.

## Referencing ARM properties of target resource

Any ARM properties of target resource can be used in the configuration template.

For example, if you prefer to insert the name or ID of the target in the template, it can be done using the following syntax:

```yaml
TargetName= ${{$target(name)}}
TargetArmId = ${{$target(id)}}
```
 
## Nested Expression

Expression can be nested, for example ${{$if($eq($val(key1), value), equal, notEqual)}}. In this case, the expression is evaluated from inside out. The inner expression is evaluated first, $val(key1), then the outer expression $eq($val(key1), value) is evaluated, and finally the outermost expression *$if($eq($val(key1), value), equal, notEqual) is evaluated.

### Example 

The application Contoso has the following configuration template:

```yaml
schema:
    subscription: contoso_sub_id
    resourceGroupName: contoso
    name: commonSchema
    version: 1.0.0
```

```yaml
configs:
    SqlServerEndpoint: ${{$config(/subscriptions/contoso_sub_id/resourceGroups/contoso/providers/Microsoft.Edge/configurations/contosonConfig/dynamicConfigurations/contosoconfig/versions/1.0.0, SqlServerEndpoint)}}
    AppName: ${{$val(AppName)}}
    LineTag: ${{$val(LineTag)}}
    Temperature: ${{$val(Temperature)}}
    LightColor: ${{$if($gt($val(Temperature), 50),RED,GREEN)}}
```

The schema for the configuration template is as follows:

```yaml
rules:
    AppName:
        type: string
        required: true
        pattern: ^\\w+$
        editable_at:
        - Factory
        editable_by:
        - OT
        description: Name of the App
        placeHolder: Contoso
        label: Application Name
        disabled: false
    LineTag:
        type: string
        required: true
        pattern: ^\\w+$
        editable_at:
        - Factory
        editable_by:
        - OT
        description: Line Tag
        placeHolder: Contoso line
        label: Line Tag
        disabled: false
    Temperature:
        type: int
        required: true
        min_value: 1
        max_value: 120
        editable_at:
        - Line
        editable_by:
        - OT
        description: Temperature of the system
        placeHolder: In degrees
        label: System Temperature
        disabled: false
```

Rules for `SqlServerEndpoint` and `LightColor` aren't defined, because `SqlServerEndpoint` referred from common configuration and `LightColor` is variable derived from the expression.

Configuration template for common configuration:

```yaml

schema:
    subscription: contoso_sub_id
    resourceGroupName: contoso
    name: commonSchema
    version: 1.0.0
configs:
    SqlServerEndpoint: ${{$val(SqlServerEndpoint)}}
```

Schema for common configuration:

```yaml
rules:
    SqlServerEndpoint:
        type: string
        required: true
        editable_at:
        - Factory
        editable_by:
        - IT
        description: Sql Server Endpoint
        placeHolder: Sql Server
        label: Sql Server Endpoint
        disabled: false
```

## Dependencies

Dependencies define the dependency details of a solution. The dependencies section is only supported for a configuration template used within solution template, and it's not supported for configuration template alone. 

If `solutionTemplateVersion` isn't provided, the service looks for latest available solution template version.

```yaml

schema: # (OPTIONAL)
    subscription: # optional
    resourceGroupName: #optional
    name: #schema name
    version: #schema version
dependencies:
    solutionTemplateId: <arm Id of solution template>
    solutionTemplateVersion: <solution template version> #optional
    configsToBeInjected:  #optional
      - from: <config path to be copied from current config template>
        to: <config path to be injected to shared application config>                 
configs:
    key: value # ex: temperature: 100
```

## Including and referencing other configurations

If you prefer to reuse same configuration across multiple solutions or other configuration templates, you can do it using the `include` function. 

Let's say you have the following configuration template `LogSettingsConfig` with version 1.0.0 and it references `SampleSchema` which is defined as:

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
     LogSettings:
        LogOutput:
            EnableConsole: ${{$val(LogSettings.LogOutput.EnableConsole)}}
            EnableCache: false
    SupportedHttpMethods: ${{$val(SupportedHttpMethods)}}

```

### Example 1

You include full `LogSettings` in `SampleConfig`:

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
    TempSettings:
        Threshold: ${{$val(TempSettings.Threshold)}}             
    ${{$include(LogSettingsConfig/1.0.0)}}:

```  

The resulting configuration template is generated as follows:

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
    TempSettings:
        Threshold: ${{$val(TempSettings.Threshold)}}             
    LogSettings:
        LogOutput:
            EnableConsole: ${{$val(LogSettings.LogOutput.EnableConsole)}}
            EnableCache: false
    SupportedHttpMethods: ${{$val(SupportedHttpMethods)}}

```

### Example 2

You include only the subsection from `LogsettingsConfig`:

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
    TempSettings:
        Threshold: ${{$val(TempSettings.Threshold)}}             
    ${{$include(LogSettingsConfig/1.0.0, LogSettings)}}:

```

The final configuration template is generated as follows. Only `LogSettings` and its children are included. `SupportedHttpMethods` is skipped as the syntax included specific object, `LogSettings`.

```yaml
schema:
    name: "SampleSchema"
    version: "1.0.0"
configs:
    TempSettings:
        Threshold: ${{$val(TempSettings.Threshold)}}             
    LogSettings:
        LogOutput:
            EnableConsole: ${{$val(LogSettings.LogOutput.EnableConsole)}}
            EnableCache: false

```

## Related content

- [Configuration schema](configuring-schema.md)
- [RBAC guide](rbac-guide.md)
- [Service groups with workload orchestration](service-group.md)