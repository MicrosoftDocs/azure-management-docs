---
title: CLI reference for `azcmagent show`
description: Syntax for the `azcmagent show` command line tool
ms.topic: reference
ms.date: 11/19/2025
# Customer intent: As a system administrator, I want to use the command line to check the status of the Azure Connected Machine agent, so that I can ensure proper connectivity and monitor the health of the services.
---

# `azcmagent show`

Displays the current state of the Azure Connected Machine agent, including whether or not it's connected to Azure, the Azure resource information, and the status of dependent services.

> [!NOTE]
> You don't need administrator privileges to run `azcmagent show`.

## Usage

```
azcmagent show [property1] [property2] ... [propertyN] [flags]
```

## Examples

Check the status of the agent:

```
azcmagent show
```

Check the status of the agent and save it in a JSON file in the current directory:

```
azcmagent show -j > "agent-status.json"
```

Show only the agent status and last heartbeat time (using display names):

```
azcmagent show "Agent Status" "Agent Last Heartbeat"
```

Show only the agent status and last heartbeat time (using JSON keys):

```
azcmagent show status lastHeartbeat
```

## Flags

This command supports the flags described in [Common flags](azcmagent.md#common-flags) and the flags listed in this section.

`[property]`

The name of a property to include in the output. To show more than one property, separate each property by spaces. You can use either the display name or the JSON key name to specify a property. For display names with spaces, enclose the property in quotes.

`--os`

Outputs additional information about the operating system.
