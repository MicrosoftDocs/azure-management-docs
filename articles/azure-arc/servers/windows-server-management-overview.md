---
title: Windows Server Management enabled by Azure Arc
description: Enrollment in Windows Server Management enabled by Azure Arc.
ms.date: 03/13/2025
ms.topic: concept-article
# Customer intent: As an IT administrator managing hybrid environments, I want to enroll Windows Server Management through Azure Arc, so that I can leverage remote management, configuration, and update capabilities for improved oversight and operational efficiency across my server infrastructure.
---

# Windows Server Management enabled by Azure Arc

Windows Server Management enabled by Azure Arc offers customers with Windows Server licenses that have active Software Assurances or Windows Server licenses that are active subscription licenses the following key benefits:

|Benefit  |Supported Operating Systems  |Description  |
|---------|---------|---------|
|Azure Update Manager  |Windows Server 2012 and above  |Assess the update status and deploy updates to machines (one off, recurring, maintenance windows) with visibility into update compliance and auditing.  |
|Azure Change Tracking and Inventory  |Windows Server 2012 and above  |Discovery of and changes of software, services/daemons, files, and registries of Azure Arc-enabled servers.  |
|Azure Machine Configuration  |Windows Server 2012 and above  |Configuration of machine properties for OS, app, and environment settings, with Azure Policy. Available natively with the Azure Connected Machine agent.  |
|Windows Admin Center in Azure for Arc  |Windows Server 2016 and above  |Securely manage hybrid machines from anywhere without needing a VPN, public IP address, or other inbound connectivity to your machine with RDP, Hyper-V management, event viewer, and much more.  |
|Remote Support  |Windows Server 2016 and above  |Offers customers with professional support the ability to grant JIT access with detailed execution transcripts and revocation rights.  |
|Network HUD  |Windows Server 2025 only  |A host networking diagnostics and operational tool. Runs spot checks, health checks, and cluster wide checks to make sure your host networking set-up is healthy and set up in an optimal and expected way.  |
|Best Practices Assessment  |Windows Server 2016 and above  |Collection and analysis of server data to generate issues and remediation guidance and performance improvements.  |
|Azure Site Recovery Configuration  |Windows Server 2016 and above  |Configuration of Azure Site Recovery to ensure business continuity, provides replication and data resilience for critical workloads.  |
|Azure File Sync | Windows Server 2016 and above  | Arc-enabled Windows servers will receive discounted per-server pricing for Azure File Sync. This benefit applies when running agents V22 or later, providing customers with a cost advantage. |

Together, these capabilities afford robust governance, configuration, and remote management capabilities for Azure Arc-enabled server customers.  

> [!IMPORTANT]
> Customers enrolled in Windows Server pay-as-you-go enabled by Azure Arc are enrolled in these benefits.
> 

## Billing

Upon attestation, customers receive access to the following at no additional cost beyond networking, storage, and log ingestion:

- Azure Update Manager
- Azure Change Tracking and Inventory
- Azure Machine Configuration
- Windows Admin Center in Azure for Arc
- Remote Support
- Network HUD
- Best Practices Assessment
- Azure Site Recovery Configuration

Azure Change Tracking and Inventory and Best Practices Assessment require a Log Analytics workspace that may incur data ingestion costs. While the configuration of Azure Site Recovery is included as a benefit, customers incur costs for the Azure Site Recovery service itself, including for any storage, compute, and networking associated with the service. 

Customers need to explicitly attest for their Azure Arc-enabled servers or enroll in Windows Server pay-as-you-go to be exempt from billing for these services. Eligibility isn't inferred directly from the enablement to Azure Arc. Eligibility is not inferred from licensing status for the Azure Arc-enabled SQL Server instances that may be connected to an Azure Arc-enabled.   

Customer invoices reflect both the complementary benefits included and the enrollment in these benefits through attestation or through Windows Server pay-as-you-go.  

Customers that aren't attesting or enrolled through Windows Server pay-as-you-go can purchase Azure Update Manager, Azure Change Tracking and Inventory, and Azure Machine Configuration for their Azure Arc-enabled servers. The other services aren't available through Azure Arc for non-SA and non-pay-as-you-go customers.

## Requirements

- Agent Version: Connected Machine Agent version 1.47 or higher is required.  

- Operating Systems: The Azure Arc-enabled server’s Operating Systems must be Windows Server 2012 or higher with both Standard/Datacenter editions supported.  

- Networking: Connectivity methods supported include Public Endpoint, Proxy, Azure Arc Gateway, and Private Endpoint. No additional endpoints need to be allowed.  

- Licensing: The Azure Arc-enabled server must be officially licensed through a valid licensing channel. Unlicensed servers aren't eligible for these benefits. Azure Arc-enabled servers enrolled in Windows Server pay-as-you-go are automatically activated for these benefits.  

- Connectivity: The Azure Arc-enabled server must be *Connected* for enrollment. Disconnected and expired servers aren't eligible. Usage of the included benefits requires connectivity.   

- Regions: Activation is available in all regions where Azure Arc-enabled servers has regional availability except for US Gov Virginia, US Gov Arizona, China North 2, China North 3, and China East 2.

- Environments: Supported environments include Hyper-V, VMware, SCVMM, Stack HCI, AVS, and bare-metal where servers are connected to Azure Arc. 

- Modes: Customers can use Monitor mode and extension allowlists or blocklists with their attestation to Azure Arc-enabled servers.  

## Enrollment

You can enroll in Windows Server Management enabled by Azure Arc through the Azure portal or using PowerShell.

### [Portal](#tab/portal)

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/), then navigate to the **Azure Arc** page.

1. In the service menu, under **Licensing**, select **Windows Server Azure benefits and licenses**.

    :::image type="content" source="media/windows-server-management-overview/windows-benefits.png" alt-text="Screenshot of Azure portal showing Windows Server benefits and licenses with benefits pop up.":::

1. Select the Azure Arc-enabled servers that are eligible for enrollment in benefits and select **Activate benefits**.

1. Review the terms to make the attestation and select **Activate** for the Azure benefits for the selected Azure Arc-enabled servers. 

Upon activation of Azure benefits, the Azure Arc-enabled servers show as *Activated* within 10 minutes. 

### [PowerShell](#tab/powershell)

The following PowerShell script can be adapted for attestation at scale of Azure Arc-enabled servers to enroll in Windows Server Management enabled by Azure Arc:

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






