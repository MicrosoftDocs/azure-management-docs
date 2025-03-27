

## Listing Running Tasks

To list the most recently executed Continuous Patching tasks, the following List command is available:
```sh
az acr supply-chain workflow list -r <registryname> -g <resourcegroup> [–-run-status <failed || successful || running>] -t continuouspatchv1
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
```sh
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
```sh
az acr supply-chain workflow cancel-run -r <registryname> -g <resourcegroup> --type <continuouspatchv1>
```

This command will cancel all Continuous Patching tasks within the registry with a status of "Running”, "Queued” and "Started”. The command will output a success or failure. Failure results will follow the failure pattern of the other workflow commands if the input is incorrect.
Running the cancel command will only affect tasks in the current schedule. 

For example, if a user has their schedule for 1d, and runs the cancel command, tasks in those 3 states will be canceled for that day, but will be requeued for the next day. If the schedule was a week, then that week's tasks would be canceled, but the following week would have the tasks requeued. The main scenario for this command is when a user misconfigures their continuous patching workflow and doesn't want to wait for all tasks to finish running.

## Troubleshooting Tips

Use the task list command to output all failed tasks. Specifying the "cssc-patch” command is best for failure. The documentation on the task-list [command](https://learn.microsoft.com/en-us/cli/azure/acr/task?view=azure-cli-latest#az-acr-task-list-runs) is here. 

Task-list command for top 10 failed patch tasks
```sh
az acr task list-runs -r registryname -n cssc-patch-image --run-status Failed --top 10
```

This command will output all failed tasks. To investigate a specific failure, grab the runID that's outputted from this command and run
```sh
az acr task logs -r registryname --run-id <run-id>
```
If the logs aren't sufficient, or an issue is persistent, or for any feedback, please email the ACR team at acr-patching-preview@microsoft.com 

## Appendix

**Possible CLI Outputs for 'List' Command**

```sh
az acr supply-chain workflow list -r <registryname> -g <resourcegroup> [–-run-status <Failed || Queued || Running || Skipped || Succeeded || Unknown>]
```

If scan and patch are successful

```sh
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
```sh
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
```sh
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
```sh
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

if scan is successful and no patch is needed and NO patched image exists yet
```sh
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
```sh
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
```sh
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
```sh
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
```sh
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
```sh
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
```sh
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