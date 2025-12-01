---

title: Manage and migrate storage accounts using Azure Copilot
description: Learn how Azure Copilot can improve the security posture and data resiliency of storage accounts and help with storage migration solutions
ms.date: 11/19/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: "As a cloud administrator, I want to utilize an AI assistant to migrate data to Azure and effectively manage and improve storage accounts, so that I can efficiently manage and optimize our cloud storage resources."
---

# Manage and migrate storage accounts using Azure Copilot

Azure Copilot can provide contextual and dynamic responses to harden the security posture and enhance data resiliency of [storage accounts](/azure/storage/common/storage-account-overview). When [migrating data to Azure](#discover-storage-migration-solutions), you can get help finding the right solution. Azure Copilot can also help you troubleshoot and resolve common [Azure File Sync](/azure/storage/file-sync/file-sync-introduction) issues related to your stored data.

Responses are dynamic and based on your specific storage account and settings. Based on your prompts, Azure Copilot provides specific recommendations to improve your storage account or resolve issues.

When you ask Azure Copilot about storage accounts, it automatically pulls context when possible, based on the current conversation or on the page you're viewing in the Azure portal. If the context isn't clear, you'll be prompted to specify the storage resource for which you want information.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Improve security

You can ask Azure Copilot to help you improve the security of your storage account. Based on your prompts, Azure Copilot runs a security check and provides specific recommendations to improve your storage account.

### Security sample prompts

Here are a few examples of the kinds of prompts you can use to improve your storage account's security. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "How can I make this storage account more secure?"
- "Does this storage account follow security best practices?"
- "Is this storage account vulnerable?"
- "Prevent malicious users from accessing this storage account."

### Security example

You can ask "**How can I make this storage account more secure?**" If you're already working with a storage account, Azure Copilot asks if you'd like to run a security check on that resource. If it's not clear which storage account you're asking about, you'll be prompted to select one. After the check, you'll see specific recommendations about things you can do to align your storage account with security best practices.

:::image type="content" source="media/improve-storage-accounts/storage-account-security.png" alt-text="Screenshot showing Azure Copilot providing suggestions on storage account security best practices.":::

## Enhance data resiliency

You can ask Azure Copilot questions about enhancing your storage account's data resiliency. Azure Copilot runs a data resiliency check and lets you know how you can help protect your storage account's data.

### Data resiliency sample prompts

Here are a few examples of the kinds of prompts you can use to improve your storage account's data resiliency. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "Prevent this storage account from data loss."
- "How can I prevent this storage account from being deleted?"
- "How do I protect this storage account's data from data loss or theft?"

### Data resiliency example

You can say things like "**Prevent this storage account from data loss during a disaster recovery situation**." After confirming you'd like Azure Copilot to run a data resiliency check, you'll see specific recommendations for protecting its data.

:::image type="content" source="media/improve-storage-accounts/storage-account-data-resiliency.png" alt-text="Screenshot showing Azure Copilot providing suggestions to improve storage account data resiliency.":::

## Use storage migration solutions advisor

Selecting the appropriate tool for migrating data to Azure can be challenging. Various solutions are available, both online and offline, with some solutions providing features like repeatable sync, merge, and hybrid deployment options. Often, these tools bring overlapping technical capabilities, and multiple tools could appear to be suitable for similar use cases. Rather than evaluating a multitude of solutions, you can chat with Azure Copilot to explore your migration solution recommendations. This conversational approach makes it easier for anyone – from IT managers to storage admins – to gain insights and make data-driven decisions.

When you ask Azure Copilot for migration help, it asks questions to understand your scenario. Once Azure Copilot has the necessary information, it reviews native Azure services as well as partner and independent software vendor (ISV) solutions, then guides you to a recommended approach.

### Storage migration solution advisor sample prompts

Here are a few examples of the kinds of prompts you can use to get recommendations for migration solutions. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "How can I migrate my data into Azure?"
- "How can I migrate on-premises data into Azure?"
- "What solution should I use to migrate my data into Azure?"
- "Help me decide what migration service I should use to move my data into Azure"
- "How can I migrate data from an SMB share or NFS volume to Azure?"
- "I'm moving data from AWS S3 or Google Cloud Storage to Azure, how can I do this transfer?"
- "What’s the best tool to migrate large-scale unstructured data from on-premises to Azure?"
- "I need to migrate data from my on-premises file shares (like SMB or NFS) to Azure Storage (Blob, Files, Disks). How can I do that?"

### Storage migration solution advisor example

You can start getting help by saying "**How can I migrate my data into Azure?**" Azure Copilot starts by asking about the source of your data.

:::image type="content" source="media/improve-storage-accounts/storage-migration.png" alt-text="Screenshot of Azure Copilot responding to a question about migrating data to Azure.":::

Azure Copilot asks more questions to better understand your scenario, such as the data source, current protocols, and the target storage destination in Azure.

:::image type="content" source="media/improve-storage-accounts/storage-migration-questions.png" alt-text="Screenshot of Azure Copilot asking follow-up questions about a data migration scenario.":::

Next, Azure Copilot asks about your network and data requirements, such as the amount of data to be migrated, the available bandwidth, and the data transfer direction.

:::image type="content" source="media/improve-storage-accounts/storage-migration-network.png" alt-text="Screenshot of Azure Copilot asking about network and data requirements for a migration scenario.":::

Finally, Azure Copilot provides a recommended migration solution based on your responses. In this example, the recommendation is to use Azure Storage Mover. Azure Copilot provides a link to learn more about this solution, along with an option to start the Storage Mover creation process. You can also refine your answers to get a different recommendation.

:::image type="content" source="media/improve-storage-accounts/storage-migration-recommendation.png" alt-text="Screenshot of Azure Copilot providing a migration recommendation.":::

## Reduce storage costs

You can ask Azure Copilot to help you manage the costs for your storage accounts. One way to save on storage costs is by tiering blobs that haven't been accessed or modified for some time. In some instances, you may even wish to delete those blobs.

When you ask Azure Copilot about reducing your storage costs, it provides suggestions and helps you accomplish bulk tiering by automating [lifecycle management policy](/azure/storage/blobs/lifecycle-management-overview) authoring.

### Storage cost reduction sample prompts

Here are a few examples of the kinds of prompts you can use to reduce costs for your storage account. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "How can I save costs on my storage?"
- "Help me save costs on my storage"
- "I want to reduce the costs of my storage account"
- "Help me reduce the costs of my storage account"
- "I want to reduce the amount I am paying in storage"
- "Help me reduce the amount I’m paying in storage"
- "Help me write a lifecycle management policy for my storage account"
- "I want to create an LCM rule for my storage account"
- "Create an LCM policy for my storage account"
- "Help me create a lifecycle management rule for my storage account"

### Storage cost reduction example

If you say "**Help me lower my storage costs**", Azure Copilot will begin the lifecycle management rule creation process.

:::image type="content" source="media/improve-storage-accounts/storage-costs-lower.png" alt-text="Screenshot of Azure Copilot responding to a request to lower storage costs.":::

If the context isn't clear, Azure Copilot prompts you to select the storage account that you want to work with.  Next, you define a condition on which to trigger the rule.

:::image type="content" source="media/improve-storage-accounts/storage-costs-action-condition.png" alt-text="Screenshot of Azure Copilot confirming the action and condition for a storage account lifecycle management rule.":::

Finally, Azure Copilot asks you whether you want the rule to run on the entire account, or only on a subset of the account. In this example, the rule will only run on a subset of the account, defined by a blob prefix.

:::image type="content" source="media/improve-storage-accounts/storage-costs-account.png" alt-text="Screenshot of Azure Copilot confirming where to run a new lifecycle management rule.":::

Now that Azure Copilot has all of the necessary information, it creates the lifecycle management rule. You can copy and paste the rule yourself, or have Azure Copilot create and apply the rule for you.

:::image type="content" source="media/improve-storage-accounts/storage-costs-lifecycle-management-rule.png" alt-text="Screenshot of Azure Copilot providing a customized lifecycle management rule for a storage account." lightbox="media/improve-storage-accounts/storage-costs-lifecycle-management-rule.png":::

## Update storage account redundancy type

You can ask Azure Copilot to help you change the redundancy type of your storage account. Based on your prompts, Azure Copilot analyzes your storage account's configuration, checks for compatibility, and provides step-by-step guidance to help you complete the conversion successfully.

### Redundancy configuration sample prompts

Here are a few examples of the kinds of prompts you can use to manage redundancy changes. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific configurations.

- "Help me convert my storage account from LRS to ZRS."
- "Why can’t I migrate my storage account from ZRS to GRS?"
- "Upgrade the replication setting of my storage account to RA-GZRS."
- "Check what factors are preventing my storage account from converting to RA-GRS."

### Redundancy configuration example

You can ask, “**Help me convert my storage account from LRS to ZRS.**” If you're already working with a storage account, Copilot will use that context to run compatibility checks. If the account isn’t specified, you’ll be prompted to select one and define the target redundancy type.

Azure Copilot analyzes your account's configuration. It checks for any blockers, such as enabled features like point-in-time restore, archive data, or boot diagnostics, that may prevent the conversion. If blockers are found, you receive a detailed list along with resolution steps, such as disabling specific features or performing a manual migration.

If your desired conversion path is unsupported (for example, ZRS to GRS), Azure Copilot guides you through the required two-step process (such as ZRS → LRS → GRS or ZRS → GZRS → GRS), including all necessary checks and resolution steps if applicable.

If no blockers are found, Azure Copilot confirms that your account is ready for conversion and offers instructions for completing the change using the Azure portal.

:::image type="content" source="media/improve-storage-accounts/storage-account-redundancy-type.png" alt-text="Screenshot of Azure Copilot confirming the target replication for a storage account.":::

## Troubleshoot and resolve Azure File Sync issues

If you use [Azure File Sync](/azure/storage/file-sync/file-sync-introduction), Azure Copilot can help you quickly troubleshoot and resolve common issues. It analyzes your environment to identify potential root causes, such as network issues, incorrect permissions, or missing file shares. After that, Azure Copilot provides actionable recommendations.

### Azure File Sync sample prompts

Here are a few examples of the kinds of prompts you can use to help diagnose and fix your Azure File Sync environment. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

Troubleshooting specific error codes:

- "Help me troubleshoot error code 0x80C8305F."
- "Help me troubleshoot error code 0x80C83096."

Permissions and network configuration:

- "Do I have correct permissions to access the file share?"
- "Is my network configured correctly to access the file share?"

General troubleshooting:

- "How do I fix my Azure File Sync environment that stopped syncing?"
- "Help me fix sync errors in my AFS environment."
- "How do I fix a server that has a state of 'appears offline'?"
- "Help me diagnose issues with my Azure File Sync environment."

### Azure File Sync example

If you see an error code, you can ask Azure Copilot to help you understand the problem and how to fix it. When you say "**Help me troubleshoot this error**", Azure Copilot provides information about the error and the things that Azure Copilot can check to help resolve the issue. In some cases, Azure Copilot can even help fix the issue.

:::image type="content" source="media/improve-storage-accounts/storage-file-sync-error-troubleshoot.png" alt-text="Screenshot showing Azure Copilot troubleshooting an Azure File Sync error." lightbox="media/improve-storage-accounts/storage-file-sync-error-troubleshoot.png":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure Storage](/azure/storage/common/storage-introduction).
