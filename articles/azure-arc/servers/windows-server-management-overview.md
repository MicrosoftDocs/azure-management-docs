---
title: Windows Server Management enabled by Azure Arc
description: Enrollment in Windows Server Management enabled by Azure Arc.
ms.date: 04/01/2026
ms.topic: concept-article
# Customer intent: As an IT administrator managing hybrid environments, I want to enroll Windows Server Management through Azure Arc, so that I can leverage remote management, configuration, and update capabilities for improved oversight and operational efficiency across my server infrastructure.
---

# Windows Server Management enabled by Azure Arc

Windows Server Management enabled by Azure Arc offers customers with Windows Server licenses that have active Software Assurances or Windows Server licenses that are active subscription licenses the following key benefits:

|Benefit  |Supported Operating Systems  |Description  |
|---------|---------|---------|
|Azure Update Manager  |Windows Server 2012 and later  |Assess the update status and deploy updates to machines (one-time, recurring, maintenance windows) with visibility into update compliance and auditing.  |
|Azure Change Tracking and Inventory  |Windows Server 2012 and later  |Discovery of and changes to software, services/daemons, files, and registries of Azure Arc-enabled servers.  |
|Azure Machine Configuration  |Windows Server 2012 and later  |Configuration of machine properties for OS, app, and environment settings, with Azure Policy. Available natively with the Azure Connected Machine agent.  |
|Windows Admin Center in Azure for Arc  |Windows Server 2016 and later  |Securely manage hybrid machines from anywhere without needing a VPN, public IP address, or other inbound connectivity to your machine with RDP, Hyper-V management, event viewer, and much more.  |
|Remote Support  |Windows Server 2016 and later  |Offers customers with professional support the ability to grant JIT access with detailed execution transcripts and revocation rights.  |
|Network HUD  |Windows Server 2025 only  |A host networking diagnostics and operational tool. Runs spot checks, health checks, and cluster wide checks to make sure your host networking set-up is healthy and set up in an optimal and expected way.  |
|Best Practices Assessment  |Windows Server 2016 and later  |Collection and analysis of server data to generate issues and remediation guidance and performance improvements.  |
|Azure Site Recovery Configuration  |Windows Server 2016 and later  |Configuration of Azure Site Recovery to ensure business continuity, provides replication and data resilience for critical workloads.  |
|Azure File Sync | Windows Server 2016 and later  | Arc-enabled Windows servers receive discounted per-server pricing for Azure File Sync. This benefit applies when running agents V22 or later, providing customers with a cost advantage. |

Together, these capabilities provide robust governance, configuration, and remote management capabilities for Azure Arc-enabled server customers.  

> [!IMPORTANT]
> Customers enrolled in Windows Server pay-as-you-go enabled by Azure Arc are enrolled in these benefits.

## Billing

Upon attestation, customers get access to the following services at no extra cost beyond networking, storage, and log ingestion costs:

- Azure Update Manager
- Azure Change Tracking and Inventory
- Azure Machine Configuration
- Windows Admin Center in Azure for Arc
- Remote Support
- Network HUD
- Best Practices Assessment
- Azure Site Recovery Configuration
- Azure File Sync

Azure Change Tracking and Inventory and Best Practices Assessment require a Log Analytics workspace that might incur data ingestion costs. While the configuration of Azure Site Recovery is included as a benefit, customers incur costs for the Azure Site Recovery service itself, including any storage, compute, and networking costs associated with the service.

To be exempt from billing for these services, customers need to explicitly attest for their Azure Arc-enabled servers or enroll in Windows Server pay-as-you-go. Eligibility isn't inferred directly from the enablement to Azure Arc or from the licensing status for Azure Arc-enabled SQL Server instances connected to an Arc-enabled server.

Customer invoices reflect both the complementary benefits included and the enrollment in these benefits through attestation or through Windows Server pay-as-you-go.  

Customers that aren't attesting or enrolled through Windows Server pay-as-you-go can purchase Azure Update Manager, Azure Change Tracking and Inventory, and Azure Machine Configuration for their Azure Arc-enabled servers. The other services aren't available through Azure Arc for non-SA and non-pay-as-you-go customers.

## Requirements

- Agent version: Connected Machine Agent version 1.47 or higher.  

- Operating systems: The Azure Arc-enabled server's operating system must be Windows Server 2012 or higher, with support for both Standard and Datacenter editions.

- Networking: Supported connectivity methods include public endpoint, proxy, Azure Arc Gateway, and private endpoint. You don't need to allow any additional endpoints.  

- Licensing: The Azure Arc-enabled server must be officially licensed through a valid licensing channel. Unlicensed servers aren't eligible for these benefits. Azure Arc-enabled servers enrolled in Windows Server pay-as-you-go are automatically activated for these benefits.  

- Connectivity: The Azure Arc-enabled server must be *Connected* for enrollment. Disconnected and expired servers aren't eligible. Usage of the included benefits requires connectivity.

- Regions: Activation is available in all regions where Azure Arc-enabled servers has regional availability except for US Gov Virginia, US Gov Arizona, China North 2, China North 3, and China East 2.

- Environments: Supported environments include Hyper-V, VMware, SCVMM, Stack HCI, AVS, and bare-metal where servers are connected to Azure Arc.

- Modes: Customers can use Monitor mode and extension allow lists or block lists with their attestation to Azure Arc-enabled servers.  

## Enrollment

You can enroll in Windows Server Management enabled by Azure Arc through the Azure portal or by using PowerShell.

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/), and then go to the **Azure Arc** page.

1. In the service menu, under **Licenses**, select **Azure Benefits - Windows Server**.

1. Select the Azure Arc-enabled servers that are eligible for enrollment in benefits and select **Activate benefits**.

    :::image type="content" source="media/windows-server-management-overview/windows-server-benefits.png" alt-text="Screenshot of Azure portal showing Windows Server benefits and licenses with benefits pop up." lightbox="media/windows-server-management-overview/windows-server-benefits.png":::

1. Review the terms and check the checkbox to make the attestation.

1. Select **Activate** to activate Azure benefits for the selected Azure Arc-enabled servers.

Within ten minutes, the **Benefits** status for the eligible Azure Arc-enabled servers that you activated will change to **Activated**.

### [PowerShell](#tab/powershell)

Use the following PowerShell script for attestation at scale of Azure Arc-enabled servers to enroll in Windows Server Management enabled by Azure Arc. Be sure to replace the variables in the script with your values before running it.

```powershell
$subscriptionId    = '' #Your subscription id 
$resourceGroupName = '' # your Resource Group 
$machineName       = '' # Arc resource name 
$location = "" # The region where the test machine is arc enabled. 

$account       = Connect-AzAccount 
$context       = Set-azContext -Subscription $subscriptionId 
$profile       = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile 
$profileClient = [Microsoft.Azure.Commands.ResourceManager.Common.rmProfileClient]::new( $profile ) 
$token         = $profileClient.AcquireAccessToken($context.Subscription.TenantId) 
$header = @{ 
   'Content-Type'='application/json' 
   'Authorization'='Bearer ' + $token.AccessToken 
} 

$uri = [System.Uri]::new( "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.HybridCompute/machines/$machineName/licenseProfiles/default?api-version=2023-10-03-preview" ) 
$contentType = "application/json"  
$data = @{         
    location = $location; 
    properties = @{ 
        softwareAssurance = @{ 
            softwareAssuranceCustomer= $true; 
        }; 
    }; 
}; 
$json = $data | ConvertTo-Json; 
$response = Invoke-RestMethod -Method PUT -Uri $uri.AbsoluteUri -ContentType $contentType -Headers $header -Body $json; 
$response.properties
```

---

