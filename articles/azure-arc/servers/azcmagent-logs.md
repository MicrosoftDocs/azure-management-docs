---
title: CLI reference for `azcmagent logs`
description: Syntax for the `azcmagent logs` command line tool
ms.topic: reference
ms.date: 11/19/2025
# Customer intent: "As a system administrator, I want to collect log files from the Azure connected machine agent into a ZIP archive, so that I can efficiently troubleshoot and analyze issues related to the system's performance."
---

# `azcmagent logs`

Collects log files for the Azure connected machine agent and extensions into a ZIP archive.

## Usage

```
azcmagent logs [flags]
```

## Examples

Collect the most recent log files and store them in a ZIP archive in the current directory:

```
azcmagent logs
```

Collect all log files and store them in a specific location:

```
azcmagent logs --full --output "/tmp/azcmagent-logs.zip"
```

## Flags

This command supports the flags described in [Common flags](azcmagent.md#common-flags) and the flags listed in this section.

`-f`, `--full`

Collect all log files on the system, instead of just the most recent. Useful when troubleshooting older problems.

`-o`, `--output`

Specifies the path and name for the ZIP file. If this flag isn't specified, the ZIP is saved to the console's current directory with the name `azcmagent-_TIMESTAMP_-_COMPUTERNAME_.zip`.

Sample value: `custom-logname.zip`
