---
title: Find your recommendations using Microsoft Copilot in Azure
description: Learn how to find your recommendations using Microsoft Copilot in Azure.
ms.date: 11/08/2024
ms.topic: conceptual
ms.service: copilot-for-azure
ms.author: v-josmartin
author: jm-247-ms

---

# Find your recommendations using Microsoft Copilot in Azure

Microsoft Copilot in Azure (preview) directs you to your Azure Advisor recommendations across the Well Architected Framework pillars. The Well Architected Framework pillars include Reliability, Cost Optimization, Operational Excellence, Performance Efficiency, and Security. To learn more about the Well Architected Framework, see [Azure Well-Architected Framework](/azure/well-architected "Azure Well-Architected Framework | Microsoft Learn").

When you ask Microsoft Copilot in Azure questions about your recommendations, refine the scope of your request about recommendations by including the following information.

*  Pillar

*  Impact

*  Subscription

*  Resource selection panel

> [!NOTE]
> To view a few of the areas for which Microsoft Copilot in Azure (preview) is especially helpful, see [Sample prompts](#sample-prompts). The list is not a complete list of potential things. The Azure Advisor team encourages you to experiment with your own prompts and see how Microsoft Copilot in Azure (preview) helps you manage your Azure resources and environment.

> [!IMPORTANT]
> Microsoft Copilot in Azure (preview) is currently in PREVIEW. To review the legal terms that apply to Azure features that are not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms "Supplemental Terms of Use for Microsoft Azure Previews | Microsoft Azure").

## Sample prompts

The following prompts are examples for Advisor recommendations. Modify the prompts to align with your real-life scenarios. Try other prompts to create new queries.

```text
Show me my reliability recommendations
```

```text
Show me my Advisor recommendations
```

```text
What are the top cost saving recommendations?
```

```text
How do I improve performance?
```

```text
Show me my security recommendations
```

## Example

1.  Prompt Copilot with following text.

    ```text
    Show me my reliability recommendations
    ```

In Copilot, Copilot asks you if you want to pick specific resources for which to view the associated recommendations.

*   If you don't want to pick specific resources, select `No`.

    Copilot returns a list of top reliability recommendations for you that includes links to each recommendation page in Advisor.

    <!--
    :::image alt-text="Screenshot of ..." lightbox="./media/---.png" source="./media/----preview.png" type="content":::
    -->

*   If you do want to pick specific resources, select `Yes`.

    Copilot shows the **Select Resources** panel.

On the **Select Resources** panel.

See all of the resources to which you have access.

1.  Filter resources by Subscription, Resource group, Type, Location to scope the resources you want to target.

1.  Search using text for a resource.

1.  Select one or more resources.

    Copilot returns a list of recommendations for the resources.

    <!--
    :::image alt-text="Screenshot of ..." lightbox="./media/---.png" source="./media/----preview.png" type="content":::
    -->

## Related articles

*   [Microsoft Copilot in Azure capabilities](/azure/copilot/capabilities "Microsoft Copilot in Azure capabilities | Microsoft Copilot in Azure (preview) | Microsoft Learn")

*   [Write effective prompts for Microsoft Copilot in Azure](/azure/copilot/write-effective-prompts "Write effective prompts for Microsoft Copilot in Azure | Microsoft Copilot in Azure (preview) | Microsoft Learn")

*   [Introduction to Azure Advisor](/azure/advisor/advisor-overview "Introduction to Azure Advisor | Azure Advisor | Microsoft Learn")