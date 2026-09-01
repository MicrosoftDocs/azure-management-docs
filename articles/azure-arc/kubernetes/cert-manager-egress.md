---
title: "Egress support for Certificate Management for Azure Arc-enabled Kubernetes"
ms.date: 08/28/2026
ms.topic: how-to
description: "Learn how to secure external egress traffic when using Certificate Management for Azure Arc-enabled Kubernetes."

# Customer intent: As a customer using Azure Arc-enabled Kubernetes, I want to understand how to configure  Certificate Management for Azure Arc-enabled Kubernetes on egress scenarios, so that I can secure outgoing external traffic from my Arc-enabled Kubernetes clusters.
---

# Egress support for Certificate Management for Azure Arc-enabled Kubernetes

The Certificate Management for Azure Arc extension provides a cluster-level mechanism for managing certificates and distributing trusted CA material to workloads running inside an Azure Arc-enabled Kubernetes cluster.

In many scenarios, Kubernetes workloads must initiate outbound (egress) connections to systems that live outside the cluster boundary, such as factory devices, on‑premises services, or enterprise infrastructure. These external systems often use TLS certificates issued by private or enterprise Certificate Authorities (CAs) rather than publicly trusted CAs.

For these egress scenarios, the Certificate Management for Azure Arc extension doesn't terminate TLS or proxy outbound traffic. Instead, it focuses on:

- Managing trust material (CA certificates) required by workloads.
- Ensuring trust can be consistently distributed across namespaces and pods.
- Integrating with existing Kubernetes and cert‑manager primitives.

At a high level, the model is:

- The cluster administrator defines which external CAs to trust.
- The Certificate Management for Azure Arc extension distributes this trust material inside the cluster.
- Application pods consume the trust material and use it during outbound TLS handshakes.

This model keeps TLS enforcement and protocol semantics in the application, while the Certificate Management for Azure Arc extension handles trust distribution and lifecycle.

## Trust bundles and how workloads use them

A trust bundle is a collection of one or more CA certificates that represent the roots (and optionally intermediates) a workload should trust.

In an egress TLS scenario:

- The trust bundle typically contains the CA chain that the external system uses.
- You manage the bundle centrally, but workloads consume it locally.
- If workloads connect to the same external PKI, they can share the same trust bundle.
- The Certificate Management for Azure Arc extension uses trust-manager to distribute these bundles in a Kubernetes-native way. This approach avoids ad-hoc, per-pod certificate copying and keeps trust configuration declarative.

Using the Certificate Management for Azure Arc extension doesn't change application behavior. Workloads still perform TLS the same way they would outside Kubernetes.

Conceptually, a pod:

- Receives a trust bundle through a mounted file or environment-level configuration
- Configures its TLS client stack to trust that bundle
- Initiates outbound TLS connections as normal
- Testing and debugging at a high level

## Egress TLS validation

At a high level, validating egress TLS involves:

- Verifying the workload has the trust bundle
- Confirming the external service presents a certificate chain rooted in that bundle
- Ensuring application TLS libraries are configured to use the provided trust material

Common failure modes include:

- Missing or outdated CA certificates
- Incorrect certificate chains on the external system
- Application clients not reloading trust after updates

The Certificate Management for Azure Arc extension provides observability signals around certificate and trust distribution, but connection-level failures surface in application logs.
