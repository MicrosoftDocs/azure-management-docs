---
title: "Troubleshoot Artifact Streaming in Azure Container Registry"
description: "Learn how to troubleshoot Artifact streaming in Azure Container Registry to diagnose and resolve issues with managing and deploying artifacts."
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: troubleshooting-general #Don't change.
ms.date: 10/31/2023

# Customer intent: "As a cloud administrator, I want to troubleshoot artifact streaming issues in the Azure Container Registry, so that I can effectively diagnose and resolve problems related to image retrieval and deployment in my AKS environment."
---

# Troubleshoot Artifact streaming

The troubleshooting steps in this article can help you resolve common issues that you might encounter when using artifact streaming in Azure Container Registry (ACR). These steps and recommendations can help diagnose and resolve issues related to artifact streaming as well as provide insights into the underlying processes and logs for debugging purposes.

## Symptoms

* Conversion operation failed due to an unknown error.
* Troubleshooting Failed AKS Pod Deployments.
* Pod conditions indicate "UpgradeIfStreamableDisabled."
* Digest usage instead of Tag for Streaming Artifact.

## Causes

* Issues with authentication, network latency, image retrieval, streaming operations, or other issues.
* Issues with image pull or streaming, streaming artifacts configurations, image sources, and resource constraints.
* Issues with ACR configurations or permissions.

## Conversion operation failed

| Error Code                  | Error Message                                                                      | Troubleshooting Info                                                                                                                                                                                                                                      |
| --------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| UNKNOWN_ERROR               | Conversion operation failed due to an unknown error.                               | Caused by an internal error. A retry helps here. If retry is unsuccessful, contact support.                                                                                                                                                                |
| RESOURCE_NOT_FOUND          | Conversion operation failed because target resource isn't found.                   | If the target image isn't found in the registry, verify typos in the image digest. If the image is deleted, or missing in the target region (replication consistency isn't immediate for example)                                                        |
| UNSUPPORTED_PLATFORM        | Conversion isn't currently supported for image platform.                          | Only linux/amd64 images are initially supported.                                                                                                                                                                                                          |
| NO_SUPPORTED_PLATFORM_FOUND | Conversion isn't currently supported for any of the image platforms in the index. | Only linux/amd64 images are initially supported. No image with this platform is found in the target index.                                                                                                                                                |
| UNSUPPORTED_MEDIATYPE       | Conversion isn't supported for the image MediaType.                               | Conversion can only target images with media type: application/vnd.oci.image.manifest.v1+json, application/vnd.oci.image.index.v1+json, application/vnd.docker.distribution.manifest.v2+json, or application/vnd.docker.distribution.manifest.list.v2+json |
| UNSUPPORTED_ARTIFACT_TYPE   | Conversion isn't supported for the image ArtifactType.                             | Streaming Artifacts (Artifact type: application/vnd.azure.artifact.streaming.v1) can't be converted again.                                                                                                                                                |
| IMAGE_NOT_RUNNABLE          | Conversion isn't supported for nonrunnable images.                                 | Only linux/amd64 runnable images are initially supported.                                                                                                                                                                                                 |

## Troubleshooting Failed AKS Pod Deployments

If AKS pod deployment fails with an error related to image pulling, like the following example.

```bash
Failed to pull image "mystreamingtest.azurecr.io/jupyter/all-spark-notebook:latest":
rpc error: code = Unknown desc = failed to pull and unpack image
"mystreamingtest.azurecr.io/latestobd/jupyter/all-spark-notebook:latest":
failed to resolve reference "mystreamingtest.azurecr.io/jupyter/all-spark-notebook:latest":
unexpected status from HEAD request to http://localhost:8578/v2/jupyter/all-spark-notebook/manifests/latest?ns=mystreamingtest.azurecr.io:503 Service Unavailable
```

To troubleshoot this issue, you should check the following guidelines:

1. Verify if the AKS has permissions to access the container registry `mystreamingtest.azurecr.io`.
1. Ensure that the container registry `mystreamingtest.azurecr.io` is accessible and properly attached to AKS.

## Checking for "UpgradeIfStreamableDisabled" Pod Condition:

If the AKS pod condition shows "UpgradeIfStreamableDisabled," check if the image is from an Azure Container Registry.

## Using Digest Instead of Tag for Streaming Artifact:

If you deploy the streaming artifact using digest instead of tag (for example, mystreamingtest.azurecr.io/jupyter/all-spark-notebook@sha256:4ef83ea6b0f7763c230e696709d8d8c398e21f65542db36e82961908bcf58d18), AKS pod event and condition message won't include streaming related information. However, you see fast container startup as the underlying container engine. This engine stream the image to AKS if it detects the actual image content is streamed. 

## Related content

> [!div class="nextstepaction"]
> [Artifact streaming](./container-registry-artifact-streaming.md)
