---
title: Disable the Run command extension
description: TBD
ms.date: 02/28/2025
ms.topic: how-to
ms.custom: devx-track-azurecli, devx-track-azurepowershell
---

# Disable the Run command extension

**DELETE - BECAUSE THIS IS PART OF SECURITY. SAME INFO. ADD SENTENCE THERE.**

To disable the Run command on Azure Arc-enabled servers, open an administrative command prompt and run the following commands. These commands use the local agent configuration capabilities for the Connected Machine agent in the Extension blocklist.

**Windows**

`azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerwindows"`

**Linux**

`sudo azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerlinux"`
