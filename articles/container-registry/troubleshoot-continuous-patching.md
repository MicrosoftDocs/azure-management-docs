---
title: "Troubleshoot Continuous Patching in Azure Container Registry"
description: "Learn how to troubleshoot continuous patching in Azure Container Registry."
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: troubleshooting-general #Don't change.
ms.date: 03/27/2025

# Customer intent: "As a DevOps engineer, I want to troubleshoot continuous patching tasks in a container registry, so that I can ensure proper configuration and timely updates of container images."
---
# Troubleshoot continuous patching in Azure Container Registry
The troubleshooting tips in this article can help you resolve common issues that you may encounter when using continuous patching in Azure Container Registry. Two new commands will be introduced to help debug. 

## Listing Running Tasks

To list the most recently executed Continuous Patching tasks, the following List command is available:
```azurecli
az acr supply-chain workflow list -r <registryname> -g <resourcegroup> [--run-status <failed || successful || running>] -t continuouspatchv1
```

A successful result will return the following information:
-	Image name and tag
-	Workflow type
-	Scan status
-	Last scan date and time (if status failed, date would be left blank)
-	Scan task ID (for further debugging)
-	Patch Status
-	Last patch date and time (if status failed, date would be left blank)
-	Patched image name + tag
-	Patch task ID (for further debugging)

Example
```azurecli
ubuntu:jammy-20240111
scan status: successful
scan date: 2024-07-02T14:02:00
scan task ID: abc
patch status: successful
patch date: 2024-07-02T14:04:00
patch task id: def
patched image: ubuntu:jammy-20240111-1
workflow type: continuouspatchv1
```
The [--run-status] will return all tasks statuses that match the specified filter. This CLI command provides important debugging information.

For example, If the "failed" value is specified under run-status, only images which have failed their patching will be listed.

See Appendix for a full list of possible outputs.

## Canceling Running Tasks

Certain scenarios may require you to cancel tasks which are currently running or waiting to run. For this purpose, please run the following CLI command:
```azurecli
az acr supply-chain workflow cancel-run -r <registryname> -g <resourcegroup> --type <continuouspatchv1>
```

This command cancels all continuous patching tasks with a status of “Running,” “Queued,” or “Started” for the current schedule. For example, if you cancel tasks on a daily schedule (--schedule 1d), tasks in those states are canceled for that day but are requeued the next day. If your schedule is weekly, canceled tasks appear again the following week.

A typical reason to cancel is a misconfiguration you’d prefer to fix right away, rather than waiting for the patch tasks to complete. The command then returns a success or failure status.

## Troubleshooting Tips
### Finding Failed Tasks
Use the task list command to output all failed tasks. Specifying the "cssc-patch” command is best for failure. 

Task-list command for top 10 failed patch tasks
```azurecli
az acr task list-runs -r <registryname> -n cssc-patch-image --run-status Failed --top 10
```

This command will output all failed tasks. To investigate a specific failure, grab the runID that's outputted from this command and run
```azurecli
az acr task logs -r <registryname> --run-id <run-id>
```
### Misconfigured Workflow
Cancel queued tasks with the cancel command.
```azurecli
az acr supply-chain workflow cancel-run -r <registryname> -g <resourcegroup> --type <continuouspatchv1>
```
Reconfigure your continuous patching workflow after.

## Appendix

**Possible CLI Outputs for 'List' Command**

```azurecli
az acr supply-chain workflow list -r <registryname> -g <resourcegroup> [--run-status <Failed || Queued || Running || Skipped || Succeeded || Unknown>]
```

If scan and patch are successful

```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Succeeded
        patch date: 2024-09-13 21:07:32.841962+00:00
        patch task ID: xyz2
        last patched image: import:dotnetapp-manual-patched
        workflow type: continuouspatchv1
```

If scan is successful but patch isn't (with a previous patched image available)
```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Failed
        patch date: 2024-09-13 21:07:32.841962+00:00
        patch task ID: xyz2
        last patched image: import:dotnetapp-manual-patched
        workflow type: continuouspatchv1
```

If scan is successful but patch isn't (with NO previous patched image available)
```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Failed
        patch date: 2024-09-13 21:07:32.841962+00:00
        patch task ID: xyz2
        last patched image: ---No patch image available---
        workflow type: continuouspatchv1
```

If scan is successful and no patch is needed (no OS vulnerabilities found)
```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Skipped
        skipped patch reason: no vulnerability found in the image import:dotnetapp-manual image: 
        patch date: ---Not Available---
        patch task ID: ---Not Available---
        last patched image: import:dotnetapp-manual-patched
        workflow type: continuouspatchv1
```

If scan is successful and no patch is needed and NO patched image exists yet
```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Skipped
        skipped patch reason: no vulnerability found in the image import:dotnetapp-manual image: 
        patch date: ---Not Available---
        patch task ID: ---Not Available---
        last patched image: ---Not Available---
        workflow type: continuouspatchv1
```

If scan is a failure and a patched image exists
```azurecli
image: import:dotnetapp-manual
        scan status: Failed
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: ---Not Available---
        patch date: ---Not Available---
        patch task ID: ---Not Available---
        last patched image: import:dotnetapp-manual-patched
        workflow type: continuouspatchv1
```

If scan is a failure and NO previous patched image exists
```azurecli
image: import:dotnetapp-manual
        scan status: Failed
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: ---Not Available---
        patch date: ---Not Available---
        patch task ID: ---Not Available---
        last patched image: ---Not Available---
        workflow type: continuouspatchv1
```

If scan is currently running and a patched image exists
```azurecli
image: import:dotnetapp-manual
        scan status: Running
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: ---Not Available---
        patch date: ---Not Available---
        patch task ID: ---Not Available---
        last patched image: import:dotnetapp-manual-patched
        workflow type: continuouspatchv1
```

If scan is currently running and NO patched image exists
```azurecli
image: import:dotnetapp-manual
        scan status: Running
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: ---Not Available---
        patch date: ---Not Available---
        patch task ID: ---Not Available---
        last patched image: ---Not Available---
        workflow type: continuouspatchv1
```

If patch is currently running and a patched image exists
```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Running
        patch date: 2024-09-13 21:07:32.841962+00:00
        patch task ID: xyz2
        last patched image: import:dotnetapp-manual-patched
        workflow type: continuouspatchv1
```

If patch is currently running and NO patched image exists
```azurecli
image: import:dotnetapp-manual
        scan status: Succeeded
        scan date: 2024-09-13 21:05:58.841962+00:00
        scan task ID: dt21
        patch status: Running
        patch date: 2024-09-13 21:07:32.841962+00:00
        patch task ID: xyz2
        last patched image: ---Not Available---
        workflow type: continuouspatchv1
```