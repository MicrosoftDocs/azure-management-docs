---
ms.topic: include
ms.date: 07/18/2024
# Customer intent: "As a cloud administrator, I want to create virtual machines from existing resources in my vSphere environment, so that I can automate and streamline my virtual infrastructure management."
---

```azurecli-interactive
az connectedvmware vm create-from-machines --subscription contoso-sub --vcenter-id /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/allhands-demo/providers/microsoft.connectedvmwarevsphere/VCenters/ContosovCentervcenters/contoso-vcenter
```
