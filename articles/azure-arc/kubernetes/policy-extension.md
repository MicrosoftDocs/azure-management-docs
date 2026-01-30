---
title: "Azure Policy extension Installation"
ms.date: 01/28/2026
ms.topic: how-to
description: "Install Azure Policy extension in Kubernetes clusters connected via Azure Arc"
# Customer intent: As a DevOps engineer, I want to understand which Kubernetes distributions have passed conformance tests for Azure Policy extension for Azure Arc, so that I can ensure compatibility and successful integration for managing my clusters across cloud environments.
---
# Azure Policy Extension for Azure Arc

This page documents **supported Kubernetes distributions for the Azure Policy Extension for Azure Arc**.

> **Important**: This documentation applies **only to the Azure Policy Extension**. It does not describe general Azure Arc–enabled Kubernetes support.

The tables below indicate where the Azure Policy Extension has been validated through conformance testing versus where installation is supported but conformance validation has not yet been completed.

---

## Distributions with Conformance Validation

The following Kubernetes distributions **have passed conformance testing**. This means we have explicitly validated that the **Azure Policy Extension installs correctly and functions as expected** on these platforms.

| Provider Name | Distribution Name | Kubernetes Version Required |
|--------------|------------------|-----------------------------|
| Azure | aks_workload | 1.27+ |
| Red Hat | openshift | 1.27+ |
| CNCF | kind | 1.27+ |
| SUSE/Rancher | rke2 | 1.27+ |
| CNCF | minikube | 1.27+ |
| Rancher | k3s | 1.27+ |

---

## Supported Distributions (No Conformance Validation)

The Azure Policy Extension **can be installed** on the following Kubernetes distributions; however, **conformance testing has not been completed**.

> Installation is supported, but **there is no guarantee of full functionality** or behavioral consistency until conformance validation is complete.

| Provider Name | Distribution Name | Kubernetes Version Required |
|--------------|------------------|-----------------------------|
| Azure | aks_edge_k3s | 1.27+ |
| Azure | aks_edge_k8s | 1.27+ |
| AWS | eks | 1.27+ |
| Google | gke | 1.27+ |
| Rancher | rancher_rke | 1.27+ |
| VMware | tkg | 1.27+ |
| Any | generic | 1.27+ |


---

## Azure Red Hat OpenShift (ARO) Considerations

Azure Red Hat OpenShift (ARO) clusters ship with **Guardrails pre-installed**. These guardrails **conflict with the Azure Policy Extension** and must be disabled before installation.

To disable ARO Guardrails, run the following commands **in order**:

```bash
oc patch cluster.aro.openshift.io cluster --type json -p '[{ "op": "replace", "path": "/spec/operatorflags/aro.guardrails.deploy.managed", "value":"false" }]'

oc get all -n openshift-azure-guardrails

oc patch cluster.aro.openshift.io cluster --type json -p '[{ "op": "replace", "path": "/spec/operatorflags/aro.guardrails.enabled", "value":"false" }]'
```

Once guardrails are disabled, you may proceed with installing the Azure Policy Extension.

---

## Azure Policy Extension Release Notes

### 1.16.1

Fixed policy extension installation bug in aks_workload distribution
Added RKE2 support.
Enabled mutation.
Enabled external data.
 - Released Jan 2026
 - Policy Image v1.15.4
 - Gatekeeper v3.21.0-1

### 1.16.0
Added RKE2 support.
Enabled mutation.
Enabled external data.
 - Released Jan 2026
 - Policy Image v1.15.4
 - Gatekeeper v3.21.0-1

> Release notes in this document are **specific to the Azure Policy Extension** and do not reflect other extensions in Azure Arc.

---