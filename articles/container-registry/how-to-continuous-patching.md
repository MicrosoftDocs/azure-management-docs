## Installing the Continuous Patching Workflow

Run the following command to install the CLI extension:
```sh
az extension add --source https://acrcssc.z5.web.core.windows.net/acrcssc-1.1.1rc7-py3-none-any.whl
```

## Enable the Continuous Patching Workflow
To enable Continuous Patching, follow the series of steps below that outline the CLI process. These guidelines detail the lifecycle of a continuous patching workflow, from its creation to subsequent updates to eventual deletion. 
1. Login to Azure CLI with az login
```sh
az login
```
2. Login to ACR 
```sh
az acr login -n <myRegistry>
```
3. Run the following command to create a file named ```continuouspatching.json```, which contains the Continuous Patching JSON.

```sh
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
The schema ingests specific repositories and tags in an array format. Each variable is defined below:

- ```version``` allows the ACR team to track what schema version you're on. Do not change this variable unless instructed to.
- ```tag-convention``` this is an optional field. Allowed values are "incremental" or "floating" - refer to [Key Concepts](#key-concepts) for more information.

- ```repositories``` is an array that consists of all objects that detail repository and tag information
    - ```repository``` refers to repository name
    - ```tags``` is an array of tags separated by commas. The wildcard ```*``` can be used to signify all tags within that repository
    - ```enabled``` is a Boolean value of true or false determining if the specified repo is on or off

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
4. After creating your configuration file, it is recommended to execute a dry run to verify the intended artifacts are selected by the JSON criteria. The dry run requires a parameter called ```schedule```, which specifies how often your continuous patching cycle will run. The schedule flag is measured in days, with a minimum value of 1 day, and a maximum value of 30 days. For example, if you want an image to be patched everyday, you would specify schedule as ```1d```, or 1 day. If you want a weekly patch (once a week), you would fill schedule as ```7d```, or 7 days. 

Command Schema:
```sh
az acr supply-chain workflow create -r <registryname> -g <resourcegroupname> -t continuouspatchv1 --config <JSONfilepath> --schedule <number of days> --dry-run 
```
Example Command: 
```sh
az acr supply-chain workflow create -r myRegistry -g myResourceGroup -t continuouspatchv1 -–config ./continuouspatching.json --schedule 1d –-dry-run   
```

The ```--dry-run``` flag will output all specified artifacts by the JSON file configuration. Customers can verify that the right artifacts are selected. With the sample ubuntu configuration above, the following results should be displayed as output. 
```sh
Ubuntu: jammy-20240111
Ubuntu: jammy-20240125
```
Help command to see all required/optional flags.
```sh
az acr supply-chain workflow create --help
```
 
5. Once satisfied with the dry-run results, run the ```create``` command again without the ```--dry-run``` flag to officially create your continuous patching workflow.  

**Important**

The ```--schedule``` parameter follows a fixed-day multiplier starting from day 1 of the month. This means:

- If you specify ```--schedule 7d``` and run the command on the 3rd, the next scheduled run will be on the 7th—because 7 is the first multiple of 7 (days) after the 3rd, counting from day 1 of the month.

- If ```--schedule``` is 3d and today is the 7th, then the next scheduled run lands on the 9th—since 9 is the next multiple of 3 that follows 7.

- If you add the flag ```--run-immediately```, you trigger an immediate patch run. The subsequent scheduled run will still be aligned to the nearest day multiple from the 1st of the month, based on your ```--schedule``` value.

- The schedule counter **resets** every month. Regardless of the designated schedule, your workflow will run on the 1st of every month, then follow the specified schedule value for the remainder of the month. If my patching runs on the 28th of January, and my schedule is 7d, my next patch will run on Feburary 1st, then 8th, and continue following the 7 days. 

Command Schema:
```sh
az acr supply-chain workflow create -r <registryname> -g <resourcegroupname> -t continuouspatchv1 -–config <JSONfilename> --schedule <number of days> --run-immediately
```

Example Command: 
```sh
az acr supply-chain workflow create -r myRegistry -g myResourceGroup -t continuouspatchv1 -–config ./continuouspatching.json --schedule 1d --run-immediately
```

Upon a successful command (whether or not you include ```--run-immediately```), you will see:

- A success message confirming that your workflow tasks have been queued.

- An output parameter indicating when the next run of your workflow is scheduled, so you can track exactly when patching will occur again.

Help command to see all required/optional flags.
```sh
az acr supply-chain workflow create --help
```
## Use Azure Portal to view workflow tasks

Once the workflow succeeds, go to the Azure Portal to view your running tasks. Click into Services -> Repositories, and you should see a new repository named ```csscpolicies/patchpolicy```. This repository hosts the JSON configuration artifact that will be continuously referenced for continuous patching.  

![PortalRepos](./media/portal_repos1.png)

Next, click on "Tasks” under "Services”. You should see 3 new tasks, named the following:

![PortalTasks](./media/portal_tasks1.png)

- cssc-trigger-workflow – this task scans the configuration file and calls the scan task on each respective image.    
- cssc-scan-image – this task scans the image for operating system vulnerabilities. This task will only trigger the patching task only if (1) operating system vulnerabilities were found, and (2) the image is not considered End of Service Life (EOSL). For more information on EOSL, please consult [Preview Limitations](#preview-limitations).
- cssc-patch-image – this task patches the image.
These tasks work in conjunction to execute your continuous patching workflow.

You can also click on "Runs” within the "Tasks” view to see specific task runs. Here you can view status information on whether the task succeeded or failed, along with viewing a debug log. 

![PortalRun](./media/portal_runs1.png)

## Use CLI to view workflow tasks

You can also run the following CLI show command to see more details on each task and the general workflow. The command will output
- Schedule
- Creation date
- System data such as last modified date, by who, etc.

Command Schema
```sh
az acr supply-chain workflow show -r <registry> -g <resourceGroup> -t continuouspatchv1   
```
Example Command 
```sh
az acr supply-chain workflow show -r myRegistry -g myResourceGroup -t continuouspatchv1 
```
Help command to see all required/optional flags
```sh
az acr supply-chain workflow show --help
```

## Updating the Continuous Patching Workflow

In scenarios where you want to make edits to your continuous patching workflow, the update command is the easiest way to do so. You can update your schedule or JSON config schema with the update CLI command directly. 

Command Schema
```sh
az acr supply-chain workflow update -r <registry> -g <resourceGroup> -t continuouspatchv1 --config <JSONfilename> --schedule <number of days>
```
Example Command 
```sh
az acr supply-chain workflow update -r myRegistry -g myResourceGroup -t continuouspatchv1 --config ./continuouspatching.json --schedule 1d
```
Help command to see all required/optional flags
```sh
az acr supply-chain workflow update --help
```

To update your schedule, run the previous command with a new input for schedule. To update your JSON configuration, we recommend making changes to the file, running a dry-run, and running the update command. 

You can verify the updated workflow configuration by running the following show command or by clicking into your registry portal view. 
```sh
az acr supply-chain workflow show -r myregistry -g myresourcegroup -t continuouspatchv1
```

## Deleting the Continuous Patching Workflow

To delete the continuous patching workflow, please run the following CLI command

Command Schema
```sh
az acr supply-chain workflow delete -r <registry> -g <resourceGroup> -t continuouspatchv1 
```
Example Command 
```sh
az acr supply-chain workflow delete -r myregistry -g myresourcegroup –t continuouspatchv1
```
Help command to see all required/optional flags
```sh
az acr supply-chain workflow delete --help
```

Once a workflow is successfully deleted, the repository "csscpolicies/patchpolicy” will be automatically deleted. The 3 tasks that run your workflow will also be automatically deleted, along with any currently queued runs and previous logs. 