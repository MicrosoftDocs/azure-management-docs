# Securing your data

## Use a vault to store your secrets and sync to the cluster as needed

Keep secret values (passwords, keys, etc) in a vault, such as [Azure Key Vault](/azure/key-vault/general/overview). It is good practice for such secrets never to leave the vault, and to be used only for signing/crypto operations inside the vault itself. However, for on-premises clusters, you may not be able to reliably maintain guarantee your cloud connection to this vault and hence your ability always to use secrets when they are needed. 

If so, consider using the [Azure Key Vault Secret Store extension for Kubernetes (preview)](/azure/azure-arc/kubernetes/secret-store-extension) ("SSE") that can help automatically synchronize selected secrets from an Azure Key Vault and store them in the Kubernetes secrets store of an Azure Arc-enabled Kubernetes cluster for offline use. 

### References
* [CIS Kubernetes Benchmark - Sections 1, 2, and 4](https://www.cisecurity.org/benchmark/kubernetes)
* [NIST Application Container Security Guide - Section 4.5.1-3](https://csrc.nist.gov/pubs/sp/800/190/final)

## Protect the Kubernetes secrets store

Secrets stored in the Kubernetes secret store should be encrypted using a KMS plugin.  

If you’re running AKS enabled by Azure Arc on Azure Local, then access to the K8s configuration store, etcd, is restricted. Further, secrets stored in etcd are automatically encrypted by a built-in [KMS plugin](/azure/aks/aksarc/encrypt-etcd-secrets).  This plugin generates the [Key Encryption Key](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#kms-encryption-and-per-object-encryption-keys) (KEK), isolates it away from the cluster in the underlying Windows host, and automatically rotates it every 30 days.

If you’ve connected your own cluster via Arc-enabled Kubernetes, then help ensure etcd is protected, and your secrets are encrypted, by following your vendor’s guidance.

### References
* [Section 2 of the CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
* [ NSA Kubernetes Hardening Guidance – “Secrets”](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – “Securing data”](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Protect other workload data

Beyond your secret values, consider the protection at rest of other workload data that may still be sensitive. If you’re running AKS on Azure Local, then all data volumes are [encrypted at rest using BitLocker](/azure/azure-local/concepts/security-features?view=azloc-24113#bitlocker-encryption). If you’ve connected your own cluster via Arc-enabled Kubernetes, or mounting external volumes, then use any similar protection offered by the vendor.

In addition to helping protect your workload data at rest, it’s also important to help protect your workload data in transit. See sections 2.5-2.7 above about establishing encryption, authentication, and authorization for data traffic between your workloads and to/from Azure. See also section 5 below for additional network layer protections. Such transit protections are established automatically if you use [Azure Container Storage enabled by Azure Arc](/azure/azure-arc/container-storage/overview) to store data locally and synchronize it with Azure in the cloud.

## Enable cluster recovery without impacting your security posture

Plan how you would recover from a loss of your cluster. If you’re using AKS enabled by Azure Arc on Azure Local or other high-availability options, then this can help protect against regular hardware failures, but it’s still possible that the cluster be lost due to an incident that impacts your whole site or to a cyberattack.

A starting point is to aim for all your configuration and data to be sourced from, and sync’d back to, the cloud. This means that re-instating your cluster is like initial activation. This might mean configuring your cluster using [GitOps wth Flux](/azure/azure-arc/kubernetes/tutorial-use-gitops-flux2?tabs=azure-cli), synchronizing your Azure Key Vault secrets using the [Secret Store extension (preview)](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s), and synchronizing your data using [Azure Container Storage (preview)](/azure/azure-arc/container-storage/overview) – all as discussed in previous sections.

However, if the only copy of some of your data is stored in your cluster, and therefore won’t be recoverable from the cloud, or if you require additional protection, then consider using a dedicated cluster backup solution such as [Velero](https://velero.io/).

## Next steps

- Learn about [securing your network](conceptual-securing-your-network.md)
- Return to the top of this [security book](conceptual-security-book.md)