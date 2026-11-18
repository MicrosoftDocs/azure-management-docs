---
title: Azure Linux compliance overview
description: Understand the Azure Linux compliance boundary, review CIS Benchmark and FIPS status, and prepare evidence for an assessment.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Azure Linux compliance overview

This page describes the compliance status of Azure Linux 4 and how to prepare
for an assessment. Assessments must cover the applicable platform, guest,
workload, and operational controls.

## Understand the compliance boundary

Azure and the workload owner control different parts of a deployed system.
Document both sides when you define the assessment scope.

| Area | Responsibility |
| ---- | -------------- |
| Azure platform | Azure datacenter, physical infrastructure, and applicable Azure service controls and authorizations. |
| Azure resource configuration | Subscription policy, identity, networking, managed disk encryption, monitoring, backup, and resource governance. |
| Guest operating system | Azure Linux packages, security settings, accounts, services, audit rules, cryptographic policy, and update state. |
| Workload | Application code, dependencies, secrets, data handling, access policy, logging, and recovery procedures. |
| Operations | Change control, vulnerability management, incident response, evidence retention, exceptions, and periodic reassessment. |

An Azure authorization doesn't automatically cover every guest or workload
configuration. Similarly, applying a security profile inside the guest doesn't
establish that the complete deployment meets a compliance program.

## Compliance status

| Area | Status |
| ---- | ------ |
| Center for Internet Security (CIS) Benchmark | The general-purpose Azure Linux 4 image meets the Level 1 - Server profile of the CIS Azure Linux 4 Benchmark by default. Get the benchmark from [CIS Azure Linux Benchmarks](https://www.cisecurity.org/benchmark/azure_linux). |
| Federal Information Processing Standards (FIPS) 140-3 | In progress. Azure Linux 4 includes cryptographic modules designed for FIPS 140-3, with validation in progress. FIPS mode isn't enabled by default. Use a certified release for workloads that require FIPS. |

Level 2 settings aren't applied by default. Before you apply them, follow
[Apply security baselines safely](#apply-security-baselines-safely).

For Azure service authorizations, such as the Federal Risk and Authorization
Management Program (FedRAMP), see the
[Azure compliance documentation](/azure/compliance/). An Azure service
authorization doesn't cover the guest operating system configuration or your
workload.

## Apply security baselines safely

Preserve the image defaults unless a documented requirement justifies a change.
Don't apply another distribution's benchmark or remediation script to Azure
Linux.

> [!CAUTION]
> Level 2 settings can disrupt startup, administrative access, and application
> behavior, or increase resource use and audit volume. Apply them under the
> supervision of your security and workload owners. For production and
> business-critical systems, require representative testing, an approved
> maintenance window, and a proven recovery path. Your organization must assess
> and accept the operational risk before rollout.

Before you apply a security baseline:

1. Identify the exact benchmark version and profile.
1. Record any exceptions and the reason for each one.
1. Capture the current configuration and test rollback or redeployment from a
   known-good image.
1. Apply the profile in nonproduction. Test startup, access, application
   behavior, monitoring, and recovery under a representative workload.
1. Measure performance and log volume, then obtain security and workload-owner
   approval for a staged production rollout.
1. Reassess the system after image, kernel, package, and workload updates.

## Prepare assessment evidence

Assessment evidence should identify the evaluated system, profile,
configuration, and results:

- The Azure Linux image reference and running kernel version.
- The benchmark or compliance-program version and selected profile.
- The Azure resource configuration that forms part of the control.
- The effective guest configuration, not only the source configuration files.
- Assessment tool output, timestamps, exceptions, and remediation records.
- Package and node-image update state.
- Evidence for monitoring, access reviews, backup, recovery, and change
  control.

Keep evidence outside the assessed system when practical. An administrator who
can change the system can also change evidence stored only on that system.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Inspect kernel hardening](./kernel-hardening.md)
- [Configure SELinux security](./security-enhanced-linux-security.md)
- [Configure security logging and auditing](./logging-and-auditing.md)
- [Manage security updates](./manage-common-vulnerabilities-exposures.md)
- [Azure compliance documentation](/azure/compliance/)
