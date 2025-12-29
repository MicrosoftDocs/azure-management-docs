---
title: "Configure Continuous Patching in Azure Container Registry"
description: "Learn how to configure continuous patching in Azure Container Registry."
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to
ms.date: 03/27/2025

# Customer intent: As a DevOps engineer, I want to configure continuous patching in my container registry, so that I can automatically detect and remediate OS vulnerabilities in container images to maintain security and compliance.
---
# Configure continuous patching in Azure Container Registry

In this article, you learn how to install, enable, and configure continuous patching. Continuous patching when enabled for a container registry will automatically detect and remediate OS (operating system) level vulnerabilities for container images.

## Prerequisites        

- You can use the Azure Cloud Shell or a local installation of the Azure CLI with a minimum version of 2.15.0 or later. 
- You have an existing Resource Group with an Azure Container Registry (ACR).
- You have an Azure Container Registry with ACR Tasks enabled (ACR Tasks isn't supported in the free tier of ACR).

## Installing the Continuous Patching Workflow

Run the following command to install the CLI extension:
```azurecli
    az extension add -n acrcssc
```

## Enable the Continuous Patching Workflow

1. Log in to Azure CLI with az login.
```azurecli
az login
```
2. Log in to ACR.
```azurecli
az acr login -n <myRegistry>
```
3. Run the following command to create a file named ```continuouspatching.json```, which contains the Continuous Patching JSON. The JSON file name is flexible. 

```azurecli
cat <<EOF > continuouspatching.json
{
    "version": "v1",
    "tag-convention" : "<incremental|floating>",
    "repositories": [{
        "repository": "<Repository Name>",
        "tags": ["<comma-separated-tags>"],   
        "enabled": <true|false>
    }] 
}
EOF
```
The schema ingests specific repositories and tags in an array format. Each variable is defined here:

- ```version``` allows the ACR team to track what schema version you're on. Don't change this variable unless instructed to.
- ```tag-convention```  is an optional field. Allowed values are "incremental" or "floating" - refer to [Key Concepts of Continuous Patching](key-concept-continuous-patching.md) for more information. 

- ```repositories``` is an array that consists detailed repository and tag information
    - ```repository``` refers to repository name
    - ```tags``` is an array of tags separated by commas. The wildcard ```*``` can be used to signify all tags within that repository.
    - ```enabled``` is a Boolean value of true or false determining if the specified repo is enabled or not.

The following details an example configuration for a customer who wants to patch all tags (use the * symbol) within the repository ```python```, and to patch specifically the ```jammy-20240111``` and ```jammy-20240125``` tags in the repository ```ubuntu```. 

JSON example:
```json
{
"version": "v1",
"tag-convention" : "incremental",
"repositories": [{
        "repository": "python",
        "tags": ["*"],
        "enabled": true
    },
    {
        "repository": "ubuntu",
        "tags": ["jammy-20240111", "jammy-20240125"],
        "enabled": true, 
    }]
}
```
4. After creating your configuration file, it's recommended to execute a dry run to verify the intended artifacts are selected by the JSON criteria. The dry run requires a parameter called ```schedule```, which specifies how often your continuous patching cycle runs. The schedule flag is measured in days, with a minimum value of one day, and a maximum value of 30 days. For example, if you want an image to be patched everyday, you would specify schedule as ```1d```, or 1 day. If you want a weekly patch (once a week), you would fill schedule as ```7d```, or 7 days. 

Command Schema:
```azurecli
az acr supply-chain workflow create -r <registryname> -g <resourcegroupname> -t continuouspatchv1 --config <JSONfilepath> --schedule <number of days> --dry-run 
```
Example Command: 
```azurecli
az acr supply-chain workflow create -r myRegistry -g myResourceGroup -t continuouspatchv1 --config ./continuouspatching.json --schedule 1d --dry-run   
```

The ```--dry-run``` flag outputs all specified artifacts by the JSON file configuration. Customers can verify that the right artifacts are selected. With the sample ubuntu configuration, the following results should be displayed as output. 
```azurecli
Ubuntu: jammy-20240111
Ubuntu: jammy-20240125
```
Help command to see all required/optional flags:
```azurecli
az acr supply-chain workflow create --help
```
 
5. Once satisfied with the dry-run results, run the ```create``` command again without the ```--dry-run``` flag to officially create your continuous patching workflow.  

> [!NOTE]
> The ```--schedule``` parameter follows a fixed-day multiplier starting from day 1 of the month. This means:
> - If you specify ```--schedule 7d``` and run the command on the 3rd, the next scheduled run will be on the 7th—because 7 is the first multiple of 7 (days) after the 3rd, counting from day 1 of the month.
> - If ```--schedule``` is 3d and today is the 7th, then the next scheduled run lands on the 9th—since 9 is the next multiple of 3 that follows 7.
> - If you add the flag ```--run-immediately```, you trigger an immediate patch run. The subsequent scheduled run will still be aligned to the nearest day multiple from the first of the month, based on your ```--schedule``` value.
> - The schedule counter **resets** every month. Regardless of the designated schedule, your workflow will run on the first of every month, then follow the specified schedule value for the remainder of the month. If my patching runs on January 28, and my schedule is 7d, my next patch will run on February first, then eighth, and continue following the 7 days. 

Command Schema:
```azurecli
az acr supply-chain workflow create -r <registryname> -g <resourcegroupname> -t continuouspatchv1 --config <JSONfilename> --schedule <number of days> --run-immediately
```

Example Command: 
```azurecli
az acr supply-chain workflow create -r myRegistry -g myResourceGroup -t continuouspatchv1 --config ./continuouspatching.json --schedule 1d --run-immediately
```

Upon a successful command (whether or not you include ```--run-immediately```), you should see:

- A success message confirming that your workflow tasks are queued.

- An output parameter indicating when the next run of your workflow is scheduled, so you can track exactly when patching will occur again.

Help command for all required/optional flags.
```azurecli
az acr supply-chain workflow create --help
```
## Use Azure portal to view workflow tasks

1. Once the workflow succeeds, go to the Azure portal to view your running tasks. In the service menu, under **Services**, select **Repositories. You should see a new repository named ```csscpolicies/patchpolicy```. This repository hosts the JSON configuration artifact that is continuously referenced for continuous patching.

2. Next, under **Services**, select select **Tasks**. You should see three new tasks:

:::image type="content" source="media/continuous-patching-media/portal-tasks-1.png" alt-text="Screenshot that shows the tasks created for continuous patching." lightbox="media/continuous-patching-media/portal-tasks-1.png":::

Tasks:
- cssc-trigger-workflow - this task scans the configuration file and calls the scan task on each respective image.    
- cssc-scan-image - this task scans the image for operating system vulnerabilities. This task triggers the patching task only if operating system vulnerabilities were found.
- cssc-patch-image - this task patches the image.
These tasks work in conjunction to execute your continuous patching workflow.

3. You can also select on "Runs” within the "Tasks” view to see specific task runs. Here you can view status information on whether the task succeeded or failed, along with viewing a debug log. 

:::image type="content" source="media/continuous-patching-media/portal-runs-1.png" alt-text="Screenshot that shows tasks run for continuous patching." lightbox="media/continuous-patching-media/portal-runs-1.png":::

## Use CLI to view workflow tasks

You can also run the following CLI show command to see more details on each task and the general workflow. The command outputs:
- Schedule
- Creation date
- System data such as last modified date, by who, etc.

Command Schema:
```azurecli
az acr supply-chain workflow show -r <registry> -g <resourceGroup> -t continuouspatchv1   
```
Example Command:
```azurecli
az acr supply-chain workflow show -r myRegistry -g myResourceGroup -t continuouspatchv1 
```
Help command for all required/optional flags:
```azurecli
az acr supply-chain workflow show --help
```

## Updating the Continuous Patching Workflow

In scenarios where you want to make edits to your continuous patching workflow, the update command is the easiest way to do so. You can update your schedule or JSON config schema with the update CLI command directly. 

Command Schema:
```azurecli
az acr supply-chain workflow update -r <registry> -g <resourceGroup> -t continuouspatchv1 --config <JSONfilename> --schedule <number of days>
```
Example Command:
```azurecli
az acr supply-chain workflow update -r myRegistry -g myResourceGroup -t continuouspatchv1 --config ./continuouspatching.json --schedule 1d
```
Help command for all required/optional flags:
```azurecli
az acr supply-chain workflow update --help
```

To update your schedule, run the previous command with a new input for schedule. To update your JSON configuration, we recommend making changes to the file, running a dry-run, and then running the update command. 


## Deleting the Continuous Patching Workflow

To delete the continuous patching workflow, please run the following CLI command.

Command Schema:
```azurecli
az acr supply-chain workflow delete -r <registry> -g <resourceGroup> -t continuouspatchv1 
```
Example Command:
```azurecli
az acr supply-chain workflow delete -r myregistry -g myresourcegroup -t continuouspatchv1
```
Help command for all required/optional flags:
```azurecli
az acr supply-chain workflow delete --help
```

Once a workflow is successfully deleted, the repository "csscpolicies/patchpolicy” will be automatically deleted. The three tasks that run your workflow will be deleted as well, along with any currently queued runs. 