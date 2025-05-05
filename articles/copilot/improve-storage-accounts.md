---
title: Manage and troubleshoot storage accounts using Microsoft Copilot in Azure
description: Learn how Microsoft Copilot in Azure can improve the security posture and data resiliency of storage accounts.
ms.date: 04/21/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
---

# Manage and improve storage accounts using Microsoft Copilot in Azure

Microsoft Copilot in Azure can provide contextual and dynamic responses to harden the security posture and enhance data resiliency of [storage accounts](/azure/storage/common/storage-account-overview). It can also help you troubleshoot and resolve common [Azure File Sync](/azure/storage/file-sync/file-sync-introduction) issues related to your stored data.

Responses are dynamic and based on your specific storage account and settings. Based on your prompts, Microsoft Copilot in Azure provides specific recommendations to improve your storage account or resolve issues.

When you ask Microsoft Copilot in Azure about storage accounts, it automatically pulls context when possible, based on the current conversation or on the page you're viewing in the Azure portal. If the context isn't clear, you'll be prompted to specify the storage resource for which you want information.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Improve security

You can ask Copilot in Azure to help you improve the security of your storage account. Based on your prompts, Copilot in Azure runs a security check and provides specific recommendations to improve your storage account.

### Security sample prompts

Here are a few examples of the kinds of prompts you can use to improve your storage account's security. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "How can I make this storage account more secure?"
- "Does this storage account follow security best practices?"
- "Is this storage account vulnerable?"
- "Prevent malicious users from accessing this storage account."

### Security example

You can ask "**How can I make this storage account more secure?**" If you're already working with a storage account, Microsoft Copilot in Azure asks if you'd like to run a security check on that resource. If it's not clear which storage account you're asking about, you'll be prompted to select one. After the check, you'll see specific recommendations about things you can do to align your storage account with security best practices.

:::image type="content" source="media/improve-storage-accounts/storage-account-security.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing suggestions on storage account security best practices.":::

## Enhance data resiliency

You can ask Copilot in Azure questions about enhancing your storage account's data resiliency. Copilot in Azure runs a data resiliency check and lets you know how you can help protect your storage account's data.

### Data resiliency sample prompts

Here are a few examples of the kinds of prompts you can use to improve your storage account's data resiliency. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "Prevent this storage account from data loss."
- "How can I prevent this storage account from being deleted?"
- "How do I protect this storage account's data from data loss or theft?"

### Data resiliency example

You can say things like "**Prevent this storage account from data loss during a disaster recovery situation**." After confirming you'd like Microsoft Copilot in Azure to run a data resiliency check, you'll see specific recommendations for protecting its data.

:::image type="content" source="media/improve-storage-accounts/storage-account-data-resiliency.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing suggestions to improve storage account data resiliency.":::

## Troubleshoot and resolve Azure File Sync issues

If you use [Azure File Sync](/azure/storage/file-sync/file-sync-introduction), Copilot in Azure can help you quickly troubleshoot and resolve common issues. It analyzes your environment to identify potential root causes, such as network issues, incorrect permissions, or missing file shares. After that, Copilot in Azure provides actionable recommendations.

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

If you see an error code, you can ask Copilot in Azure to help you understand the problem and how to fix it. When you say "**Help me troubleshoot this error**", Copilot in Azure provides information about the error and the things that Copilot in Azure can check to help resolve the issue. In some cases, Copilot in Azure can even help fix the issue.

:::image type="content" source="media/improve-storage-accounts/storage-file-sync-error-troubleshoot.png" alt-text="Screenshot showing Microsoft Copilot in Azure troubleshooting an Azure File Sync error." lightbox="media/improve-storage-accounts/storage-file-sync-error-troubleshoot.png":::

## Reduce storage costs

You can ask Copilot in Azure to help you manage the costs for your storage accounts. One way to save on storage costs is by tiering blobs that haven't been accessed or modified for some time. In some instances, you may even wish to delete those blobs.

When you ask Copilot in Azure about reducing your storage costs, it provides suggestions and helps you accomplish bulk tiering by automating [lifecycle management rule](/azure/storage/blobs/lifecycle-management-overview#lifecycle-management-rule-definition) authoring.

### Storage cost reduction sample prompts

- "How can I save costs on my storage?"
- "Help me save costs on my storage"
- "I want to reduce the costs of my storage account"
- "Help me reduce the costs of my storage account"
- "I want to reduce the amount I am paying in storage"
- "Help me reduce the amount I’m paying in storage"
- "Help me write an LCM rule for my storage account"
- "I want to create an LCM rule for my storage account"
- "Create an LCM rule for my storage account"
- "Help me create an LCM rule for my storage account."

### Storage cost reduction example

If you say "**Help me lower my storage costs**", Copilot in Azure will begin the lifecycle management rule creation process.

:::image type="content" source="media/improve-storage-accounts/storage-costs-lower.png" alt-text="Screenshot of Microsoft Copilot in Azure responding to a request to lower storage costs.":::

If the context isn't clear, Copilot in Azure prompts you to select the storage account that you want to work with.  Next, you define a condition on which to trigger the rule.

:::image type="content" source="media/improve-storage-accounts/storage-costs-action-condition.png" alt-text="Screenshot of Microsoft Copilot in Azure confirming the action and condition for a storage account lifecycle management rule.":::

Finally, Copilot in Azure asks you whether you want the rule to run on the entire account, or only on a subset of the account. In this example, the rule will only run on a subset of the account, defined by a blob prefix.

:::image type="content" source="media/improve-storage-accounts/storage-costs-account.png" alt-text="Screenshot of Microsoft Copilot in Azure confirming where to run a new lifecycle management rule.":::

Now that Copilot in Azure has all of the necessary information, it creates the lifecycle management rule. You can copy and paste the rule yourself, or have Copilot in Azure create and apply the rule for you.

:::image type="content" source="media/improve-storage-accounts/storage-costs-lifecycle-management-rule.png" alt-text="Screenshot of Microsoft Copilot in Azure providing a customized lifecycle management rule for a storage account." lightbox="media/improve-storage-accounts/storage-costs-lifecycle-management-rule.png":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [Azure Storage](/azure/storage/common/storage-introduction).
