---
title: How to troubleshoot delivery of Extended Security Updates for Windows Server 2012 through Azure Arc
description: Learn how to troubleshoot delivery of Extended Security Updates (ESU) for Windows Server 2012 through Azure Arc.
ms.topic: troubleshooting
author: JnHs
ms.author: jenhayes
ms.date: 01/15/2026
# Customer intent: As an IT administrator managing Windows Server workloads, I want to troubleshoot Extended Security Updates delivery through Azure Arc, so that I can ensure compliance and maintain security for my servers on extended support.
---

# Troubleshoot delivery of Extended Security Updates for Windows Server 2012

This article explains how to identify and resolve problems when enabling [Extended Security Updates (ESU) for Windows Server 2012 and Windows Server 2012 R2](deliver-extended-security-updates.md) through Azure Arc-enabled servers. Use these troubleshooting steps to address common issues with ESU licensing, enrollment, resource provider registration, and patch delivery.

## License provisioning issues

If you're unable to provision a Windows Server 2012 ESU license for Azure Arc-enabled servers, verify you meet these conditions:

- **Permissions:** Verify you have sufficient permissions (**Contributor** role or higher) within the scope of ESU provisioning and linking.

- **Core minimums:** Verify that you specified sufficient cores for the ESU license. Physical core-based licenses require a minimum of 16 cores per machine, and virtual core-based licenses require a minimum of eight cores per virtual machine (VM).

- **Conventions:** Verify that you selected an appropriate subscription and resource group and provided a unique name for the ESU license.

## ESU enrollment issues

If you're unable to successfully link your Azure Arc-enabled server to an activated ESU license, verify you meet these conditions:

- **Connectivity:** Azure Arc-enabled server is **Connected**. For information about viewing the status of Azure Arc-enabled machines, see [Agent status](overview.md#agent-status).

- **Agent version:** Connected Machine agent is version 1.34 or higher. If the agent version is less than 1.34, you need to update it to this version or higher.

- **Operating system:** Only Azure Arc-enabled servers running the Windows Server 2012 and Windows Server 2012 R2 operating system are eligible to enroll in ESU.

- **Environment:** The connected machine shouldn't be running on Azure Local, Azure VMware solution (AVS), or as an Azure virtual machine. **In these scenarios, Windows Server 2012 ESU is available for free**. For information about no-cost ESUs through Azure Local, see [Free Extended Security Updates through Azure Local](/azure/azure-local/manage/azure-benefits-esu?tabs=windows-server-2012).

- **License properties:** Verify the license is activated and allocated sufficient physical or virtual cores to support the intended scope of servers.

## Resource providers

If you're unable to enable this service offering, review the resource providers registered on the subscription. If you receive an error while attempting to register the resource providers, validate the role assignments on the subscription. Also review any potential Azure policies that may be set with a **Deny** policy, preventing the enablement of these resource providers:

- **Microsoft.HybridCompute:** This resource provider is essential for Azure Arc-enabled servers, allowing you to onboard and manage on-premises servers in the Azure portal.

- **Microsoft.GuestConfiguration:** Enables Guest Configuration policies, which are used to assess and enforce configurations on your Arc-enabled servers for compliance and security.

- **Microsoft.Compute:** This resource provider is required for Azure Update Management, which is used to manage updates and patches on your on-premises servers, including ESU updates.

- **Microsoft.Security:** Enabling this resource provider is crucial for implementing security-related features and configurations for both Azure Arc and on-premises servers.

- **Microsoft.OperationalInsights:** This resource provider is associated with **Azure Monitor and Log Analytics**, which is used for monitoring and collecting telemetry data from your hybrid infrastructure, including on-premises servers.

- **Microsoft.Sql:** If you're managing on-premises SQL Server instances and require ESU for SQL Server, enabling this resource provider is necessary.

- **Microsoft.Storage:** Enabling this resource provider is important for managing storage resources, which may be relevant for hybrid and on-premises scenarios.

## ESU patch issues

### ESU patch status

To detect whether your Azure Arc-enabled servers are patched with the most recent Windows Server 2012 (R2) ESU, use Azure Update Manager or Azure Policy. The [(Preview): Extended Security Updates should be installed on Windows Server 2012 Arc machines](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetail.ReactView/id/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F14b4e776-9fab-44b0-b53f-38d2458ea8be/version~/null/scopes~/%5B%22%2Fsubscriptions%2F4fabcc63-0ec0-4708-8a98-04b990085bf8%22%5D) policy checks whether the most recent Windows Server 2012 ESU patches were received.

Both of these options are available at no additional cost for Azure Arc-enabled servers enrolled in Windows Server 2012 ESU enabled by Azure Arc.

### ESU prerequisites

Ensure that both the licensing package and servicing stack update (SSU) are downloaded for the Azure Arc-enabled server as documented in [KB5031043: Procedure to continue receiving security updates after extended support ended on October 10, 2023](https://support.microsoft.com/topic/kb5031043-procedure-to-continue-receiving-security-updates-after-extended-support-has-ended-on-october-10-2023-c1a20132-e34c-402d-96ca-1e785ed51d45). Ensure you're following all of the networking prerequisites as documented in [Prepare to deliver Extended Security Updates for Windows Server 2012](prepare-extended-security-updates.md?tabs=azure-cloud#networking).

## Troubleshooting errors

### Error: Trying to check IMDS again (HRESULT 12002 or 12029)

When you install the ESU enabled by Azure Arc and it fails with the following Instance Metadata Service (IMDS) errors:

```error
ESU: Trying to Check IMDS Again LastError=HRESULT_FROM_WIN32(12002)

ESU: Trying to Check IMDS Again LastError=HRESULT_FROM_WIN32(12029)
```

You may need to update the intermediate certificate authorities trusted by your computer using one of the following methods.

> [!IMPORTANT]
> If you're running the [latest version of the Azure Connected machine agent](agent-release-notes.md), it's not necessary to install the intermediate CA certificates or allow access to the Public Key Infrastructure (PKI) URL. However, if a license was already assigned before the agent was upgraded, it can take up to 15 days for the older license to be replaced. During this time, the intermediate cert is still required. After upgrading the agent, you can delete the license file `%ProgramData%\AzureConnectedMachineAgent\certs\license.json` to force it to be refreshed.

#### Option 1: Allow access to the PKI URL

Configure your network firewall and/or proxy server to allow access from the Windows Server 2012 (R2) machines to `http://www.microsoft.com/pkiops/certs` and `https://www.microsoft.com/pkiops/certs` (both TCP 80 and 443). This enables the machines to automatically retrieve any missing intermediate CA certificates from Microsoft.

```powershell
# Define firewall rule name
$ruleNameHttp = "Allow_HTTP_to_MicrosoftPKI"
$ruleNameHttps = "Allow_HTTPS_to_MicrosoftPKI"

# Allow outgoing traffic to any IP address on TCP port 80 (HTTP)
New-NetFirewallRule -DisplayName $ruleNameHttp `
    -Direction Outbound `
    -Action Allow `
    -Protocol TCP `
    -RemotePort 80 `
    -Profile Any `
    -Description "Allow outbound HTTP traffic to Microsoft PKI OPS"

# Allow outgoing traffic to any IP address on TCP port 443 (HTTPS)
New-NetFirewallRule -DisplayName $ruleNameHttps `
    -Direction Outbound `
    -Action Allow `
    -Protocol TCP `
    -RemotePort 443 `
    -Profile Any `
    -Description "Allow outbound HTTPS traffic to Microsoft PKI OPS"
```

Once the network changes are made to allow access to the PKI URL, try installing the Windows updates again. You may need to reboot your computer for the automatic installation of certificates and validation of the license to take effect.

#### Option 2: Manually download and install the intermediate CA certificates

If you're unable to allow access to the PKI URL from your servers, you can manually download and install the certificates on each machine.

1. On any computer with internet access, download these intermediate CA certificates:

   1. [Microsoft Azure RSA TLS Issuing CA 03](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2003%20-%20xsign.crt)
   1. [Microsoft Azure RSA TLS Issuing CA 04](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2004%20-%20xsign.crt)
   1. [Microsoft Azure RSA TLS Issuing CA 07](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2007%20-%20xsign.crt)
   1. [Microsoft Azure RSA TLS Issuing CA 08](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20RSA%20TLS%20Issuing%20CA%2008%20-%20xsign.crt)

1. Copy the certificate files to your Windows Server 2012 (R2) machines.
1. Run any one set of the following commands in an elevated command prompt or PowerShell session to add the certificates to the "Intermediate Certificate Authorities" store for the local computer. The command should be run from the same directory as the certificate files. These commands are safe to run multiple times and if the certificate is already installed, nothing changes.

   ```powershell
   certutil -addstore CA "Microsoft Azure RSA TLS Issuing CA 03 - xsign.crt"
   certutil -addstore CA "Microsoft Azure RSA TLS Issuing CA 04 - xsign.crt"
   certutil -addstore CA "Microsoft Azure RSA TLS Issuing CA 07 - xsign.crt"
   certutil -addstore CA "Microsoft Azure RSA TLS Issuing CA 08 - xsign.crt"
   ```

1. Try installing the Windows updates again. You may need to reboot your computer for the validation logic to recognize the newly imported intermediate CA certificates.

### Error: Not eligible (HRESULT 1633)

If you encounter the error `ESU: not eligible HRESULT_FROM_WIN32(1633)`, run the following command:

```powershell
Remove-Item "$env:ProgramData\AzureConnectedMachineAgent\Certs\license.json" -Force
Restart-Service HIMDS
```

If you have other issues receiving ESU after successfully enrolling the server through Arc-enabled servers, or you need additional information related to issues affecting ESU deployment, see [Troubleshoot issues in ESU](/troubleshoot/windows-client/windows-7-eos-faq/troubleshoot-extended-security-updates-issues).
