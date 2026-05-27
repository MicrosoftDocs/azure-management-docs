---
title: FIPS 140-3 and Cryptography in Azure Linux
description: Learn how Azure Linux supports FIPS 140-3 compliance by using validated cryptographic modules, enforcing approved algorithms, and providing mechanisms to enable and validate FIPS mode.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# FIPS 140-3 and cryptography in Azure Linux

This article explains Azure Linux FIPS 140-3 compliance, the crypto-policies framework, post-quantum cryptography (PQC) support, and how to enable and validate FIPS mode on Azure Linux.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## FIPS 140-3 overview

FIPS 140-3 is a U.S. government standard that defines security requirements for cryptographic modules. A cryptographic module can be hardware, software, or firmware that implements approved security functions. The Cryptographic Module Validation Program (CMVP) validates modules against FIPS 140-3 requirements. The Cryptographic Algorithm Validation Program (CAVP) validates approved algorithms used by those modules.

FIPS 140-3 certification is critical for government and regulated industry adoption. Azure Linux provides FIPS 140-3 support at no extra cost.

## Azure Linux and FIPS

Azure Linux supports FIPS mode by using validated cryptographic modules and enforcing approved algorithms when FIPS is enabled.

### Validate FIPS mode

Use an Azure Linux FIPS-enabled image, and then run the validation checks to confirm FIPS is active.

For general-purpose Azure Linux virtual machines (VMs), use a FIPS-enabled Azure Linux image. For Azure Kubernetes Service (AKS) clusters, ensure the node pool is configured to use FIPS-enabled Azure Linux nodes.

Use the following checks to confirm FIPS is enabled:

```bash
cat /proc/sys/crypto/fips_enabled
sysctl crypto.fips_enabled
openssl md5
```

Expected results:

- `/proc/sys/crypto/fips_enabled` returns `1`.
- `sysctl crypto.fips_enabled` returns `crypto.fips_enabled = 1`.
- `openssl md5` fails with a FIPS-related error.

### Algorithms disabled in FIPS mode

When FIPS mode is enabled, the following algorithms and features are disabled to comply with FIPS 140-3 requirements:

| Algorithm or feature | Reason |
| -------------------- | ------ |
| MD5 | Not FIPS approved |
| SHA-1 (for signatures) | Deprecated in FIPS |
| RSA < 2048-bit | Key size too small |
| DH < 2048-bit | Key size too small |
| X25519/Ed25519 | Not NIST curves |
| ChaCha20-Poly1305 | Not FIPS approved |
| Triple-DES | Deprecated |

## Crypto-policies framework

Azure Linux uses the Fedora crypto-policies framework to centralize cryptographic configuration across all applications and libraries. This provides a unified, automated approach to managing cryptographic settings rather than configuring each application individually.

### Baseline cryptographic policy

The default crypto policy enforces the following minimum standards:

| Feature | Minimum standard |
| ------- | ---------------- |
| TLS | 1.2 or later only (SSLv3, TLS 1.0, and TLS 1.1 are disabled) |
| Signatures | SHA-1 isn't permitted for digital signatures |
| RSA | Minimum 2048-bit key size, 3072-bit preferred |
| Ciphers | 3DES, RC4, and other deprecated ciphers are disabled |

### View and change the active crypto policy

1. Install the `crypto-policies-scripts` package if it isn't already installed.

    ```bash
    sudo dnf install -y crypto-policies-scripts
    ```

1. View or update the active crypto policy:

    ```bash
    # View the current policy
    update-crypto-policies --show
    
    # Set a stricter policy
    sudo update-crypto-policies --set FUTURE
    
    # Set FIPS policy
    sudo update-crypto-policies --set FIPS
    ```

## Post-quantum cryptography (PQC)

Azure Linux includes support for the following post-quantum cryptography (PQC) algorithms to mitigate _harvest now, decrypt later_ quantum threats:

- **OpenSSL 3.5+** with native Module-Lattice-Based Key Encapsulation Mechanism (ML-KEM) support
- **OpenSSH** with ML-KEM hybrid key exchange (`mlkem768x25519-sha256`)

## Build FIPS-compliant RPMs

If you build or ship RPMs, ensure they use SHA-256 digests and validate them properly in FIPS mode.

### Validate RPM digests

To validate that an RPM uses SHA-256 digests, use the `rpm -Kv` command against the package you want to check.

```bash
rpm -Kv <YOUR_RPM_PACKAGE>
```

Expected output includes:

- Header SHA256 digest: OK
- Payload SHA256 digest: OK

If the RPM only contains SHA1 or MD5 digests, rebuild with SHA-256 enabled.

### Build RPMs with fpm

If you use `fpm` to build RPMs, set the digest explicitly:

```bash
fpm --rpm-digest sha256 ...
```

### Container builds

FIPS mode inside a container follows the host configuration. If the host is FIPS-enabled, container workloads inherit that mode.

## Configure OpenSSL cipher suites (optional)

You can optionally set a stricter default cipher list in the OpenSSL configuration file.

1. Print the OpenSSL configuration directory:

    ```bash
    openssl version -d
    ```

1. Edit `openssl.cnf` and add a `default_conf` section with the cipher policy you need.
1. Verify the active cipher list:

    ```bash
    openssl ciphers -s
    ```

## Related content

- [Configure applications to use system crypto libraries in Azure Linux](./system-crypto-libraries.md)
- [Kernel livepatching on Azure Linux](./kernel-live-patching.md)
- [Manage CVEs on Azure Linux and Azure Container Linux (ACL)](./manage-cves.md)
- [Azure Linux certifications and compliance](./certifications-compliance.md)
