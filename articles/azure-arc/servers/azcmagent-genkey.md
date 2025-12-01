---
title: CLI reference for `azcmagent genkey`
description: Syntax for the `azcmagent genkey` command line tool
ms.topic: reference
ms.date: 12/01/2025
# Customer intent: "As a system administrator, I want to generate a private-public key pair using the command line, so that I can onboard a server to Azure Arc-enabled virtual machines efficiently and securely."
---

# `azcmagent genkey`

Generates a private-public key pair that can be used to onboard a machine asynchronously. This command is only used when connecting a server to an Azure Arc-enabled virtual machine offering (for example, [Azure Arc-enabled VMware vSphere](../vmware-vsphere/overview.md)). You should normally use [`azcmagent connect`](azcmagent-connect.md) to configure the agent.

## Usage

```
azcmagent genkey [flags]
```

## Examples

Generate a key pair and print the public key to the console:

```
azcmagent genkey
```

## Flags

This command supports the flags described in [Common flags](azcmagent.md#common-flags).
