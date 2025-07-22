---
title: "Secure your data in Azure Arc-enabled Kubernetes"
ms.date: 06/06/2025
ms.topic: concept-article
description: "Best practices for protecting data at rest and in transit in Azure Arc-enabled Kubernetes clusters."
---

# Secure your data

Protecting your data is essential for maintaining confidentiality, integrity, and availability in your Kubernetes environment. This article outlines strategies for securing secrets, encrypting data at rest and in transit, and planning for cluster recovery. By following these practices, you can help safeguard sensitive information and support business continuity.

## Use a vault to store your secrets and sync to the cluster as needed

Keep secret values (passwords, keys, etc.) in a vault, such as [Azure Key Vault](/azure/key-vault/general/overview). It's good practice for such secrets never to leave the vault, and to be used only for signing/crypto operations inside the vault itself. However, for on-premises clusters, you may not be able to reliably maintain your cloud connection to this vault and hence maintain your ability always to use secrets when they're needed. 

If so, consider using the [Azure Key Vault Secret Store extension for Kubernetes (preview)](/azure/azure-arc/kubernetes/secret-store-extension) ("SSE"). This extension can help automatically synchronize selected secrets from an Azure Key Vault and store them for offline use in the Kubernetes secrets store of an Azure Arc-enabled Kubernetes cluster. 

### References
* [CIS Kubernetes Benchmark - Sections 1, 2, and 4](https://www.cisecurity.org/benchmark/kubernetes)
* [NIST Application Container Security Guide - Section 4.5.1-3](https://csrc.nist.gov/pubs/sp/800/190/final)

## Protect the Kubernetes secrets store

If you’re running AKS enabled by Azure Arc on Azure Local, then it restricts direct access to the Kubernetes configuration store, etcd, which holds Kubernetes secrets. Further, it has a built-in [KMS plugin](/azure/aks/aksarc/encrypt-etcd-secrets) that helps to automatically encrypt these secrets. This plugin generates the [Key Encryption Key](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#kms-encryption-and-per-object-encryption-keys) (KEK), isolates it away from the cluster in the underlying Windows host, and automatically rotates it every 30 days.

If you connect your own cluster via Arc-enabled Kubernetes, then help ensure etcd is protected, and your secrets are encrypted, by following your vendor’s guidance.

### References
* [Section 2 of the CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
* [ NSA Kubernetes Hardening Guidance – "Secrets"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Securing data"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Protect other workload data

Beyond your secret values, consider the protection at rest of other workload data that may still be sensitive. If you’re running AKS on Azure Local, then all data volumes are [encrypted at rest using BitLocker](/azure/azure-local/concepts/security-features#bitlocker-encryption). If you connect your own cluster via Arc-enabled Kubernetes then use any similar protection offered by the vendor.

In addition to helping protect your workload data at rest, it’s also important to help protect your workload data in transit. See sections 2.5-2.7 above about establishing encryption, authentication, and authorization for data traffic between your workloads and to/from Azure. See also [our guidance](conceptual-secure-your-network.md) on adding further network layer protections. If you use [Azure Container Storage enabled by Azure Arc](/azure/azure-arc/container-storage/overview) to store data locally and synchronize it with Azure in the cloud, then it uses such transit protections automatically.

## Enable cluster recovery without impacting your security posture

Plan how you would recover from a loss of your cluster. If you’re using AKS enabled by Azure Arc on Azure Local or other high-availability options, then it can help protect against regular hardware failures. But it’s still possible that the cluster can be lost due to an incident that impacts your whole site or to a cyberattack.

A starting point is to aim for all your configuration and data to be sourced from, and synchronized back to, the cloud. This synchronization means that reinstating your cluster is like initial activation. Consider configuring your cluster using [GitOps with Flux](/azure/azure-arc/kubernetes/tutorial-use-gitops-flux2?tabs=azure-cli), synchronizing your Azure Key Vault secrets using the [Secret Store extension (preview)](/azure/azure-arc/kubernetes/secret-store-extension?tabs=arc-k8s), and synchronizing your data using [Azure Container Storage (preview)](/azure/azure-arc/container-storage/overview).

Alternatively, you may still need to consider using a dedicated cluster backup solution such as [Velero](https://velero.io/). Such as solution could be necessary if the only copy of some of your data is stored in your cluster or if you require extra protection.

## Next steps

- Learn how to [secure your network](conceptual-secure-your-network.md)
- Return to the top of this [security book](conceptual-security-book.md)