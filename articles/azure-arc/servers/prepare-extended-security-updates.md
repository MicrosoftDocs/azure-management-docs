---
title: Prepare to deliver Extended Security Updates for Windows Server through Azure Arc
description: Learn how to prepare to deliver Extended Security Updates for Windows Server 2012 and Windows Server 2016 through Azure Arc.
ms.date: 07/16/2026
ms.topic: how-to
zone_pivot_groups: extended-security-updates-windows-server
# Customer intent: As a system administrator managing Windows Server 2012 or Windows Server 2016 machines, I want to enroll these servers in Extended Security Updates through Azure Arc, so that I can maintain security and compliance after the end of support and streamline the migration to Azure.
---

# Prepare to deliver Extended Security Updates for Windows Server

By using Azure Arc-enabled servers, you can enroll your existing Windows Server machines in [Extended Security Updates (ESUs)](/windows-server/get-started/extended-security-updates-overview) after they reach end of support. Offering both cost flexibility and an enhanced delivery experience, Azure Arc better positions you to migrate to Azure. Select the version of Windows Server that you're enrolling to see version-specific guidance.

The purpose of this article is to help you understand the benefits and how to prepare to use Arc-enabled servers to enable delivery of ESUs.

::: zone pivot="windows-server-2012"

Windows Server 2012 and Windows Server 2012 R2 reached end of support on October 10, 2023. Billing for Windows Server 2012 ESUs enabled by Azure Arc starts from October 2023, after end of support.

> [!NOTE]
> Azure VMware Solution (AVS) machines and virtual machines on Azure Local are eligible for free ESUs and shouldn't enroll in ESUs enabled through Azure Arc.

::: zone-end

::: zone pivot="windows-server-2016"

Windows Server 2016 reaches end of support on January 12, 2027. Extended Security Updates for Windows Server 2016 provide Critical and Important security updates for up to three years, through 2030. ESUs don't include new features, customer-requested nonsecurity hotfixes, or design change requests.

You can configure Windows Server 2016 ESUs in the Azure portal starting August 3, 2026. Billing for Windows Server 2016 ESUs enabled by Azure Arc begins January 13, 2027.

::: zone-end

## Key benefits

Delivering ESUs to your Windows Server machines provides the following key benefits:

- **Pay-as-you-go:** Flexibility to sign up for a monthly subscription service with the ability to migrate mid-year.

- **Azure billed:** You can draw down from your existing [Microsoft Azure Consumption Commitment (MACC)](/marketplace/azure-consumption-commitment-benefit) and analyze your costs using [Microsoft Cost Management and Billing](/azure/cost-management-billing/cost-management-billing-overview).

- **Built-in inventory:** The coverage and enrollment status of ESUs on eligible Arc-enabled servers are identified in the Azure portal, highlighting gaps and status changes.

- **Keyless delivery:** The enrollment of ESUs on Azure Arc-enabled machines doesn't require the acquisition or activation of keys.

## Access to Azure services

For Azure Arc-enabled servers enrolled in ESUs enabled by Azure Arc, free access is provided to these Azure services for enrolled servers:

* [Azure Update Manager](/azure/update-center/overview) - Unified management and governance of update compliance that includes not only Azure and hybrid machines, but also ESU update compliance for all your Windows Server machines.
    Enrollment in ESUs does not impact Azure Update Manager. After enrollment in ESUs through Azure Arc, the server becomes eligible for ESU patches. These patches can be delivered through Azure Update Manager or any other patching solution. You'll still need to configure updates from Microsoft Updates or Windows Server Update Services.
* [Change Tracking and Inventory](/azure/automation/change-tracking/overview-monitoring-agent?tabs=win-az-vm) - Track changes in virtual machines hosted in Azure, on-premises, and other cloud environments.
* [Azure Policy Guest Configuration](/azure/cloud-adoption-framework/manage/azure-server-management/guest-configuration-policy) - Audit the configuration settings in a virtual machine. Guest configuration supports Azure VMs natively and non-Azure physical and virtual servers through Azure Arc-enabled servers.

Other Azure services through Azure Arc-enabled servers are available as well, with offerings such as:

* [Microsoft Defender for Cloud](/azure/defender-for-cloud/defender-for-cloud-introduction) - As part of the cloud security posture management (CSPM) pillar, it provides server protections through [Microsoft Defender for Servers](/azure/defender-for-cloud/plan-defender-for-servers) to help protect you from various cyber threats and vulnerabilities.
* [Microsoft Sentinel](scenario-onboard-azure-sentinel.md) - Collect security-related events and correlate them with other data sources.
   
## Prepare delivery of ESUs

::: zone pivot="windows-server-2012"

1. Plan and prepare to connect your machines to Azure Arc-enabled servers by installing the [Azure Connected Machine agent](agent-overview.md) to establish a connection to Azure. For Windows Server 2012 ESUs, use agent version 1.34 or higher.

    After establishing this connection, you can enroll your servers to receive Extended Security Updates (ESUs). Both Standard and Datacenter editions are supported. Deploy your machines to Azure Arc so that you have visibility into their ESU coverage and can enroll through the Azure portal or by using Azure Policy.

1. Download both the licensing package and servicing stack update (SSU) for the Azure Arc-enabled server as documented in the applicable Microsoft Knowledge Base article. For Windows Server 2012, see [KB5031043: Procedure to continue receiving security updates after extended support has ended on October 10, 2023](https://support.microsoft.com/topic/kb5031043-procedure-to-continue-receiving-security-updates-after-extended-support-has-ended-on-october-10-2023-c1a20132-e34c-402d-96ca-1e785ed51d45).

::: zone-end

::: zone pivot="windows-server-2016"

1. Plan and prepare to connect your machines to Azure Arc-enabled servers by installing the [Azure Connected Machine agent](agent-overview.md) to establish a connection to Azure. For Windows Server 2016 ESUs, use agent version 1.62 or higher.

    After establishing this connection, you can enroll your servers to receive Extended Security Updates (ESUs). Both Standard and Datacenter editions are supported. We recommend you deploy your machines to Azure Arc so that you have visibility into their ESU coverage and can enroll through the Azure portal or by using Azure Policy.

::: zone-end

Review the version-specific eligibility and licensing requirements before you enroll.

::: zone pivot="windows-server-2012"

Windows Server 2012 Extended Security Updates support Windows Server 2012 and 2012 R2 Standard and Datacenter editions. Windows Server 2012 Storage isn't supported. Billing for this service starts from October 2023 (that is, after Windows Server 2012 end of support).

> [!NOTE]
> To purchase ESUs, you must have Software Assurance through Volume Licensing Programs such as an Enterprise Agreement (EA), Enterprise Agreement Subscription (EAS), Enrollment for Education Solutions (EES), Server and Cloud Enrollment (SCE), or through Microsoft Open Value Programs. Alternatively, if your Windows Server 2012/2012 R2 machines are licensed through SPLA or with a Server Subscription, Software Assurance isn't required to purchase ESUs.

::: zone-end

::: zone pivot="windows-server-2016"

Windows Server 2016 Extended Security Updates support Windows Server 2016 Standard and Datacenter editions. You can configure Windows Server 2016 ESUs in the Azure portal starting August 3, 2026, and billing begins January 13, 2027.

> [!NOTE]
> To purchase ESUs, you must have Software Assurance through Volume Licensing Programs such as an Enterprise Agreement (EA), Enterprise Agreement Subscription (EAS), Enrollment for Education Solutions (EES), Server and Cloud Enrollment (SCE), or through Microsoft Open Value Programs, or an equivalent Server Subscription. Software Assurance is required for on-premises workloads. The Services Provider License Agreement (SPLA) isn't available for Windows Server 2016 ESUs.

::: zone-end

### Deployment options

There are several at-scale onboarding options for Azure Arc-enabled servers:

- Run a [Custom Task Sequence](onboard-configuration-manager-custom-task.md) through Configuration Manager.

- Deploy a [Scheduled Task through Group Policy](onboard-group-policy-powershell.md). 

- Use [VMware vCenter managed VMs](../vmware-vsphere/deliver-extended-security-updates-for-vmware-vms-through-arc.md) through Azure Arc.

- Use [SCVMM managed VMs](../system-center-virtual-machine-manager/deliver-esus-for-system-center-virtual-machine-manager-vms.md) through Azure Arc.

> [!NOTE]
> Delivery of ESUs through Azure Arc to virtual machines running on Virtual Desktop Infrastructure (VDI) is not recommended. VDI systems should use Multiple Activation Keys (MAK) to apply ESUs. See [Access your Multiple Activation Key from the Microsoft 365 Admin Center](/windows-server/get-started/extended-security-updates-deploy) to learn more.
> 

### Networking

Ensure your machines have the necessary network connectivity to Azure Arc. Connectivity options include:

- Public endpoint
- Proxy server
- Private link or Azure Express Route.

Review the [networking prerequisites](network-requirements.md) to prepare non-Azure environments for deployment to Azure Arc.

[!INCLUDE [esu-network-requirements](./includes/esu-network-requirements.md)]

> [!TIP]
> To take advantage of the full range of offerings for Arc-enabled servers, such as extensions and remote connectivity, ensure that you allow the additional URLs that apply to your scenario. For more information, see [Connected machine agent networking requirements](network-requirements.md).

## Required Certificate Authorities

The following [Certificate Authorities](/azure/security/fundamentals/azure-ca-details?tabs=root-and-subordinate-cas-list) are required for Extended Security Updates:

- [Microsoft Azure RSA TLS Issuing CA 03](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2003%20-%20xsign.crt)
- [Microsoft Azure RSA TLS Issuing CA 04](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2004%20-%20xsign.crt)
- [Microsoft Azure RSA TLS Issuing CA 07](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2007%20-%20xsign.crt)
- [Microsoft Azure RSA TLS Issuing CA 08](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2008%20-%20xsign.crt)

If necessary, you can [manually download and install](troubleshoot-extended-security-updates.md#option-2-manually-download-and-install-the-intermediate-ca-certificates) these Certificate Authorities.

## Next steps

* Find out more about [planning for Windows Server and SQL Server end of support](https://www.microsoft.com/en-us/windows-server/extended-security-updates) and [getting Extended Security Updates](/windows-server/get-started/extended-security-updates-deploy).

* Learn about best practices and design patterns through the [Azure Arc landing zone accelerator for hybrid and multicloud](/azure/cloud-adoption-framework/scenarios/hybrid/arc-enabled-servers/eslz-identity-and-access-management).
* Learn more about [Arc-enabled servers](overview.md) and how they work with Azure through the Azure Connected Machine agent.
* Explore options for [onboarding your machines](plan-at-scale-deployment.md) to Azure Arc-enabled servers.
