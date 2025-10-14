---
ms.service: azure-arc
ms.subservice: azure-arc-container-storage
ms.topic: include
ms.date: 03/12/2025
author: sethmanheim
ms.author: sethm
# Customer intent: "As a system administrator, I want to configure Linux with Ubuntu for a cluster setup, so that I can ensure adequate file monitoring capabilities are in place."
---

## Prepare Linux with Ubuntu

This section describes how to prepare Linux with Ubuntu if you run a single-node or two-node cluster.

1. Run the following command to determine if you set `fs.inotify.max_user_instances` to 1024:

   ```bash
   sysctl fs.inotify.max_user_instances
   ```

   After you run this command, if it outputs less than 1024, run the following command to increase the maximum number of files and reload the **sysctl** settings:

   ```bash
   echo 'fs.inotify.max_user_instances = 1024' | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```
