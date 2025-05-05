---
title: Use Microsoft Copilot in Azure with AI Shell
description: Learn about the things you can do with Microsoft Copilot in Azure from the command line.
ms.date: 04/08/2025
ms.topic: concept-article
ms.service: copilot-for-azure
ms.author: sewhee
author: sdwheeler
---

# Use Microsoft Copilot in Azure with AI Shell

AI Shell introduced a seamless way to access Copilot in Azure directly from your command-line
interface (CLI). This enhancement allows developers, administrators, and DevOps teams to generate
Azure CLI and PowerShell commands faster and more intuitively, using natural language to interact
with Copilot in Azure right in your preferred terminal environment.

## What's AI Shell?

[AI Shell][01] is an interactive shell that provides a chat interface for AI language models. The
shell provides agents that connect to different AI models and other assistance providers. Users can
interact with the agents in a conversational manner. With AI Shell, you can access Copilot in
Azure's AI-driven suggestions, using natural language prompts, directly in your CLI environment to:

- Quickly resolve common Azure CLI and PowerShell errors with guided suggestions
- Simplify your Azure workflows
- Automate complex setups
- Gain insights into best practices

![Animation showing the Azure agent in action.][03]

### Key use cases

- **Command generation**: Generate Azure CLI or PowerShell commands by describing what you need in
  natural language directly from the CLI. For example, ask AI Shell to:

  `Set up a virtual network in West US with three subnets`

  Copilot in Azure provides you with the exact command needed in the AI Shell terminal.

- **Enhanced learning**: If you're new to Azure CLI or PowerShell, Copilot in Azure's command
  suggestions can serve as an educational tool, guiding you through syntax and structure to achieve
  desired outcomes without extensive knowledge of command syntax.

- **Troubleshooting and optimization**: Copilot in Azure can help diagnose and suggest improvements
  to existing command structures, ensuring your commands align with best practices and efficient
  configurations.

## How to get started

To get started, you must install AI Shell and the **AIShell** PowerShell module. You can find system
requirements and installation instructions in the AI Shell [documentation][01].

## How to provide feedback

As part of Azure's commitment to continuous improvement, your feedback is crucial to refining
Copilot in Azure's CLI capabilities in AI Shell. You can provide feedback directly within the AI
Shell interface using the `/like` and `/dislike` commands to provide feedback on the responses
you're given. This feedback helps us enhance the accuracy and relevance of Copilot in Azure's
recommendations. You share suggestions, report issues, and help shape future releases by submitting
feedback in the AI Shell [GitHub repository][02]. For detailed feedback instructions, see the AI
Shell [documentation][01].

<!-- link references -->
[01]: /powershell/utility-modules/aishell/overview
[02]: https://github.com/PowerShell/ProjectMercury
[03]: media/ai-shell-overview/ai-shell-azure-agent.gif
