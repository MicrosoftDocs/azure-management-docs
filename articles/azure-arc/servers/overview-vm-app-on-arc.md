---
title: VM Applications overview for Azure Arc-enabled servers (preview)
description: Learn how to prepare, deploy, and manage VM Applications on Azure Arc-enabled servers.
ms.date: 11/05/2026
ms.topic: overview
ai-usage: ai-assisted
---

# VM applications overview for Azure Arc-enabled servers (preview)

VM Applications simplify application management for Azure Arc-enabled servers across on-premises, edge, and multicloud environments. You package applications once, publish them to Azure Compute Gallery, and deploy them to both Azure virtual machines (VMs) and Arc-enabled servers. For shared capabilities, gallery resource properties, limits, and costs, see [VM Applications overview](/azure/virtual-machines/vm-applications).

## How VM applications on Azure Arc-enabled servers work

VM Applications integrate with your existing development workflows and Azure infrastructure. Teams publish versioned application packages, scripts, and configurations to Azure Compute Gallery, then deploy or update them on Azure Arc-enabled servers. Use the Azure portal, command-line tools, APIs, or infrastructure as code, and automate publishing and rollout through your CI/CD pipelines.

After deployment, review application status and operation results for individual servers, or use Azure Resource Graph to track inventory across your fleet. Use Azure Policy to audit application requirements and deploy missing applications.
## Prepare and create a VM application

Follow the preparation and publishing instructions in [Create and deploy VM Application](/azure/virtual-machines/vm-applications-how-to). Ensure your package and commands are compatible with the target server's operating system and application dependencies.

1. **Set up Azure resources.** Create the storage account, blob container, and Azure Compute Gallery described in the [prerequisites](/azure/virtual-machines/vm-applications-how-to#prerequisites). For publishing without SAS tokens, see [Publish VM Applications using a managed identity](/azure/virtual-machines/vm-applications-publish-with-managed-identity).
1. **Prepare the package and commands.** Package your application, prepare an optional configuration file, and define installation and removal commands. See [Package the application](/azure/virtual-machines/vm-applications-how-to#package-the-application) for packaging requirements and script examples.
1. **Upload the application files.** Upload the package and optional configuration file, then obtain the blob URLs or read-only SAS URLs needed for publishing. See [Upload the application files to Azure storage account](/azure/virtual-machines/vm-applications-how-to#upload-the-application-files-to-azure-storage-account).
1. **Create the application definition and version.** Specify the supported operating system, package source, commands, and replication regions. Follow [Create the VM Application](/azure/virtual-machines/vm-applications-how-to#create-the-vm-application).

Replicate the application version to the Azure region associated with each target Arc-enabled server. After publishing and replication finish, follow [Deploy VM Applications on Arc-enabled servers](deploy-vm-app-on-arc.md).

## Deploy and manage applications on Arc-enabled servers

Arc-enabled servers require Azure Connected Machine agent version 1.69 or later and outbound access to the required Azure Storage endpoints. Arc-enabled Linux servers also require OpenSSL version 1.1.1 or later installed. Review the full [deployment prerequisites](deploy-vm-app-on-arc.md#prerequisites) before you deploy.

Use the Arc-specific guides for the rest of the application lifecycle:

- [Deploy VM Applications on Arc-enabled servers](deploy-vm-app-on-arc.md) for deployment commands and resource properties.
- [Manage VM Applications on Arc-enabled servers](manage-vm-app-on-arc.md) to review operation results, update a deployed application, or remove it.
- [Govern and monitor VM Applications on Arc-enabled servers](monitor-vm-app-on-arc.md) to inventory deployments with Azure Resource Graph and audit required applications with Azure Policy.

## Next steps

- [Prepare and publish a VM Application](/azure/virtual-machines/vm-applications-how-to).
- [Deploy a published version to an Arc-enabled server](deploy-vm-app-on-arc.md).
