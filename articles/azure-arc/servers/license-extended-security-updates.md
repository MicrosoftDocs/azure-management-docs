---
title: License provisioning guidelines for Extended Security Updates for Windows Server
description: Learn about license provisioning guidelines for Extended Security Updates for Windows Server 2012 and Windows Server 2016 through Azure Arc.
ms.date: 07/16/2026
ms.topic: concept-article
zone_pivot_groups: extended-security-updates-windows-server
# Customer intent: "As an IT administrator managing Windows Server environments, I want to provision Extended Security Updates licenses through Azure Arc, so that I can ensure my servers remain secure and compliant despite being out of support."
---

# License provisioning guidelines for Extended Security Updates for Windows Server

Flexibility is critical when enrolling end of support infrastructure in Extended Security Updates (ESUs) through Azure Arc to receive critical patches. To give ease of options across virtualization and disaster recovery scenarios, you must first provision Windows Server Arc ESU licenses and then link those licenses to your Azure Arc-enabled servers. You can link and provision licenses through the Azure portal.

::: zone pivot="windows-server-2012"

The guidance in this article applies to Windows Server 2012/2012 R2 ESU licenses.

::: zone-end

::: zone pivot="windows-server-2016"

The guidance in this article applies to Windows Server 2016 ESU licenses.

::: zone-end

When provisioning ESU licenses, specify:

* Either virtual core or physical core license
* Standard or Datacenter license

You also need to attest to the number of associated cores (broken down by the number of 2-core and 16-core packs).

To assist with the license provisioning process, this article provides general guidance and sample customer scenarios for planning your deployment of Windows Server ESUs through Azure Arc. Select your version of Windows Server at the top of the page to see version-specific examples.

## General guidance: Standard vs. Datacenter, Physical vs. Virtual Cores

### Physical core licensing

If you choose to license based on physical cores, the licensing requires a minimum of 16 physical cores per machine. Most customers choose to license based on physical cores and select Standard or Datacenter edition to match their original Windows Server licensing. While Standard licensing can be applied to up to two virtual machines (VMs), Datacenter licensing has no limit to the number of VMs it can be applied to. Depending on the number of VMs covered, it may make sense to choose the Datacenter license instead of the Standard license.

### Virtual core licensing

If you choose to license based on virtual cores, the licensing requires a minimum of eight virtual cores per Virtual Machine. There are two main scenarios where this model is advisable:

- If the VM is running on a third-party host or cloud service provider like AWS, GCP, or OCI.
- The Windows Server operating system was licensed on a virtualization basis.

Another scenario (scenario 1, below) is a candidate for VM/Virtual core licensing when the ESU-eligible VMs are running on a newer Windows Server host.

> [!IMPORTANT]
> Virtual core licensing can't be used on physical servers. When creating a license with virtual cores, always select the standard edition instead of datacenter, even if the operating system is datacenter edition.

### License limits

Each ESU license can cover up to and including 10,000 cores. If you need ESUs for more than 10,000 cores, split the total number of cores across multiple licenses. Additionally, you can create only 800 licenses in a single resource group. Use more resource groups if you need to create more than 800 license resources.

### Software Assurance/Services Provider License Agreement conformance

::: zone pivot="windows-server-2012"

In all cases, you must attest to conformance with Software Assurance (SA) or an equivalent Server Subscription. You need Software Assurance or an equivalent Server Subscription to purchase Extended Security Updates on-premises and in hosted environments. You can purchase Extended Security Updates through Enterprise Agreement (EA), Enterprise Subscription Agreement (EAS), a Server & Cloud Enrollment (SCE), and Enrollment for Education Solutions (EES). On Azure, you don't need Software Assurance to get free Extended Security Updates, but you need Software Assurance or Server Subscription to take advantage of the Azure Hybrid Benefit.

For Windows Server 2012 and 2012 R2, you can also attest to conformance through a Services Provider License Agreement (SPLA). If you license your machines through SPLA or with a Server Subscription, you don't need Software Assurance to purchase ESUs.

::: zone-end

::: zone pivot="windows-server-2016"

In all cases, you must attest to conformance with Software Assurance (SA) or an equivalent Server Subscription. You need Software Assurance or an equivalent Server Subscription to purchase Extended Security Updates on-premises and in hosted environments. You can purchase Extended Security Updates through Enterprise Agreement (EA), Enterprise Subscription Agreement (EAS), a Server & Cloud Enrollment (SCE), and Enrollment for Education Solutions (EES). 

For Windows Server 2016, you need Software Assurance (or an equivalent Server Subscription) for on-premises workloads. The Services Provider License Agreement (SPLA) isn't available for Windows Server 2016 ESUs.

::: zone-end

### Visual Studio subscription benefit for dev/test scenarios

::: zone pivot="windows-server-2012"

Visual Studio subscriptions [allow developers to get product keys](/visualstudio/subscriptions/product-keys) for Windows Server at no extra cost to help them develop and test their software. If a Windows Server 2012 server's operating system is licensed through a product key obtained from a Visual Studio subscription, you can also get extended security updates for these servers at no extra cost. To configure ESU licenses for these servers using Azure Arc, you must have at least one server with paid ESU usage. You can't create an ESU license where all associated servers are entitled to the Visual Studio subscription benefit. See [additional scenarios](deliver-extended-security-updates.md#additional-scenarios) in the deployment article for more information on how to provision an ESU license correctly for this scenario.

Development, test, and other non-production servers that have a paid operating system license (from your organization's volume licensing key, for example) **must** use a paid ESU license. The only dev/test servers entitled to ESU licenses at no extra cost are those whose operating system licenses came from a Visual Studio subscription.

::: zone-end

::: zone pivot="windows-server-2016"

The Visual Studio subscription benefit for dev/test scenarios isn't available for Windows Server 2016 ESUs.

::: zone-end

## Cost savings with migration and modernization of workloads

::: zone pivot="windows-server-2012"

As you migrate and modernize your Windows Server 2012 and Windows 2012 R2 infrastructure through the end of 2023, you can utilize the flexibility of monthly billing with Windows Server 2012 ESUs enabled by Azure Arc for cost savings benefits.

As servers no longer require ESUs because they've been migrated to Azure, Azure VMware Solution (AVS), or Azure Local **where they’re eligible for free ESUs**, or updated to Windows Server 2016 or higher, you can modify the number of cores associated with a license or delete/deactivate licenses. You can also link the license to a new scope of additional servers. See [Programmatically deploy and manage Azure Arc Extended Security Updates licenses](api-extended-security-updates.md) to learn more. For information about no-cost ESUs through Azure Local, see [Free Extended Security Updates through Azure Local](/azure/azure-local/manage/azure-benefits-esu?tabs=windows-server-2012).

::: zone-end

::: zone pivot="windows-server-2016"

As you migrate and modernize your Windows Server 2016 infrastructure, use the flexibility of monthly billing with Windows Server 2016 ESUs enabled by Azure Arc for cost savings benefits.

As servers no longer require ESUs because they're migrated to Azure or updated to Windows Server 2019 or higher, you can modify the number of cores associated with a license or delete or deactivate licenses. You can also link the license to a new scope of additional servers. To learn more, see [Programmatically deploy and manage Azure Arc Extended Security Updates licenses](api-extended-security-updates.md).

::: zone-end

> [!NOTE]
> This process is not automatic; billing is tied to the activated licenses and you are responsible for modifying your provisioned licensing to take advantage of cost savings.

## Scenario based examples: Compliant and Cost Effective Licensing

::: zone pivot="windows-server-2012"

### Scenario 1: Eight modern 32-core hosts (not Windows Server 2012). While each of these hosts are running four 8-core VMs, only one VM on each host is running Windows Server 2012 R2

In this scenario, you can use virtual core-based licensing to avoid covering the entire host by provisioning eight Windows Server 2012 Standard licenses for eight virtual cores each and link each of those licenses to the VMs running Windows Server 2012 R2. Alternatively, you could consider consolidating your Windows Server 2012 R2 VMs into two of the hosts to take advantage of physical core-based licensing options.

### Scenario 2: A branch office with four VMs, each 8-cores, on a 32-core Windows Server 2012 Standard host

In this case, you should provision two WS2012 Standard licenses for 16 physical cores each and apply to the four Arc-enabled servers. Alternatively, you could provision four WS2012 Standard licenses for eight virtual cores each and apply individually to the four Arc-enabled servers.

### Scenario 3: Eight physical servers in retail stores, each server is standard with eight cores each and there's no virtualization

In this scenario, you should apply eight WS2012 Standard licenses for 16 physical cores each and link each license to a physical server. Note that the 16 physical core minimum applies to the provisioned licenses.

### Scenario 4: Multicloud environment with 12 AWS VMs, each of which have 12 cores and are running Windows Server 2012 R2 Standard

In this scenario, you should apply 12 Windows Server 2012 Standard licenses with 12 virtual cores each, and link individually to each AWS VM.

### Scenario 5: You have already purchased the traditional Windows Server 2012 ESUs through Volume Licensing

In this scenario, the Azure Arc-enabled servers that have been enrolled in Extended Security Updates through an activated MAK Key are as enrolled in ESUs in the Azure portal. You have the flexibility to switch from this key-based traditional ESU model to WS2012 ESUs enabled by Azure Arc between Year one and Year two.

### Scenario 6: Migrating or retiring your Azure Arc-enabled servers enrolled in Windows Server 2012 ESUs

In this scenario, you can deactivate or decommission the ESU Licenses associated with these servers. If only part of the server estate covered by a license no longer requires ESUs, you can modify the ESU license details to reduce the number of associated cores.  

### Scenario 7: 128-core Windows Server 2012 Datacenter server running between 10 and 15 Windows Server 2012 R2 VMs that get provisioned and deprovisioned regularly

In this scenario, you should provision a Windows Server 2012 Datacenter license associated with 128 physical cores and link this license to the Arc-enabled Windows Server 2012 R2 VMs running on it. The deletion of the underlying VM also deletes the corresponding Arc-enabled server resource, enabling you to link another Arc-enabled server.

### Scenario 8: An insurance customer is running a 16 node VMware cluster with 1024 physical cores on-premises. 44 of the VMs on the cluster are running Windows Server 2012 R2. Those 44 VMs consume 506 virtual cores, which was calculated by summing up the maximum of 8 or the actual number of cores assigned to each VM.

In this scenario, you could either license the entire cluster with 1024 Windows Server 2012 Datacenter ESU physical cores or license each VM individually with a total of 506 standard edition virtual cores. In this case, it's cheaper to purchase an Arc ESU Windows Server 2012 Standard edition license associated with 506 virtual cores. You'll need to onboard each of the 44 VMs to Azure Arc and then link the license to the Arc machines.

> [!IMPORTANT]
> If you migrate the VMs to Azure VMware Solution (AVS), these servers become eligible for free WS2012 ESUs and should not enroll in ESUs enabled through Azure Arc.
> 

::: zone-end

::: zone pivot="windows-server-2016"

### Scenario 1: Eight modern 32-core hosts (not Windows Server 2016). While each of these hosts runs four 8-core VMs, only one VM on each host runs Windows Server 2016

In this scenario, use virtual core-based licensing to avoid covering the entire host by provisioning eight Windows Server 2016 Standard licenses for eight virtual cores each and link each of those licenses to the VMs running Windows Server 2016. Alternatively, consider consolidating your Windows Server 2016 VMs into two of the hosts to take advantage of physical core-based licensing options.

### Scenario 2: A branch office with four VMs, each 8-cores, on a 32-core Windows Server 2016 Standard host

In this case, provision two Windows Server 2016 Standard licenses for 16 physical cores each and apply to the four Arc-enabled servers. Alternatively, provision four Windows Server 2016 Standard licenses for eight virtual cores each and apply individually to the four Arc-enabled servers.

### Scenario 3: Eight physical servers in retail stores, each server is standard with eight cores each and there's no virtualization

In this scenario, apply eight Windows Server 2016 Standard licenses for 16 physical cores each and link each license to a physical server. Note that the 16 physical core minimum applies to the provisioned licenses.

### Scenario 4: Multicloud environment with 12 AWS VMs, each of which have 12 cores and are running Windows Server 2016 Standard

In this scenario, apply 12 Windows Server 2016 Standard licenses with 12 virtual cores each, and link individually to each AWS VM.

### Scenario 5: Migrating or retiring your Azure Arc-enabled servers enrolled in Windows Server 2016 ESUs

In this scenario, you can deactivate or decommission the ESU licenses associated with these servers. If only part of the server estate covered by a license no longer requires ESUs, you can modify the ESU license details to reduce the number of associated cores.  

### Scenario 6: 128-core Windows Server 2016 Datacenter server running between 10 and 15 Windows Server 2016 VMs that get provisioned and deprovisioned regularly

In this scenario, you should provision a Windows Server 2016 Datacenter license associated with 128 physical cores and link this license to the Arc-enabled Windows Server 2016 VMs running on it. The deletion of the underlying VM also deletes the corresponding Arc-enabled server resource, enabling you to link another Arc-enabled server.

### Scenario 7: An insurance customer is running a 16 node VMware cluster with 1,024 physical cores on-premises. 44 of the VMs on the cluster are running Windows Server 2016. Those 44 VMs consume 506 virtual cores, which was calculated by summing up the maximum of 8 or the actual number of cores assigned to each VM.

In this scenario, you could either license the entire cluster with 1,024 Windows Server 2016 Datacenter ESU physical cores or license each VM individually with a total of 506 standard edition virtual cores. In this case, it's cheaper to purchase an Arc ESU Windows Server 2016 Standard edition license associated with 506 virtual cores. You need to onboard each of the 44 VMs to Azure Arc and then link the license to the Arc machines.

::: zone-end

## License operations

::: zone pivot="windows-server-2012"

The management scenarios for provisioned Windows Server Arc ESU license resources have several limitations:

- License cores are a mutable property, and customers are able to increment or decrement cores. This is subject to the mandatory minimums of both: (i) 16 cores for Physical core based licenses and (ii) 8 cores for Virtual core based licenses. 

- License edition and type is not a mutable property. Standard licenses can't be changed to Datacenter licenses, and vice versa. Similarly, Physical core licenses can't be changed to Virtual core licenses, and vice versa. Note that there are three valid licensing combinations: Standard Virtual Core, Standard Physical Core, and Datacenter Physical Core. Datacenter Virtual cores aren't a viable licensing combination. Erroneously provisioned Datacenter Virtual core licenses have been translated to Datacenter Physical core licenses with core counts compliant with licensing guidelines.   

- Licenses can be moved between resource groups and subscriptions. License are modeled in Azure Resource Manager and can be queried using Azure Resource Graph. 

- Licenses can be linked to servers in another subscription within the same tenant, but licenses can't be linked to servers within subscriptions of other tenants.

- Tagging a license under evaluation scenarios such as Dev Test or Disaster Recovery doesn't impact billing. Billing is strictly tied to the number of cores associated with the license regardless of tags. The cores used for evaluation or free scenarios shouldn't be provisioned for the Azure Arc ESU license. 

::: zone-end

::: zone pivot="windows-server-2016"

The management scenarios for provisioned Windows Server Arc ESU license resources have several limitations:

- License cores are a mutable property, and you can increment or decrement cores. This change is subject to the mandatory minimums of both: (i) 16 cores for Physical core based licenses and (ii) 8 cores for Virtual core based licenses. 

- License edition and type aren't mutable properties. You can't change Standard licenses to Datacenter licenses, and vice versa. Similarly, you can't change Physical core licenses to Virtual core licenses, and vice versa. Note that there are three valid licensing combinations: Standard Virtual Core, Standard Physical Core, and Datacenter Physical Core. Datacenter Virtual cores aren't a viable licensing combination. Erroneously provisioned Datacenter Virtual core licenses are translated to Datacenter Physical core licenses with core counts compliant with licensing guidelines.   

- Licenses can be moved between resource groups and subscriptions. License are modeled in Azure Resource Manager and can be queried using Azure Resource Graph. 

- You can link licenses to servers in another subscription within the same tenant, but you can't link licenses to servers within subscriptions of other tenants.


::: zone-end

## Next steps

* Find out more about [planning for Windows Server and SQL Server end of support](https://www.microsoft.com/en-us/windows-server/extended-security-updates) and [getting Extended Security Updates](/windows-server/get-started/extended-security-updates-deploy).

* Learn about best practices and design patterns through the [Azure Arc landing zone accelerator for hybrid and multicloud](/azure/cloud-adoption-framework/scenarios/hybrid/arc-enabled-servers/eslz-identity-and-access-management).
* Learn more about [Arc-enabled servers](overview.md) and how they work with Azure through the Azure Connected Machine agent.
* Explore options for [onboarding your machines](plan-at-scale-deployment.md) to Azure Arc-enabled servers.
