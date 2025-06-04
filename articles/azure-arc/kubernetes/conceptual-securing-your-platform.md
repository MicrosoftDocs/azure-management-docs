# Securing your platform

## Stay up to date with the latest security patches

It is recommended to upgrade your cluster and node OS with the latest security patches as they are published.  In turn, this means it’s important to keep your cluster up to date with a supported version of Kubernetes for which patches are released, and similarly to keep your nodes up to date with a supported version of their OS.

If you’re using AKS enabled by Azure Arc on Azure Local, then it is easy to [apply such upgrades](/azure/aks/aksarc/cluster-upgrade).  It’s also easy to [upgrade Azure Local itself](/azure/azure-local/update/about-updates-23h2).

If you’ve connected your own cluster via Arc-enabled Kubernetes, then it is recommended to keep it up to date following your vendor’s guidance, and to configure automatic upgrades for the Arc-enabled Kubernetes agents that maintain the cluster’s connection to Azure.

You may also deploy extensions to your cluster: many of them are recommended below for the security benefits they can help bring.  If so, configure automatic upgrades for these extensions too.

### References
Reference: NSA Kubernetes Hardening Guidance – “Upgrading and application security patches”
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Securing Kubernetes Hosts”

## Configure security protections on control plane and worker nodes

AKS enabled by Azure Arc on Azure Local automatically configures its control plane and worker nodes with more secure defaults, activating the appropriate security features in the underlying hardware, in Windows host OS, and the Linux VM nodes and filesystem.  Read the [Azure Local security book](/azure/azure-local/concepts/security-features?view=azloc-24113) to learn more about how to secure this platform, and review the [benefits of Azure Linux OS](/azure/azure-linux/intro-azure-linux#azure-linux-container-host-key-benefits), which AKS enabled by Azure Arc uses as its container host.

If you’ve connected your own cluster via Arc-enabled Kubernetes, then confirm that your vendor can similarly help to automatically configure secure defaults across their hardware, OS, and Kubernetes stack and that they have appropriate features such as a hardware root-of-trust, secure boot, and drive encryption.  Also consider if [Microsoft Defender for Endpoint](https://learn.microsoft.com/defender-endpoint/) can help further protect your cluster nodes.

In addition, whether your cluster is fully Microsoft managed or you’ve connected your own cluster, you can use Microsoft Defender for Containers to help [assess the health of your Kubernetes nodes](/azure/defender-for-cloud/kubernetes-nodes-va) and notify you of any issues – see the [support matrix](/azure/defender-for-cloud/support-matrix-defender-for-containers?tabs=azureva%2Carcrt%2Carcspm%2Carcnet) for which features are supported on which cluster types at which level (preview or general availability).

### References
Reference: Sections 1, 2, and 4 of the CIS Kubernetes Benchmark
Reference: Section 4.5.1-3 of the NIST Application Container Security Guide

## Protect the communication between Kubernetes control plane components

If you’re running AKS enabled by Azure Arc on Azure Local, then Transport Layer Security (TLS) is used to help protect all cross-node communication between control plane components such as the API server and etcd, with certificates that are regularly rotated.

If you’ve connected your own cluster via Arc-enabled Kubernetes, then determine if traffic between your nodes is similarly protected, and if there are any steps you need to take to enable this (e.g. creating and updating certificates), by following your vendor’s guidance.

### References
Reference: Section 4.3.5 of the NIST Application Container Security Guide
Reference:  NSA Kubernetes Hardening Guidance – “Encryption”
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Restricting access to etcd”

## Protect direct access to your nodes

In general, it’s not recommended to directly access your cluster’s nodes.  It’s best to administer your cluster via the API server, and Role-Based Access Control (RBAC) can help you control which users can perform which operations. See section 3.2 below for more on this.

Therefore, SSH access to your worker nodes should be disabled by default.  However, if this does prove to be required, and you’re running AKS enabled by Azure Arc on Azure Local, then it’s important to [carefully manage the SSH keys when creating the cluster](/azure/aks/aksarc/configure-ssh-keys) and [restrict SSH access to only expected network addresses](/azure/aks/hybrid/restrict-ssh-access). Beyond this, there should be no other way to reach the control plane nodes and the Kubernetes infrastructure components that run on them such as kube-scheduler, etcd, kubelet.

Finally, because edge clusters often reside in non-secure locations, consider what physical protections are appropriate (locked access, tamper evident measures, etc).

### References
Reference: Section 4.5.4 and 4.5.5 of the NIST Application Container Security Guide
