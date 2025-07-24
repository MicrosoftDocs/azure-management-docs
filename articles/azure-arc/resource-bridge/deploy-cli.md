---
title: Azure Arc resource bridge deployment command overview
description: Learn about the Azure CLI commands that can be used to manage your Azure Arc resource bridge deployment.
ms.date: 04/23/2025
ms.topic: overview
ms.custom: devx-track-azurecli
# Customer intent: As a cloud engineer, I want to use Azure CLI commands for deploying and managing the Arc resource bridge, so that I can integrate on-premises resources with Azure for better hybrid cloud management.
---

# Azure Arc resource bridge deployment command overview

[Azure CLI](/cli/azure/install-azure-cli) is required to deploy the Azure Arc resource bridge. When you deploy Arc resource bridge with a corresponding partner product, Azure CLI commands may be combined into an automation script, along with additional provider-specific commands.

The following diagram illustrates the deployment architecture for Arc resource bridge.

:::image type="content" source="media/deploy-cli/arc-resource-bridge-deploy-overview.png" alt-text="Diagram showing the deployment architecture for Azure Arc resource bridge." lightbox="media/deploy-cli/arc-resource-bridge-deploy-overview.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

To learn about installing Arc resource bridge with a corresponding partner product, see:

- [Connect VMware vCenter Server to Azure with Arc resource bridge](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md)
- [Connect System Center Virtual Machine Manager (SCVMM) to Azure with Arc resource bridge](../system-center-virtual-machine-manager/quickstart-connect-system-center-virtual-machine-manager-to-arc.md#download-the-onboarding-script)
- [Azure Arc VM Management on Azure Local through Arc resource bridge](/azure/azure-local/manage/azure-arc-vm-management-overview)

This article provides an overview of the [Azure CLI commands](/cli/azure/arcappliance) that are used to manage Arc resource bridge deployment, in the order in which they're typically used for deployment.

## `az arcappliance createconfig`

This command creates the configuration files used by Arc resource bridge. Credentials that are provided during `createconfig`, such as vCenter credentials for VMware vSphere, are stored in a configuration file and locally within Arc resource bridge. These credentials should be a separate user account used only by Arc resource bridge, with permission to view, create, delete, and manage on-premises resources. If the credentials change, then the credentials on the resource bridge should be updated.

The `createconfig` command features two modes: interactive and non-interactive. Interactive mode provides helpful prompts that explain the parameter and what to pass. To initiate interactive mode, pass only the three required parameters. Non-interactive mode allows you to pass all the parameters needed to create the configuration files without being prompted, which saves time and is useful for automation scripts.

Three configuration files are generated: resource.yaml, appliance.yaml and infra.yaml. These files should be kept and stored in a secure location, as they're required for maintenance of Arc resource bridge.

This command also calls the `validate` command to check the configuration files.

> [!NOTE]
> Azure Local uses different commands to create the Arc resource bridge configuration files.

## `az arcappliance validate`

The `validate` command checks the configuration files for a valid schema, cloud and core validations (such as management machine connectivity to [required URLs](network-requirements.md)), network settings, and proxy settings. It also performs tests on identity privileges and role assignments, network configuration, load balancer configuration, and content delivery network connectivity.

## `az arcappliance prepare`

This command downloads the OS images from Microsoft that are used to deploy the on-premises appliance VM. Once downloaded, the images are then uploaded to the local cloud image gallery to prepare for the creation of the appliance VM.

This command generally takes 10-30 minutes to complete, depending on the network speed. Allow the `prepare` command to complete before continuing with the deployment.

## `az arcappliance deploy`

The `deploy` command deploys an on-premises instance of Arc resource bridge as an appliance VM, bootstrapped to be a Kubernetes management cluster. This command gets all necessary pods and agents within the Kubernetes cluster into a running state. Once the appliance VM is up, the kubeconfig file is generated.

## `az arcappliance create`

This command creates Arc resource bridge in Azure as an ARM resource, then establishes the connection between the ARM resource and on-premises appliance VM.

Once the `create` command initiates the connection, it returns in the terminal, even though the connection between the ARM resource and on-premises appliance VM isn't complete yet. The resource bridge needs about five minutes to establish the connection between the ARM resource and the on-premises VM.

## `az arcappliance show`

The `show` command gets the status of the Arc resource bridge and ARM resource information. It can be used to check the progress of the connection between the ARM resource and on-premises appliance VM.

While the Arc resource bridge is connecting the ARM resource to the on-premises VM, the resource bridge progresses through the following stages:

`ProvisioningState` may be `Creating`, `Created`, `Failed`, `Deleting`, or `Succeeded`.

`Status` transitions between `WaitingForHeartbeat` -> `Validating` ->  `Connecting` -> `Connected` -> `Running`.

- `WaitingForHeartbeat`: Azure is waiting to receive a signal from the appliance VM.
- `Validating`: Appliance VM is checking Azure services for connectivity and serviceability.
- `Connecting`: Appliance VM is syncing on-premises resources to Azure.
- `Connected`: Appliance VM completed sync of on-premises resources to Azure.
- `Running`: Appliance VM and Azure have completed hybrid sync, and Arc resource bridge is now operational.

Successful Arc resource bridge creation results in `ProvisioningState = Succeeded` and `Status = Running`.

## `az arcappliance delete`

This command deletes the appliance VM and Azure resources. It doesn't clean up the OS image, which remains in the on-premises cloud gallery.

If a deployment fails, run this command to clean up the environment before you attempt to deploy again.

## Next steps

- Explore the full list of [Azure CLI commands and required parameters](/cli/azure/arcappliance) for Arc resource bridge.
- Get [troubleshooting tips for Arc resource bridge](troubleshoot-resource-bridge.md).
