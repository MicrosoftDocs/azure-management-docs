---
ms.topic: include
ms.date: 07/18/2024
# Customer intent: "As a cloud administrator, I want to create a virtual machine from existing machines in my VMware environment, so that I can efficiently manage resources and streamline deployment processes in my cloud infrastructure."
---

```azurecli-interactive
az connectedvmware vm create-from-machines --resource-group contoso-rg --vcenter-id /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/allhands-demo/providers/microsoft.connectedvmwarevsphere/VCenters/ContosovCentervcenters/contoso-vcenter
```
