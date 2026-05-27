---
title: Overview of Kernel Livepatching on Azure Linux
description: Overview of how kernel livepatching works on Azure Linux and its limitations.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Kernel livepatching on Azure Linux overview

Livepatching applies kernel security fixes to a running system without a reboot. Azure Linux uses livepatching to deliver fixes for critical kernel CVEs between full kernel updates at no extra cost.

This article explains how kernel livepatching works on Azure Linux and its limitations.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## How livepatching works

Livepatching works by applying small, targeted patches to the running kernel in memory. These patches modify only the vulnerable functions or code paths affected by a CVE, without requiring a full kernel replacement or system reboot. The livepatch mechanism ensures that the patched code is executed in place of the vulnerable code while the system continues to run normally. Each livepatch targets a specific kernel release. During preview periods, livepatches typically target critical-level CVEs. Livepatches aren't a replacement for kernel updates.

## Kernel livepatching limitations

- Not every kernel CVE can be delivered as a livepatch.
- Livepatching is a short-term mitigation until a full kernel update is applied.
- A livepatch might be unavailable for your current kernel version.

## Related content

- [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md)
- [Azure Linux security overview](./security-overview.md)
