---
title: "Secure your network in Azure Arc-enabled Kubernetes"
ms.date: 06/06/2025
ms.topic: concept-article
description: "Network security best practices for Azure Arc-enabled Kubernetes clusters, including segmentation, encryption, and access controls."
---

# Secure your network

Network security provides an additional layer of defense for your Kubernetes clusters. This article discusses how to use network policies, infrastructure configuration, and Azure-specific features to control traffic flow, segment workloads, and help protect communications. Implementing these measures helps reduce the attack surface and limit unauthorized access to your resources.

## Configure Kubernetes network policy to control access to/from your workloads

In addition to [helping to protect your cluster’s workload data traffic via TLS](conceptual-secure-your-workloads.md#configure-tls-encryption-and-authentication-withintofrom-workloads), you can help further protect it by creating [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/). These policies control the pods, namespaces, and IP addresses from which ingress requests can be received, and to which egress requests can be sent. You need to deploy a Network Policy Engine to enforce these policies. Evaluate if you can use the [Calico](https://docs.tigera.io/calico/latest/network-policy/get-started/calico-policy/calico-network-policy) or [Cillium](https://docs.cilium.io/en/latest/security/policy/index.html) engines in your cluster.

### References
* [CIS Kubernetes Benchmark - Sections 1, 2, and 4](https://www.cisecurity.org/benchmark/kubernetes)
* [NIST Application Container Security Guide - Section 4.5.1-3](https://csrc.nist.gov/pubs/sp/800/190/final)
* [NSA Kubernetes Hardening Guidance – "Network policies"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Use Kubernetes network policies to control traffic"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Configure your network infrastructure for further defense in depth

You can also create further defense in depth, for both your management and data traffic, by appropriately configuring the network of your underlying infrastructure. For example, if you’re using AKS enabled by Azure Arc on Azure Local, you should review the [guidance](/azure/aks/aksarc/aks-hci-network-system-requirements) for cluster IP address planning. Consider fully separating management and data traffic if your workloads don’t need to access the API server. 

Further, we recommend evaluating your organization’s external firewall rules so that they're consistent with the rules you set at the Kubernetes and infrastructure layers. Enable only those outbound and inbound destinations strictly required, and no more. You can also use [Azure Arc Gateway (preview)](/azure/azure-arc/kubernetes/arc-gateway-simplify-networking?tabs=azure-cli) to simplify the firewall rules needed to enable  your cluster’s access to Azure resources.

### References
* [NIST Application Container Security Guide - Section 4.3.3, 4.3.4, and 4.4.2](https://csrc.nist.gov/pubs/sp/800/190/final)
* [NSA Kubernetes Hardening Guidance – "Control plane hardening"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

## Use Azure Private Link (preview) to access Azure resources

In addition to helping protect your traffic to Azure with [TLS and Workload Identity Federation](conceptual-secure-your-workloads.md#use-workload-identity-for-accessing-azure-resources), also consider using [Azure Private Link for Arc-enabled clusters (preview)](/azure/azure-arc/kubernetes/private-link). This feature sets up private endpoints inside your cloud virtual network for Azure Arc, and other services such as Azure Key Vault. This network itself can then be connected to your premises using [site-to-site VPN](/azure/vpn-gateway/tutorial-site-to-site-portal) or [ExpressRoute circuit](/azure/expressroute/expressroute-howto-linkvnet-arm). Evaluate the [advantages](/azure/azure-arc/kubernetes/private-link#advantages) and [current limitations](/azure/azure-arc/kubernetes/private-link#current-limitations) to decide if this solution works for you.

## Next steps

- Return to the top of this [security book](conceptual-security-book.md)