---
title: Azure Linux Release Cadence and Lifecycle
description: Learn about the release cadence and lifecycle of Azure Linux, including how kernels, packages, and runtimes are introduced, supported, and retired.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/26/2026
---

# Azure Linux release cadence and lifecycle

Azure Linux uses a rolling lifecycle model that balances stability with progress. This article explains how Azure Linux kernels, packages, and runtimes are introduced, supported, and retired so you can plan upgrades with confidence.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Azure Linux lifecycle principles

The following table summarizes the four guiding principles that shape how Azure Linux releases, supports, and retires kernels and major packages:

| Principle | What it means for you |
| --------- | --------------------- |
| **Predictability** | You see published start dates and end-of-support dates for every kernel and major package, so you can plan upgrades on your schedule instead of reacting to surprises. |
| **Security first** | You receive upstream security fixes on defined SLAs. Critical and High CVEs are fast-tracked, so you don't wait for the next monthly window to be protected. |
| **Controlled change** | When a new kernel or major component releases, you keep support for the previous version through an overlap window. You have time to validate and migrate before the older version is retired. |
| **Cloud-native lifecycle** | You get an operating system (OS) designed for environments that update continuously, not one pinned to a single kernel for long periods of time. |

## Kernel lifecycle

Azure Linux tracks upstream Linux Long-Term Support (LTS) kernels and introduces Hardware Enablement (HWE) kernels annually. The following table summarizes the two kernel types:

| Kernel type | Support duration | Purpose | Servicing |
| ----------- | ---------------- | ------- | --------- |
| **LTS** | Four years from GA | Enterprise stability, production workloads | Monthly CVE backports, fast-track Critical/High patches |
| **HWE** | ~14 months | New hardware platforms, GPU/AI accelerator enablement | Monthly CVE servicing, GPU validation |

Azure Linux 4.0 ships with **kernel 6.18 LTS**.

### Kernel lifecycle stages

The following table outlines the stages each kernel goes through from upstream release to end of support in Azure Linux:

| Stage | Description |
| ----- | ----------- |
| **Upstream LTS** | Kernel enters upstream LTS support with community security and stability fixes (~two years). |
| **Preview** | Available in Azure Linux for evaluation and compatibility testing. Not recommended for production. |
| **General Availability (GA)** | Production-ready. Receives regular security updates. Both new and previous kernels are supported simultaneously during the overlap period. |
| **End of Support** | No further updates. You must migrate to a newer kernel before this date. Timelines are communicated well in advance. |

:::image type="content" source="./media/kernel-lifecycle.png" alt-text="Screenshot of a diagram showing the Azure Linux Kernel Lifecycle LTS and HWE tracks." lightbox="./media/kernel-lifecycle.png":::

### When to use each kernel type

Choose between the LTS and HWE kernels based on how much your workload values stability versus access to the latest hardware support.

- **LTS**: Choose the LTS kernel for production workloads that need long-term stability and a predictable kernel ABI. The four-year support window means you can keep custom kernel modules, eBPF programs, and tightly coupled drivers in service for the lifetime of a major Azure Linux release without re-validating against a new kernel every year. This is the right default for most general-purpose virtual machines (VMs), AKS node pools, and platform services.
- **HWE**: Choose the HWE kernel when you need newer drivers, accelerator support, or platform features sooner than the LTS cadence allows. Typical scenarios include AI and ML training clusters, GPU-dependent inference workloads, HPC, and services onboarding new Azure hardware SKUs. HWE kernels carry a shorter (~14-month) support window, so plan for an annual revalidation cycle as you move to each new HWE release.

## Package Lifecycle

Azure Linux defines two tiers of packages with different update models: [Tier 1: Core OS (locked)](#tier-1-core-os-locked) and [Tier 2: Language runtimes and packages (refreshed)](#tier-2-language-runtimes-and-packages-refreshed).

### Tier 1: Core OS (locked)

Core OS components are version-locked for the lifetime of a major release. They receive only CVE backports and have no version bumps.

The following table summarizes the lifecycle strategy for each core OS component and any special notes about how updates are applied:

| Component | Lifecycle strategy | Notes |
| --------- | ------------------ | ----- |
| Kernel | LTS / HWE model | See [kernel lifecycle](#kernel-lifecycle) section |
| OpenSSL | Upstream LTS aligned | Security-focused maintenance |
| glibc | Fixed for OS lifecycle | Preserves ABI compatibility |
| systemd | Fixed major version | Predictable system behavior |
| coreutils | Upstream aligned | Security fixes only |
| Kata Containers | Fixed per release | No mid-release upgrades |

### Tier 2: Language runtimes and packages (refreshed)

Tier 2 packages are updated through two **half-yearly refresh windows** — H1 and H2. Between windows, only critical/high CVE patches are backported.

The following table summarizes the timing and activities that occur during each refresh window:

| Window | Timing | What happens |
| ------ | ------ | ------------ |
| **H1** | January – June | New upstream versions integrated, tested as a cohort, and rolled out |
| **H2** | July – December | Second round of version updates with the same validation process |

:::image type="content" source="./media/tier-2-package-refresh.png" alt-text="Screenshot of a diagram showing the Azure Linux Tier 2 Package Refresh Cadence." lightbox="./media/tier-2-package-refresh.png":::

### How H1/H2 refreshes work

H1/H2 refreshes occur twice per year and represent coordinated updates of Tier 2 language runtimes and packages. Each refresh window follows a structured process to ensure that new upstream versions are integrated, tested, and rolled out safely across the Azure Linux fleet:

1. **Version selection**: Azure Linux team evaluates the latest stable upstream releases at the start of each window.
1. **Integration & testing**: Selected versions are built and integration-tested together as a cohort against Azure workloads.
1. **Staged rollout**: The validated refresh is rolled out incrementally across the fleet.
1. **Between windows**: Only critical/high CVE patches are backported. No version bumps.

### Tier 2 runtime upgrade models

Each Tier 2 runtime follows one of three upgrade models that determines how its version changes during H1/H2 refreshes:

| Model | Behavior | Use case | Example runtimes |
| ----- | -------- | -------- | ---------------- |
| **Opt-in** | Default stays the same. Newer versions available as optional parallel installs. | Stability-critical workloads where version changes could break behavior. | Python, Java (OpenJDK), .NET |
| **Sliding** | New version introduced alongside current. After one cycle, the new version becomes default. Old version available via pinning. | Workloads that benefit from staying current with a controlled transition. | Go, Node.js, Ruby |
| **Rolling** | Default moves forward automatically at each refresh. Pin explicitly to stay on an older version. | Fast-moving ecosystems where staying current is the norm. | Rust, Nginx, PHP |

#### Tier 2 runtime upgrade model examples

The following table provides examples of how each Tier 2 runtime upgrade model behaves in practice during H1/H2 refresh cycles:

| Model | Runtime | Details |
| ----- | ------- | ------- |
| **Opt-in** | Python | • **H1**: Python 3.14 remains default. Python 3.15 added as optional install. <br> • **H2**: Python 3.14 still default. Python 3.15 receives updates. <br> • Install `python3.15` explicitly to opt in. `python3` still resolves to 3.14. |
| **Sliding** | Go | • **H1**: Go 1.23 is default. Go 1.24 introduced alongside. <br> • **H2**: Go 1.24 becomes default. Go 1.23 available via `golang-1.23` but enters deprecation. <br> • **Next H1**: Go 1.23 removed from active support. |
| **Rolling** | Rust | • **H1**: Default moves from Rust 1.82 to 1.84 (even releases only). <br> • **H2**: Default moves from Rust 1.84 to 1.86. <br> • Pin `rust-1.84` explicitly to stay on a specific version. |

### Tier 2 supported runtimes

The following table lists all Tier 2 runtimes supported in Azure Linux, along with their upgrade model, refresh cadence, and the number of concurrent versions that are actively maintained and available for use:

| Runtime | Upgrade model | Cadence | Concurrent versions |
| ------- | ------------- | ------- | ------------------- |
| Python | Opt-in | ~Six months post upstream | One default + optional |
| Node.js | Sliding | LTS cadence | n and n-2 |
| .NET | Opt-in | .NET release aligned | One default + optional |
| Java (OpenJDK) | Opt-in | Upstream LTS | One default + optional |
| Go | Sliding | ~Six months | Latest + latest-1 |
| Rust | Rolling | Quarterly | Latest + latest-2 (even) |
| Ruby | Sliding | Annual | n and n-1 |
| Nginx | Rolling | Even stable releases | Latest + latest-2 |
| PHP | Opt-in | Annual | Latest + latest-1 |
| Kata Containers | Locked (Tier 1) | Per release | One fixed version |

Only runtime versions supported by their upstream communities receive full support in Azure Linux. When a version approaches upstream end of support, Azure Linux publishes advance notice and migration guidance.

## Monthly CVE fixes

Independent of kernel transitions and H1/H2 runtime refreshes, Azure Linux ships a monthly security update that backports CVE fixes to every supported package.

For the full CVE pipeline, including how vulnerabilities are triaged, patched, validated, and published, plus per-deployment guidance for Azure Linux and Azure Container Linux, see [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md).

## Related content

- [Azure Linux architecture overview](./architecture.md)
- [Azure Linux release notes](./azure-linux-overview.md#azure-linux-release-notes)
