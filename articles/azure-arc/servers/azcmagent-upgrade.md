---
title: CLI reference for `azcmagent upgrade`
description: Syntax for the `azcmagent upgrade` command line tool
ms.topic: reference
ms.date: 03/02/2026
# Customer intent: "As a system administrator, I want to upgrade the Azure Connected Machine agent using the command line, so that I can keep my servers up-to-date with the latest features and security patches."
---

# `azcmagent upgrade`

Upgrades the Azure Connected Machine agent to the latest version, or to a specified version.

> [!NOTE]
> This command is only supported for machines that are already running version 1.62 or later of the Azure Connected Machine agent. If your machine is running an older version, follow the [manual upgrade instructions](manage-agent.md#upgrade-the-agent).

## Usage

```
azcmagent upgrade [flags]
```

## Examples

Upgrade the agent to the latest version:

```
azcmagent upgrade
```

Upgrade the agent to a specific version:

```
azcmagent upgrade --version 1.63
```

## Flags

This command supports the flags described in [Common flags](azcmagent.md#common-flags) and the flags listed in this section.

`--version`

Specifies a specific version to upgrade the agent to. If not provided, the agent will be upgraded to the latest available version. Supported format is `MAJOR.MINOR`, for example `1.63`.

Supported values:

`--cloud`

Specifies the Azure cloud instance. Only required when running the command on a disconnected machine.

Supported values:

* `AzureCloud`
* `AzureUSGovernment`
* `AzureChinaCloud`

`--background`

Allows the upgrade process to run without popups. This is useful for running the command in a script or on a schedule. If not provided, the upgrade process will display popups to show progress and any errors that occur.