---
title: Support and feedback for Azure Container Storage enabled by Azure Arc
description: Learn how to get support and provide feedback on Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 08/26/2024

# Customer intent: As a cloud administrator, I want to efficiently request support for Azure Container Storage enabled by Azure Arc, so that I can address issues promptly and maintain the functionality of my Kubernetes cluster.
---

# Support and feedback for Azure Container Storage enabled by Azure Arc

If you experience an issue or need support, see the following video and steps to request support for Azure Container Storage enabled by Azure Arc in the Azure portal:

> [!VIDEO f477de99-2036-41a3-979a-586a39b1854f]

1. Navigate to the desired Arc-enabled Kubernetes cluster with the Azure Container Storage enabled by Azure Arc extension that you are experiencing issues with.
1. To expand the menu, select **Settings** on the left blade.
1. Select **Extensions**.
1. Select the name for **Type**: `microsoft.arc.containerstorage`. In this example, the name is `hydraext`.
1. Select **Help** on the left blade to expand the menu.
1. Select **Support + Troubleshooting**.
1. In the search text box, describe the issue you are facing in a few words.
1. Select "Go" to the right of the search text box.
1. For **Which service you are having an issue with**, make sure that **Edge Storage Accelerator - Preview** is selected. If not, you might need to search for **Edge Storage Accelerator - Preview** in the drop-down.
1. Select **Next** after you select **Edge Storage Accelerator - Preview**.
1. **Subscription** should already be populated with the subscription that you used to set up your Kubernetes cluster. If not, select the subscription to which your Arc-enabled Kubernetes cluster is linked.
1. For **Resource**, select **General question** from the drop-down menu.
1. Select **Next**.
1. For **Problem type**, from the drop-down menu, select the problem type that best describes your issue.
1. For **Problem subtype**, from the drop-down menu, select the subtype that best describes your issue. The subtype options vary based on your selected **Problem type**.
1. Select **Next**.
1. Based on the issue, there might be documentation available to help you triage your issue. If these articles are not relevant or don't solve the issue, select **Create a support request** at the top.
1. After you select **Create a support request at the top**, the fields in the **Problem description** section should already be populated with the details that you provided earlier. If you want to change anything, you can do so in this window.
1. Select **Next** once you verify that the information in the **Problem description** section is accurate.
1. In the **Recommended solution** section, recommended solutions appear based on the information you entered. If the recommended solutions are not helpful, select **Next** to continue filing a support request.
1. In the **Additional details** section, populate the **Problem details** with your information.
1. Once all required fields are complete, select **Next**.
1. Review your information from the previous sections, then select **Create**.

## Release notes

See the [release notes for Azure Container Storage enabled by Azure Arc](release-notes.md) for information about new features and known issues.

## Next steps

[What is Azure Container Storage enabled by Azure Arc?](overview.md)
