---
title: Evaluate the Edge RAG System
description: "Learn how to evaluate the functionality of the RAG system after you configure the chat solution in Edge RAG."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 06/05/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer Intent: As a developer working with Edge RAG, I want to evaluate the system, models, and datasets using baseline or automatic evaluations so that I can ensure the functionality, quality, and performance of the RAG system for my chat solution.
---
# Evaluate the Edge RAG Preview system

Evaluate the system, models, and datasets within Edge RAG Preview, enabled by Azure Arc. There are two types of evaluations: baseline, and automatic.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- Review [Metrics for evaluating the Edge RAG system](evaluation-metrics.md).
- To access to the developer portal, you must have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Run baseline check

The baseline check evaluates the functionality of the RAG system to make sure it's working as expected. It runs the following tasks:

- Creates an ingestion build in the documents dataset.
- Inferences by using the build of a test dataset that includes set of queries and expected answers.
- Evaluates system based on model metrics.

To run a baseline check:

1. Go to the developer portal using the domain name provided at deployment and app registration. For example: `https://arcrag.contoso.com`.
1. Sign in with developer credentials that have both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned.
1. Select the **Evaluation** tab.

   :::image type="content" source="media/evaluate-solution/evaluation-tab.png" alt-text="A screenshot showing the Evaluation tab in the developer portal, highlighting options for running checks and managing evaluations." lightbox="media/evaluate-solution/evaluation-tab.png":::

1. On the **Baseline check** tab, select **Run a check**.
1. Enter a name for your evaluation.

   :::image type="content" source="media/evaluate-solution/baseline-evaluation-check.png" alt-text="A screenshot showing the Evaluation tab in the developer portal, with options for running checks and managing evaluations.":::

1. Select **Run**.

1. Review the evaluation status.

   :::image type="content" source="media/evaluate-solution/evaluation-status.png" alt-text="A screenshot showing the evaluation status page in the developer portal, displaying the progress and details of the baseline check.":::

1. When the evaluation is completed, select the name to see the results.

   :::image type="content" source="media/evaluate-solution/baseline-results.png" alt-text="A screenshot showing the evaluation results, including metrics and detailed performance analysis of the RAG system.":::

## Run automatic evaluation

The automatic evaluation evaluates the quality of the RAG system by using your own documents and dataset.

1. In the developer portal, select **Evaluation** > **Automatic evaluation**.

   :::image type="content" source="media/evaluate-solution/automatic-evaluation-tab.png" alt-text="Screenshot of the Automatic Evaluation tab in the developer portal with options for creating evaluations.":::

1. Select **Create an automated evaluation**.
1. Enter a name for your evaluation.

   :::image type="content" source="media/evaluate-solution/automated-evaluation-basic-info.png" alt-text="A screenshot of the basic information tab, with fields for entering the evaluation name and configuration options." lightbox="media/evaluate-solution/automated-evaluation-basic-info.png" :::

1. Review the parameters like **Temperature**, **Top-N**, **Top-P**, and **System prompt**. These parameters are derived from the **Chat playground**. To change the parameters, go to the **Chat** tab and change them as needed.
1. Select **Next**.
1. Under **Test dataset**, select **Download dataset sample** to get familiar with the required structure of the test dataset JSONL format.

   :::image type="content" source="media/evaluate-solution/test-dataset.png" alt-text="Screenshot of the test dataset tab  where you can download a template and update the dataset.":::

1. Upload your dataset JSONL file.
1. Select **Next**.
1. Select the Metrics you want to evaluate for your RAG system.

   :::image type="content" source="media/evaluate-solution/select-metrics.png" alt-text="A screenshot that shows the available metrics to evaluate your system."lightbox="media/evaluate-solution/select-metrics.png":::
1. Select **Next**.
1. Review the configurations and select **Create**.

   :::image type="content" source="media/evaluate-solution/review-configurations.png" alt-text="Screenshot of the tab that summarizes your configuration for the automatic evaluation." lightbox="media/evaluate-solution/review-configurations.png":::

1. Monitor the progress and the status of the evaluation.

   :::image type="content" source="media/evaluate-solution/evaluation-status-automated.png" alt-text="Screenshot shows the results of an automatic evaluation, including metrics and evaluation details.":::

1. After the evaluation completes, review the results by selecting on the evaluation name.

   :::image type="content" source="media/evaluate-solution/evaluation-results.png" alt-text="Screenshot of the evaluation results page in the developer portal, displaying metrics, and performance analysis for the RAG system." lightbox="media/evaluate-solution/evaluation-results.png" :::

1. Review the evaluation details and metrics.

   :::image type="content" source="media/evaluate-solution/evaluation-details.png" alt-text="Screenshot of the evaluation details page in the developer portal, showing metrics, configurations, and detailed analysis for the RAG system." lightbox="media/evaluate-solution/evaluation-details.png":::

## Related content

- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- [Set up the data query for Edge RAG chat solution](set-up-data-query.md)