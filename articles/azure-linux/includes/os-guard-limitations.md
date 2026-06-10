---
ms.service: microsoft-linux
ms.topic: include
ms.date: 05/18/2025
author: schaffererin
ms.author: schaffererin
---

## Azure Linux with OS Guard considerations and limitations

Before you begin, review the following considerations and limitations for Azure Linux with OS Guard (preview):

- Kubernetes version 1.32.0 or higher is required for Azure Linux with OS Guard.
- All Azure Linux with OS Guard images have [Federal Information Process Standard (FIPS)](/azure/aks/enable-fips-nodes) and [Trusted Launch](/azure/aks/use-trusted-launch) enabled.
- Azure CLI and ARM templates are the only supported deployment methods for Azure Linux with OS Guard on AKS in preview. PowerShell and Terraform aren't supported.
- [Arm64](/azure/aks/use-arm64-vms) images aren't supported with Azure Linux with OS Guard on AKS in preview.
- `NodeImage` and `None` are the only supported [operating system (OS) upgrade channels](/azure/aks/auto-upgrade-node-os-image) for Azure Linux with OS Guard on AKS. `Unmanaged` and `SecurityPatch` are incompatible with Azure Linux with OS Guard due to the immutable /usr directory.
- [Artifact Streaming](/azure/aks/artifact-streaming) isn't supported.
- [Pod Sandboxing](/azure/aks/use-pod-sandboxing) isn't supported.
- [Confidential Virtual Machines (CVMs)](/azure/aks/confidential-containers-overview) aren't supported.
- [Gen 1 virtual machines (VMs)](/azure/aks/aks-virtual-machine-sizes#vm-support-on-aks) aren't supported.