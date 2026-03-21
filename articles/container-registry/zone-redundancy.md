---
title: Zone redundancy in Azure Container Registry
description: Learn about zone redundancy in Azure Container Registry and how it protects your registries automatically.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.date: 03/17/2026
ms.custom: references_regions, devx-track-azurecli
ms.service: azure-container-registry

# Customer intent: As a cloud architect, I want to understand how zone redundancy works in Azure Container Registry, so that I can confirm my registries are protected without needing to take additional action.
---

# Zone redundancy in Azure Container Registry

Zone redundancy is enabled by default for all Azure container registries in regions that
[support availability zones](/azure/reliability/regions-list). This applies to every service
tier—Basic, Standard, and Premium—at no additional cost, with no action required from you.

Zone redundancy distributes your registry’s data plane across multiple
[availability zones](/azure/reliability/availability-zones-overview) within a region, so that
image push and pull operations continue to function during a single-zone outage. If your Premium
registry uses [geo-replication](container-registry-geo-replication.md), all replicas in supported
regions are also zone-redundant by default.

For a comprehensive guide to Container Registry reliability—including zone failover behavior,
transient fault handling, and multi-region deployment—see
[Reliability in Azure Container Registry](/azure/reliability/reliability-container-registry).

## 1. What this means for you

Zone redundancy is not an opt-in feature. You don’t need to create a special registry, select a
specific SKU, or flip any settings to be protected. The following table summarizes the behavior
for common scenarios:

| Scenario | Zone-redundant? | Action required |
|----------|-----------------|-----------------|
| New registry in a [supported region](/azure/reliability/regions-list) | Yes | None |
| Existing registry in a supported region | Yes | None |
| Geo-replica (Premium) in a supported region | Yes | None |
| Registry in a region **without** availability zone support | No | Migrate to a supported region (see [below](#4-registries-in-unsupported-regions)) |

> [!NOTE]
> Zone redundancy applies to the registry data plane (image push and pull). Container Registry
> [tasks](/azure/container-registry/container-registry-tasks-overview) don’t currently support
> availability zones.

## 2. Why the `zoneRedundancy` property is confusing

Previously, zone redundancy was an opt-in feature available only on the Premium tier. You had to
explicitly set `zoneRedundancy: Enabled` when creating a registry or geo-replica, and the
property accurately reflected whether zone redundancy was active.

That is no longer the case. Zone redundancy is now enabled by default for all registries in
supported regions, across all service tiers, regardless of what the `zoneRedundancy` property
shows. However, the portal, CLI, and ARM API have not yet been updated to reflect this
change—so the property might still display `Disabled` even though your registry is fully
zone-redundant.

> [!IMPORTANT]
> **Your registry is zone-redundant in any supported region, regardless of what the
> `zoneRedundancy` property shows.** The property value is a legacy artifact that no longer
> controls behavior.

The `zoneRedundancy` property on the ARM registry resource
(`Microsoft.ContainerRegistry/registries`) will be deprecated in accordance with
[Azure’s GA feature deprecation policy](/lifecycle/definitions#deprecation). Until then,
setting it to `Enabled` is harmless but unnecessary, and setting it to `Disabled` has no effect
in supported regions.

## 3. Geo-replication

If your Premium registry is [geo-replicated](container-registry-geo-replication.md), each
replica in a region that supports availability zones is automatically zone-redundant. You don’t
need to enable it during replica creation.

For more information on geo-replication reliability, see
[Resilience to region-wide failures](/azure/reliability/reliability-container-registry#resilience-to-region-wide-failures).

## 4. Registries in unsupported regions

If your registry is in a region that doesn’t support availability zones, it isn’t
zone-redundant. To gain zone redundancy, create a new registry in a
[supported region](/azure/reliability/regions-list) and migrate your images using one of the
following approaches:

- [Import container images](/azure/container-registry/container-registry-import-images) into the
  new registry.
- [Create a transfer pipeline](/azure/container-registry/container-registry-transfer-prerequisites)
  for large-scale or offline migration.

## 5. Infrastructure as Code (Bicep, ARM, Terraform)

The `zoneRedundancy` property still exists in the ARM API and Bicep resource definitions for
backward compatibility, but it no longer controls behavior and is scheduled for deprecation (see
[above](#2-why-the-zoneredundancy-property-is-confusing)). Setting it explicitly is harmless but
no longer required. Here is what you need to know:

| Action | Effect |
|--------|--------|
| Omit `zoneRedundancy` entirely | Registry is zone-redundant in supported regions (default behavior) |
| Set `zoneRedundancy: 'Enabled'` | No change—matches the default. Safe to keep in existing templates |
| Set `zoneRedundancy: 'Disabled'` | Has no effect in supported regions—zone redundancy cannot be disabled |

### Azure CLI reference

The `--zone-redundancy` flag on [az acr create](/cli/azure/acr#az-acr-create) and
[az acr replication create](/cli/azure/acr/replication#az-acr-replication-create) still exists
for backward compatibility. You do not need to pass it—zone redundancy is active by default.

## 6. Frequently asked questions

### Is zone redundancy available on Basic and Standard tiers?

Yes. Zone redundancy applies to all service tiers—Basic, Standard, and Premium—in regions that
support availability zones.

### The portal shows zoneRedundancy as Disabled. Is my registry protected?

Yes. If your registry is in a [region that supports availability zones](/azure/reliability/regions-list),
it is zone-redundant regardless of what the portal or API shows. The display is being updated.

### Can I disable zone redundancy?

No. Zone redundancy cannot be disabled for registries in supported regions.

### Does zone redundancy cost extra?

No. Zone redundancy is included at no additional cost for all service tiers.

### I previously set zoneRedundancy to Enabled in my templates. Do I need to change anything?

No. The explicit `Enabled` setting is compatible with the new default and your templates continue
to work without modification. When the property is deprecated, you can safely remove it—your
registry will remain zone-redundant.

### What about registries I created before zone redundancy became the default?

Existing registries in supported regions have been retroactively upgraded to zone-redundant. No
action is needed.

### What happens during a zone outage?

The service automatically routes traffic to healthy zones. Image push and pull operations continue
with minimal impact. For details on failover behavior and expected data loss windows, see
[Resilience to availability zone failures](/azure/reliability/reliability-container-registry#resilience-to-availability-zone-failures).
