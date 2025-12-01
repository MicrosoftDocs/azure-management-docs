---
title: CLI reference for `azcmagent extension`
description: Syntax for the `azcmagent extension` command line tool
ms.topic: reference
ms.date: 12/01/2025
# Customer intent: "As a system administrator managing Azure Arc extensions, I want to use the command-line interface to list and uninstall extensions on my machine, so that I can efficiently control the installed extensions even in a disconnected state."
---

# `azcmagent extension`

Local management of Azure Arc extensions installed on the machine. These commands can be run even when a machine is in a disconnected state.

The extension manager must be stopped before running any of these commands. Stopping the extension manager interrupts any in-progress extension installs, upgrades, and removals. To disable the extension manager, run `Stop-Service ExtensionService` on Windows or `systemctl stop extd`. When you're done managing extensions locally, start the extension manager again with `Start-Service ExtensionService` on Windows or `systemctl start extd` on Linux.

## Commands

| Command | Purpose |
| ------- | ------- |
| [azcmagent extension list](#azcmagent-extension-list) | Lists extensions installed on the machine |
| [azcmagent extension remove](#azcmagent-extension-remove) | Uninstalls extensions on the machine |

## `azcmagent extension list`

Lists extensions installed on the machine.

### Usage

```
azcmagent extension list [flags]
```

### Examples

See which extensions are installed on your machine:

```
azcmagent extension list
```

### Flags

This command supports the flags described in [Common flags](azcmagent.md#common-flags).

## `azcmagent extension remove`

Uninstalls extensions on the machine.

### Usage

```
azcmagent extension remove [flags]
```

### Examples

Remove the `AzureMonitorWindowsAgent` extension from the local machine:

```
azcmagent extension remove --name AzureMonitorWindowsAgent
```

Remove all extensions from the local machine:

```
azcmagent extension remove --all
```

### Flags

This command supports the flags described in [Common flags](azcmagent.md#common-flags) and the flags listed in this section.

`--all`, `-a`

Removes all extensions from the machine.

`--name`, `-n`

Removes the specified extension from the machine. Use [azcmagent extension list](#azcmagent-extension-list) to get the name of the extension.
