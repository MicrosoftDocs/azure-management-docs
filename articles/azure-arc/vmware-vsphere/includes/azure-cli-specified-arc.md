---
ms.topic: include
ms.date: 07/18/2024
# Customer intent: "As a cloud administrator, I want to create virtual machines from existing virtual machines, so that I can efficiently scale my resources in a hybrid cloud environment."
---

```azurecli-interactive
az connectedvmware vm create-from-machines --resource-group contoso-rg --name contoso-vm --vcenter-id /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/allhands-demo/providers/microsoft.connectedvmwarevsphere/VCenters/ContosovCentervcenters/contoso-vcenter
```
