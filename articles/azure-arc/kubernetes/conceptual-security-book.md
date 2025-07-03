---
title: "Security book for Azure Arc-enabled Kubernetes and AKS enabled by Azure Arc"
ms.date: 06/06/2025
ms.topic: concept-article
description: "Comprehensive security guidance and best practices for securing Azure Arc-enabled Kubernetes and AKS enabled by Azure Arc clusters, covering platform, workloads, operations, data, and network."
---

# Security book for Azure Arc-enabled Kubernetes and AKS enabled by Azure Arc

> [!NOTE]
> This content is one of a series of security books, offering recommendations and best practices to help secure Microsoft platforms. Other security books include [Azure Local security book](https://github.com/Azure-Samples/AzureLocal/blob/main/SecurityBook/Azure%20Local%20Security%20Book_04302025.pdf), [the Windows Server 2025 security book](https://techcommunity.microsoft.com/blog/microsoft-security-blog/windows-server-2025-security-book/4283981), and the [Windows Client security book](/windows/security/book/).

Your organization may have Kubernetes deployments that include clusters running at the edge (on-premises in data centers, factories, shops) or in multiple clouds. It's often a challenge to maintain a consistent and scalable security posture in these heterogeneous environments. You can help address this challenge, and simplify your security workflows by running Microsoft-managed clusters using [Azure Kubernetes Service (AKS) enabled by Azure Arc](/azure/aks/hybrid/aks-overview) on your edge infrastructure, such as on [Azure Local](/azure/azure-local/overview). Or you can connect existing non-Microsoft edge clusters using [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/).

This book explains how these products can help you maintain a consistent and scalable security posture, covering both your cluster infrastructure and your application workloads. It advises how you can help guard against multiple threat vectors across supply chain risks, malicious external actors, or insider attacks. It builds on industry best practices such as those from:
- the [National Security Agency / Cybersecurity and Infrastructure Security Agency](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF) (NSA/CISA)
- the [Center for Internet Security (CIS)](https://www.cisecurity.org/benchmark/kubernetes)
- the [Open Worldwide application Security Project](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html) (OWASP)
- the [National Institute of Standards and Technology](https://csrc.nist.gov/pubs/sp/800/190/final) (NIST)
- the [Kubernetes project itself](https://kubernetes.io/docs/concepts/security/).
 
It also aligns with:
- the [Microsoft Threat Matrix for Kubernetes](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/)
- [Microsoft’s security advice for AKS cloud clusters](/azure/aks/concepts-security), expanding this advice to include extra considerations for edge clusters
- [Microsoft's security advice for IoT solutions](/azure/iot/iot-overview-security?tabs=edge).

If you're a security professional, this book surveys all these factors and makes multiple recommendations. If you're a leader or engineer with direct responsibility for Kubernetes development, deployment, or operations, it further points you towards detailed practical advice on steps you can take.

:::image type="content" source="media/conceptual-security-book/security-challenges.png" lightbox="media/conceptual-security-book/security-challenges.png" alt-text="Diagram showing the five categories of security challenges for edge Kubernetes.":::

The security challenges fall into five categories:
1. Secure your platform. This category includes configuring your Kubernetes cluster to operate more securely, and doing the same for its underlying OS and hardware infrastructure as needed, appropriately using all their built-in capabilities. 
1. Secure your workloads. This category includes building your containers more securely by following Kubernetes and Linux security standards. It also covers how to establish more secure authentication and authorization for requests to/from other services inside and outside of the clusters.
1. Secure your operations. This category includes controlling who can deploy to these clusters. It covers how to unify the cluster-local Kubernetes system for authentication and authorization with Microsoft Entra and Azure role-based access control (Azure RBAC) in the cloud. It also discusses how to better secure your software supply chain and enforce standards on your deployments through policies.
1. Secure your data. This category includes better securing access both to your workload application data and to the data that Kubernetes stores on your behalf, particularly secrets such as passwords.
1. Secure your network. This category includes configuring the extra defense in depth that comes from controlling management and data traffic at the network level. It discusses how to restrict which sources your clusters and workloads can receive from, and which targets they can they send to.

This book provides guidance on these challenges for Arc-enabled Kubernetes clusters in general and for AKS enabled by Azure Arc clusters in particular. There are many [deployment options](/azure/aks/aksarc/aks-overview#aks-enabled-by-azure-arc-deployment-options) for AKS enabled by Azure Arc. This book covers the [Azure Local 23H2](/azure/aks/aksarc/cluster-architecture) deployment option and builds upon the [Azure Local security features](/azure/azure-local/concepts/security-features) and [security book](https://github.com/Azure-Samples/AzureLocal/blob/main/SecurityBook/Azure%20Local%20Security%20Book_04302025.pdf). (Other deployment options offer some but not all of the same benefits.)

## Next steps

- Learn how to [secure your platform](conceptual-secure-your-platform.md)