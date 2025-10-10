---
title: Azure Linux with OS Guard (preview) for Azure Kubernetes Service (AKS) Overview
description: Learn about Azure Linux with OS Guard for AKS.
author: florataagen
ms.author: florataagen
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 08/26/2025
# Customer intent: As a cloud architect, I want to understand the Azure Linux with OS Guard for AKS, so that I can determine its benefits and compatibility for securing my container workloads in the Azure environment.
---

# What is Azure Linux with OS Guard for AKS?

Azure Linux with OS Guard is a hardened, immutable variant of Azure Linux. It provides strong runtime integrity, tamper resistance, and enterprise-grade security for container hosts on Azure Kubernetes Service (AKS). Built on Azure Linux, OS Guard adds kernel and runtime features that enforce code integrity, protect the root file system from unauthorized changes, and apply mandatory access controls.

## Overview

Use OS Guard when you need elevated assurances about your container host and workload runtime. Key features include:

- **Immutability**: The /usr directory is mounted as a read-only volume protected by dm-verity. At runtime, the kernel validates a signed root hash to detect and block tampering.
- **Code Integrity**: OS Guard integrates the Integrity Policy Enforcement (IPE) Linux Security Module to ensure that only binaries from trusted, signed volumes are allowed to execute. This helps prevent tampered or untrusted code from executing, including within container images.
- **Mandatory access control**: SELinux operates in permissive mode to limit which processes can access sensitive resources in the system.
- **Measured boot and Trusted Launch**: OS Guard supports measured boot and integrates with Trusted Launch to provide cryptographic measurements of boot components stored in a virtual TPM (vTPM).
- **Verified Container Layers**: Container images and layers are validated using signed dm-verity hashes. This ensures that only verified layers are used at runtime, reducing the risk of container escape or tampering.
- **Sovereign Supply Chain Security**: OS Guard inherits Azure Linux’s secure build pipelines, signed Unified Kernel Images (UKIs), and Software Bill of Materials (SBOMs).

## Key advantages of Azure Linux with OS Guard

- **Strong runtime integrity guarantees**: Kernel-enforced immutability and IPE prevent execution of tampered or untrusted code.
- **Reduced attack surface**: A read-only userland and SELinux policy enforcement limit opportunities for an attacker to install persistent backdoors or alter system binaries.
- **Supply-chain trust**: Builds on Azure Linux’s signed images and supply-chain processes, delivering clear provenance for system components.
- **Integration with Azure security features**: Native support for Trusted Launch and Secure Boot provides measured boot protections and attestation.
-** Open-source transparency**: Many of the underlying technologies (dm-verity, SELinux, IPE) are upstreamed or open source; Microsoft has published tooling and contributions to support these features.
- **Compliance inheritence**: OS Guard inherits compliance properties from Azure Linux (for example, cryptographic modules and certifications available to Azure Linux), making it easier to adopt in regulated environments.

## Choosing an Azure Linux container host option

Azure Linux with OS Guard is built on Azure Linux and therefore benefits from the same supply-chain protections and signed images. Both OS variants can be appropriate depending on your security, compliance, and operational requirements:

| Container Host Option | Azure Linux Container Host | Azure Linux with OS Guard |
|---|---|---|
| **Security benefits** | Azure Linux provides the security benefits Microsoft views as critical for AKS workloads. | All the benefits of Azure Linux plus the additional security benefits mentioned above. |
| **User familiarity** | Familiar to customers coming from other Linux distributions like Ubuntu. Operations and tools customers use will feel familiar. | Familiar to customers coming from AWS Bottlerocket, Google CoS or other container optimized distributions. |
| **Target audience** | Targeted for customers doing lift and shifts, migrations and coming from other Linux distributions. | Targeted for cloud-native customers. Customers who are born in the cloud or who are looking to modernize. |
| **Security controls** | Option to enable Apparmor if necessary for security minded customers. | Security toggles like SELinux and IPE are permissive by default. |



## Get started and learn more

- Get started by [Creating an Azure Linux with OS Guard for AKS cluster using Azure CLI](./quickstart-osguard-azure-cli.md)
- Follow our tutorial to [Deploy, manage, and update applications](./tutorial-azure-linux-os-guard-create-cluster.md) with OS Guard


<!-- End of file -->


