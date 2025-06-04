# Securing your network

## Configure Kubernetes network policy to control access to/from your workloads

You can help protect your cluster’s workload data traffic (workload) via TLS as described in sections and 2.6 above.

You can add also help further protect your workload data traffic by creating [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) that control the pods, namespaces, and IP addresses from which ingress requests can be received, and to which egress requests can be sent.  These policies are enforced by a Network Policy Engine.  If you are using AKS enabled by Azure Arc on Azure Local, then you can [deploy the Calico network plugin](/azure/aks/aksarc/concepts-container-networking#kubernetes-networks) to enable this (support for this is currently ‘as is’).  If you’ve connected your own cluster via Arc-enabled Kubernetes, then evaluate what networking (CNI) security capabilities they offer, and follow their guidance accordingly.

### References
* [Sections 5.3 of the CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)  
* [NSA Kubernetes Hardening Guidance – “Network policies”](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – “Use Kubernetes network policies to control traffic”](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Configure your network infrastructure for additional defense in depth

You can also create additional defense in depth, for both your management and data traffic, by appropriately configuring the network of your underlying infrastructure.  For example, if you’re using AKS enabled by Azure Arc on Azure Local, you should review the [guidance](/azure/aks/aksarc/aks-hci-network-system-requirements) for cluster IP address planning, and consider fully separating management and data traffic if your workloads don’t need to access the API server. 

Further, it’s recommended to evaluate your organization’s external firewall rules so that they are consistent with the rules you have set at the above Kubernetes and infrastructure layers: enabling only those outbound and inbound destinations strictly required, and no more.  You can also use [Azure Arc Gateway (preview)](/azure/azure-arc/kubernetes/arc-gateway-simplify-networking?tabs=azure-cli) to simplify the firewall rules needed to enable  your cluster’s access to Azure resources.

### References
* [Section 4.3.3, 4.3.4, and 4.4.2 of the NIST Application Container Security Guide](https://csrc.nist.gov/pubs/sp/800/190/final)
* [NSA Kubernetes Hardening Guidance – “Control plane hardening”](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

## Use Azure Private Link (preview) to access Azure resources

In addition to protecting your traffic to Azure resources by using TLS and Workload Identity Federation, as described in section 2.5  above, considering adding further defense in depth by using [Azure Private Link for Arc-enabled clusters (preview)](/azure/azure-arc/kubernetes/private-link).  This connects your cluster to Azure Arc, and other services such as Azure Key Vault, using private endpoints inside your cloud virtual network, which itself can be connected to your premises using [site-to-site VPN](/azure/vpn-gateway/tutorial-site-to-site-portal) or [ExpressRoute circuit}(/azure/expressroute/expressroute-howto-linkvnet-arm).  Evaluate the [advantages](/azure/azure-arc/kubernetes/private-link#advantages) and [current limitations](/azure/azure-arc/kubernetes/private-link#current-limitations) to decide if this solution works for you.