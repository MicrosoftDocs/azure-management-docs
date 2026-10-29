---
title: "Secure your platform in Azure Arc-enabled Kubernetes"
ms.date: 07/17/2026
ms.topic: concept-article
ai-usage: ai-assisted
description: "Guidance on securing the platform layer of Azure Arc-enabled Kubernetes clusters, including node, OS, and control plane protections."
---

# Secure your platform

Securing your platform is foundational to improving the overall security of your Kubernetes environment. This article focuses on configuring your Kubernetes cluster, its underlying operating system, and hardware infrastructure to operate more securely. By leveraging built-in capabilities and following best practices, you can help ensure that your platform is more resilient against threats and provides a stronger base for your workloads and operations.

## Stay up to date with the latest security patches

We recommend upgrading your cluster and node OS with the latest security patches as they're published. In turn, this means it’s important to keep your cluster up to date with a supported version of Kubernetes, for which patches are released. Similarly, it's also important to keep your nodes up to date with a supported version of their OS.

If you're using AKS enabled by Azure Arc, you can apply upgrades on [Azure Local hyperconverged deployments](/azure/aks/aksarc/cluster-upgrade) or on [bare-metal deployments](/azure/aks/aksarc/aks-bare-metal-upgrade-cluster). You can also manage updates to the underlying Azure Local platform for [hyperconverged deployments](/azure/azure-local/update/about-updates-23h2) and [small form factor deployments](/azure/azure-local/small-form-factor/small-form-factor-resource-overview).

If you connect your own cluster via Arc-enabled Kubernetes, then keep it up to date following your vendor’s guidance, and [configure automatic upgrades](/azure/azure-arc/kubernetes/agent-upgrade) for the Arc-enabled Kubernetes agents that maintain the cluster’s connection to Azure.

You may also deploy [extensions](/azure/azure-arc/kubernetes/extensions) to your cluster: other sections of this security book recommend many of these extensions for the security benefits they can help bring. If so, configure automatic upgrades for these extensions too.

### References
* [NSA Kubernetes Hardening Guidance – "Upgrading and application security patches"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Securing Kubernetes Hosts"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Configure security protections on control plane and worker nodes

AKS enabled by Azure Arc automatically configures its control plane and worker nodes with more secure defaults. For protections provided by the underlying Azure Local platform, see the [security features for hyperconverged deployments](/azure/azure-local/concepts/security-features) and the [security features for small form factor deployments](/azure/azure-local/small-form-factor/small-form-factor-security) (preview). You can also review the [benefits of Azure Linux OS](/azure/azure-linux/intro-azure-linux#azure-linux-container-host-key-benefits), which AKS enabled by Azure Arc uses as its container host.

If you connect your own cluster via Arc-enabled Kubernetes, then confirm that your vendor can similarly help to automatically configure secure defaults across their hardware, OS, and Kubernetes stack. Evaluate if they have appropriate features such as a hardware root-of-trust, secure boot, and drive encryption. Also consider if [Microsoft Defender for Endpoint](/defender-endpoint) can help further protect your cluster nodes.

In addition, whether your cluster is fully Microsoft managed or you connect your own cluster, you can use Microsoft Defender for Containers. It helps [assess the health of your Kubernetes nodes](/azure/defender-for-cloud/kubernetes-nodes-va) and notify you of any issues. See the [support matrix](/azure/defender-for-cloud/support-matrix-defender-for-containers?tabs=azureva%2Carcrt%2Carcspm%2Carcnet) for which features are supported on which cluster types at which level (preview or general availability).

### References
* [CIS Kubernetes Benchmark - Sections 1, 2, and 4](https://www.cisecurity.org/benchmark/kubernetes)
* [NIST Application Container Security Guide - Section 4.5.1-3](https://csrc.nist.gov/pubs/sp/800/190/final)

## Protect the communication between Kubernetes control plane components

If you run multinode deployments of AKS enabled by Azure Arc, Transport Layer Security (TLS) protects all cross-node communication between control plane components, such as the API server and etcd. TLS certificates rotate regularly.

If you connect your own cluster via Arc-enabled Kubernetes, then determine if traffic between your nodes is similarly protected. Evaluate if there are any steps you need to take to enable this protection (for example, creating and updating certificates) by following your vendor’s guidance.

### References
* [NIST Application Container Security Guide - Section 4.3.5](https://csrc.nist.gov/pubs/sp/800/190/final)
* [NSA Kubernetes Hardening Guidance – "Encryption"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Restricting access to etcd"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Protect direct access to your nodes

In general, we don't recommend direct access to your cluster’s nodes. It’s best to administer your cluster via the API server, and Role-Based Access Control (RBAC) can help you control which users can perform which operations. See our [further guidance](conceptual-secure-your-operations.md#control-who-can-deploy-to-your-cluster-with-role-based-access-control-rbac) on this topic.

Disable SSH access to your worker nodes by default. If you need this access and you're running AKS enabled by Azure Arc, manage it carefully as follows.  (Beyond this limited exception, there should be no other way to reach the control plane nodes and the Kubernetes infrastructure components that run on them such as kube-scheduler, etcd, kubelet.)
- On Azure Local hyperconverged deployments, [safely store the SSH keys when you create the cluster](/azure/aks/aksarc/configure-ssh-keys) and [restrict SSH access to only expected network addresses](/azure/aks/hybrid/restrict-ssh-access).
- On Azure Local small form factor deployments, control SSH through just-in-time (JIT) access rather than creating standing administrative permissions. Grant eligible role assignments through Microsoft Entra Privileged Identity Management (PIM).  This means a user activates the role only when needed, for a limited time, and then connects over SSH using a short-lived certificate that expires automatically.

Because edge clusters often reside in nonsecure locations, consider what physical protections are appropriate, such as locked access and tamper-evident measures.

### References
* [NIST Application Container Security Guide - Section 4.5.4 and 4.5.5](https://csrc.nist.gov/pubs/sp/800/190/final)

## Next steps

- Learn how to [secure your workloads](conceptual-secure-your-workloads.md)
- Return to the top of this [security book](conceptual-security-book.md).
