---
title: Configure Applications to Use System Crypto Libraries in Azure Linux
description: Learn how to configure applications on Azure Linux to use the system cryptographic libraries, ensuring they follow the active FIPS mode and crypto-policies framework.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: concept-article
ms.date: 04/27/2026
---

# Configure applications to use system crypto libraries in Azure Linux

Using system crypto libraries helps you align application behavior with platform security policies, including FIPS mode and the [crypto-policies framework](./fips-cryptography.md#crypto-policies-framework). This article provides an overview of system crypto usage in Azure Linux and how to ensure applications link against the system OpenSSL and other cryptographic libraries rather than bundling their own.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Why use system crypto libraries

System crypto libraries provide the following benefits when used by applications on Azure Linux:

- **Consistent cryptography behavior** across the operating system (OS) and applications. When you enable FIPS mode or change the system crypto policy, all applications using system libraries automatically follow the new configuration.
- **Simplified compliance validation** by enforcing FIPS mode and other compliance requirements at the OS level rather than per application.
- **Reduced maintenance** for bundled crypto libraries. Security updates to OpenSSL and other system libraries are applied centrally.

## Languages and runtimes

Most language runtimes on Azure Linux delegate cryptographic operations to the system OpenSSL library, which means they pick up the active [crypto-policy](./fips-cryptography.md#crypto-policies-framework) and FIPS mode automatically. This is the case for Python (`ssl`/`hashlib`), Ruby (`OpenSSL` binding), Perl (`Net::SSLeay`), PHP (`openssl` extension), and Node.js when linked against the system OpenSSL.

For runtimes that ship their own crypto stack (Go, Java, .NET, Rust), the steps to opt into system crypto vary by runtime and version.

## Post-quantum cryptography (PQC)

Applications using system crypto libraries can take advantage of post-quantum cryptography (PQC) support provided by OpenSSL 3.5+, including **Module-Lattice-Based Key Encapsulation Mechanism (ML-KEM)** for hybrid key exchange.

For more information on PQC capabilities, see [FIPS 140-3 and cryptography](./fips-cryptography.md#post-quantum-cryptography-pqc).

## Verify a binary uses system crypto

Once you've built or installed an application that links against the system crypto libraries, confirm it dynamically links to OpenSSL:

```bash
ldd ./your-binary | grep -E 'libcrypto|libssl'
```

A statically linked binary that bundles its own crypto won't show `libcrypto`/`libssl` here.

## Related content

- [FIPS 140-3 and cryptography in Azure Linux](./fips-cryptography.md)
- [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md)
