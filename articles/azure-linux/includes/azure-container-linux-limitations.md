---
ms.service: microsoft-linux
ms.topic: include
ms.date: 05/18/2025
author: schaffererin
ms.author: schaffererin
---

## Azure Container Linux (ACL) considerations and limitations

Before you begin, review the following considerations and limitations for ACL:

- ACL is generally available starting AKS v1.34.
- ACL requires [Trusted Launch](/azure/aks/use-trusted-launch) with Secure Boot and vTPM. Non-Trusted Launch variants aren't available.
- ACL on Arm64 requires Cobalt-based (v6) SKUs to enable Trusted Launch compatibility.
- `NodeImage` and `None` are the only supported [operating system (OS) upgrade channels](/azure/aks/auto-upgrade-node-os-image). `Unmanaged` and `SecurityPatch` are incompatible with ACL due to the immutable `/usr` directory.
- [Artifact Streaming](/azure/aks/artifact-streaming) isn't supported.
- [Pod Sandboxing](/azure/aks/use-pod-sandboxing) isn't supported.
- [Confidential Virtual Machines (CVMs)](/azure/aks/confidential-containers-overview) aren't supported.
- [Generation 1 VMs](/azure/aks/aks-virtual-machine-sizes#vm-support-on-aks) aren't supported.