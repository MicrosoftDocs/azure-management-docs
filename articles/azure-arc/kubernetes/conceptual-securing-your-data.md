# Securing your data
## Use a vault to store your secrets and sync to the cluster as needed
Keep secret values (passwords, keys, etc) in a vault, such as Azure Key Vault.  It is good practice for such secrets never to leave the vault, and to be used only for signing/crypto operations inside the vault itself.  However, for on-premises clusters, you may not be able to reliably maintain guarantee your cloud connection to this vault and hence your ability always to use secrets when they are needed.  
If so, consider using the Azure Key Vault Secret Store extension for Kubernetes (preview) ("SSE") that can help automatically synchronize selected secrets from an Azure Key Vault and store them in the Kubernetes secrets store of an Azure Arc-enabled Kubernetes cluster for offline use. 

### References
Reference: Sections 5.4 of the CIS Kubernetes Benchmark

## Protect the Kubernetes secrets store
Secrets stored in the Kubernetes secret store should be encrypted using a KMS plugin.  
If you’re running AKS enabled by Azure Arc on Azure Local, then access to the K8s configuration store, etcd, is restricted. Further, secrets stored in etcd are automatically encrypted by a built-in KMS plugin.  This plugin generates the Key Encryption Key (KEK), isolates it away from the cluster in the underlying Windows host, and automatically rotates it every 30 days.
[Note for future version of this book: add link to Leslie’s upcoming doc page for this, when published.]
If you’ve connected your own cluster via Arc-enabled Kubernetes, then help ensure etcd is protected, and your secrets are encrypted, by following your vendor’s guidance.
[Note for future versions of this book: Suggest contacting us if interested in KMS plugin for other distros – we have a private preview for K3s on Ubuntu.]

### References
Reference: Section 2 of the CIS Kubernetes Benchmark
Reference:  NSA Kubernetes Hardening Guidance – “Secrets”
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Securing data”

## Protect other workload data
Beyond your secret values, consider the protection at rest of other workload data that may still be sensitive.  If you’re running AKS on Azure Local, then all data volumes are encrypted at rest using BitLocker.  If you’ve connected your own cluster via Arc-enabled Kubernetes, or mounting external volumes, then use any similar protection offered by the vendor.
In addition to helping protect your workload data at rest, it’s also important to help protect your workload data in transit.  See sections 2.5-2.7 above about establishing encryption, authentication, and authorization for data traffic between your workloads and to/from Azure.  See also section 5 below for additional network layer protections.  Such transit protections are established automatically if you use Azure Container Storage enabled by Azure Arc to store data locally and synchronize it with Azure in the cloud.

## Enable cluster recovery without impacting your security posture
Plan how you would recover from a loss of your cluster.  If you’re using AKS enabled by Azure Arc on Azure Local or other high-availability options, then this can help protect against regular hardware failures, but it’s still possible that the cluster be lost due to an incident that impacts your whole site or to a cyberattack.
A starting point is to aim for all your configuration and data to be sourced from, and sync’d back to, the cloud.  This means that re-instating your cluster is like initial activation.  This might mean configuring your cluster using GitOps wth Flux, synchronizing your Azure Key Vault secrets using the Secret Store extension (preview), and synchronizing your data using Azure Container Storage (preview) – all as discussed in previous sections.
However, if the only copy of some of your data is stored in your cluster, and therefore won’t be recoverable from the cloud, or if you require additional protection, then consider using a dedicated cluster backup solution such as Velero.
