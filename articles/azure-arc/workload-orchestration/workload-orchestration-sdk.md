---
title: Workload Orchestration SDK Overview
description: Learn about the Azure Workload Orchestration SDK and its capabilities.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: overview
ms.date: 10/06/2025
---

# Workload orchestration SDK overview

The Azure workload orchestration SDK provides programmatic access to workload orchestration capabilities, enabling you to automate tasks like schema creation, target configuration, solution deployment, and context management, using familiar programming languages. This SDK is designed to simplify integration with orchestration workflows and improve developer productivity across environments.

## Supported languages

The SDK is available in the following languages:

- [Python](pypi.org/project/azure-mgmt-workloadorchestration)
- [Java](search.maven.org/artifact/com.azure.resourcemanager/azure-resourcemanager-workloadorchestration)
- [JavaScript (Node.js)](npmjs.com/package/@azure/arm-workloadorchestration)
- [Go](pkg.go.dev/github.com/Azure/azure-sdk-for-go/.../armworkloadorchestration)

Each implementation includes language-specific examples and utilities to help you get started quickly.

## Key capabilities

- Creation and management of context, targets, schema and solution templates
- Configuration, publishing and deployment of solutions
- Support for asynchronous operations and automatic retries
- Comprehensive logging for enhanced observability

## Get started 

To begin using the workload orchestration SDK:

1. Clone the [Azure Workload Orchestration SDK Example repository](https://github.com/atharvau/Azure-Workload-Orchestration-SDK-Example/).
1. Navigate to the folder for your preferred language.
1. Follow the setup instructions in the README file.
1. Authenticate using Azure CLI or service principal credentials.
1. Run sample scripts to validate your environment and begin orchestration.



