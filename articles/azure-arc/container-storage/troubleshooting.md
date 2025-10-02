---
title: Troubleshoot Azure Container Storage Enabled by Azure Arc
description: Learn how to diagnose and resolve common issues with Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: troubleshooting-general
ms.date: 09/25/2025
---

# Troubleshoot Azure Container Storage enabled by Azure Arc

This article helps you diagnose and resolve common issues when deploying and using Azure Container Storage enabled by Azure Arc.

## Installation issues

### Error occurred while creating custom resources needed by system extensions

**Error message:**
```
The extension operation failed with the following error: Error occurred while creating custom resources needed by system extensions : InnerError [Custom resource azure-arc-containerstorage is in a failed state]
```

This error may indicate a cert-manager is not deployed.
ACSA dependencies require a cert-manager is present prior to installation.
[Install cert-manager](quickstart-install.md#step-2-install-azure-iot-operations-dependencies)

After installing cert-manager, re-run the ACSA install operation.

### Failed Helm installation with BackoffLimitExceeded error

**Error message:**
```
Error: [ InnerError: [Helm installation failed :  : InnerError [release azure-arc-containerstorage failed, and has been uninstalled due to atomic being set: failed post-install: 1 error occurred:
	* job azure-arc-containerstorage-ensure-config-objects-job failed: BackoffLimitExceeded
```

This error typically indicates that one or more Kubernetes nodes do not meet the system requirements.
Please review [Prepare Linux for Edge Volumes](howto-prepare-linux-edge-volumes.md).

## Get help

If you continue to experience issues:

- Review the [FAQ](faq.yml) for additional common questions
- Check [release notes](release-notes.md) for known issues
- Visit [support and feedback](support-feedback.md) for assistance options
- For general Kubernetes extensions troubleshooting, visit: https://aka.ms/k8s-extensions-TSG

## Related content

- [Configure volumes and subvolumes](volumes-subvolumes.md)
- [Review best practices](storage-options.md)
