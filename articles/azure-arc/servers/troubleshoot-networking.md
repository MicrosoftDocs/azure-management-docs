---
title: Troubleshoot Azure Arc-enabled servers networking issues
description: This article tells how to troubleshoot and resolve networking issues with Azure Arc-enabled servers.
ms.date: 01/14/2026
ms.topic: troubleshooting
ms.custom:
  - build-2025
# Customer intent: "As a system administrator, I want to troubleshoot Azure Connected Machine agent connection errors, so that I can ensure successful configuration and management of Azure Arc-enabled servers."
---

# Troubleshoot Azure Arc-enabled servers networking issues

This article provides information for troubleshooting networking issues that might occur with Arc-enabled servers.

When troubleshooting connectivity issues, be sure that your environment meets all of the [Azure Arc-enabled servers networking requirements](./agent-overview.md).

## Windows TLS configuration issues

Starting from version 1.56 of the Connected Machine agent for Windows (excluding Windows Server 2012 and Windows Server 2012 R2), if the agent fails to reach Azure endpoints even after the endpoints are allowed in the environment, ensure the following cipher suites are enabled for at least one of the recommended TLS versions:

- TLS 1.3 (suites in server-preferred order):
  - TLS_AES_256_GCM_SHA384 (0x1302)   ECDH secp521r1 (eq. 15360 bits RSA)   FS
  - TLS_AES_128_GCM_SHA256 (0x1301)   ECDH secp256r1 (eq. 3072 bits RSA)   FS
- TLS 1.2 (suites in server-preferred order)
  - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH secp521r1 (eq. 15360 bits RSA)   FS
  - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH secp256r1 (eq. 3072 bits RSA)   FS

You can check the cipher suites on a machine with the following PowerShell command:

```powershell
Get-TlsCipherSuite | Format-List Name
```

To enable cipher suites, you can use one of the following methods.

> [!NOTE]
> For domain-joined machines, Group Policy Objects (GPOs) override local policies, so GPOs need to be updated to enable required cipher suites.

### Enable cipher suites with Group Policy

1. [Open the **SSL Cipher Suite Order** window](/windows-server/security/tls/manage-tls).
1. Edit the **Cipher Suites** box (comma-separated list) to include the minimum required cipher suites.

### Enable cipher suites with PowerShell (no reboot required)

For TLS 1.3:

```powershell
Enable-TlsCipherSuite -Name "TLS_AES_256_GCM_SHA384"
Enable-TlsCipherSuite -Name "TLS_AES_128_GCM_SHA256"
```

For TLS 1.2:

```powershell
Enable-TlsCipherSuite -Name "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
Enable-TlsCipherSuite -Name "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
```

### Enable cipher suites with manual registry edit

1. Navigate to the registry key: `HKLM\SYSTEM\CurrentControlSet\Control\Cryptography\Configuration\Local\SSL\00010002`.
1. Edit the 'Functions' `REG_MULTI_SZ` value to add the required cipher suites to the list (with each cipher suite on its own line).
1. Reboot the machine for changes to take effect.

## Next steps

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Open a support request to get assistance. For more information, see [Create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
