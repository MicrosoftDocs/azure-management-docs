---
title: Use system cryptographic libraries in Azure Linux
description: Read and change the Azure Linux system cryptographic policy, confirm which libraries an application uses, and tell algorithm availability apart from what a connection negotiates.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/13/2026
---

# Use system cryptographic libraries in Azure Linux

Cryptographic behavior depends on the selected system policy, the libraries
an application loads, and the application's configuration.

## Default system cryptographic policy

Azure Linux 4 installs the `crypto-policies` package and selects `DEFAULT`.
This policy sets cryptographic defaults for participating applications.
Applications can override those defaults or use their own libraries.

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.

## Verify the cryptographic policy, libraries, and negotiated algorithms

Check the selected policy, the application's library and configuration
choices, and an actual connection. These checks answer different questions.

### Read the system cryptographic policy

The policy sets algorithm and protocol defaults for the applications that
consume it. Read which policy is selected:

```bash
cat /etc/crypto-policies/config
```

Expected result for the default configuration:

```output
DEFAULT
```

Inspect the generated back-end configuration:

```bash
ls -l /etc/crypto-policies/back-ends/
```

These files supply settings to supported cryptographic back ends. Their
presence doesn't prove an application reads them. Check the application's
configuration and restart affected processes after a policy change.

### Confirm which libraries an application uses

For a trusted binary, inspect its shared-library dependencies. For example:

```bash
ldd /usr/bin/openssl | grep -E 'libcrypto|libssl'
```

Matching lines identify the libraries being loaded. They **don't** prove the
application follows the system policy: an application can override protocols,
ciphers, or configuration loading while using the same libraries.

If there are no matches, investigate static linking, libraries loaded by a
runtime module, and bundled cryptography. For example, Python loads OpenSSL
through its SSL extension; inspecting only the Python executable can miss it.
Containers also have their own libraries, which host updates don't replace.

Check where OpenSSL reads its configuration and providers:

```bash
openssl version -a
```

`OPENSSLDIR` and `MODULESDIR` show the build's default configuration and
provider locations. Check application settings and overrides such as
`OPENSSL_CONF` before assuming those defaults are used.

### Tell availability apart from negotiation

Listing an algorithm proves the library implements it. It says nothing about
what a given connection uses.

List the key encapsulation mechanisms OpenSSL provides:

```bash
openssl list -kem-algorithms
```

The default provider exposes ML-KEM-512, ML-KEM-768, and ML-KEM-1024.

List the key exchange methods OpenSSH supports:

```bash
ssh -Q kex | grep -i mlkem
```

```output
mlkem768x25519-sha256
```

Now check what's configured and what's negotiated:

```bash
sudo sshd -T | grep -E '^(kexalgorithms|ciphers|macs) '
ssh -v <user-name>@<host-name> exit 2>&1 | grep 'kex: algorithm'
openssl s_client -connect <host-name>:443 -brief </dev/null
```

The first command shows the SSH server configuration; `Match` blocks might
require connection-specific inspection. The second reports the negotiated SSH
key exchange. The third reports the Transport Layer Security (TLS) protocol
and cipher for that connection. These results don't establish what every
client can negotiate or whether all prohibited combinations are rejected.

## Change the cryptographic policy or add an optional provider

Keep `DEFAULT` unless an approved requirement calls for a different policy.
Install an additional provider only if the application needs it.

### Change the policy

> [!CAUTION]
> A stricter policy can break connections to clients, agents, or external
> services. Test compatibility in nonproduction before applying it to a
> production workload. Don't use a policy named `FIPS` as a substitute for
> published FIPS validation and configuration guidance.

The `update-crypto-policies` command ships in `crypto-policies-scripts`, which
is included in the image. Read the current policy:

```bash
update-crypto-policies --show
```

If the policy is unchanged, the result is:

```output
DEFAULT
```

Before changing it, record the current value for rollback:

```bash
previous_policy=$(update-crypto-policies --show)
printf '%s\n' "$previous_policy"
sudo update-crypto-policies --set <policy-name>
update-crypto-policies --show
```

Replace `<policy-name>` with the reviewed policy you intend to use. Restart all
affected applications; if you can't identify all consumers, schedule a VM
restart. Test required client connections and inspect what they negotiate.
Reading the policy name alone doesn't verify application behavior.

To roll back in the same shell:

```bash
sudo update-crypto-policies --set "$previous_policy"
```

Restart the affected processes again and repeat the connection checks. If you
have restarted the VM or closed the shell, use the value you recorded earlier.

A policy name that references a compliance standard describes algorithm
selection. It isn't an assessment, a validation, or evidence that a
cryptographic module is certified.

### Install the optional OQS provider

The Open Quantum Safe provider is available in the repositories and isn't
installed by default:

```bash
dnf info oqsprovider
sudo dnf install -y oqsprovider
```

Install it only when an application needs algorithms it supplies. Installing a
provider doesn't activate it: OpenSSL loads providers according to its
configuration, and applications still have to select the algorithms. Confirm
what's actually loaded:

```bash
openssl list -providers
```

## Troubleshoot cryptographic policy and handshake failures

### update-crypto-policies: command not found

The command ships in `crypto-policies-scripts`, which is included in the image.
If it was removed, reinstall it:

```bash
sudo dnf install -y crypto-policies-scripts
```

### Connections fail after a policy change

Check that the peer supports the permitted protocols, ciphers, and key sizes.
Restore the recorded previous policy if needed, not automatically `DEFAULT`,
which might be different from your earlier configuration. Restart affected
applications after the rollback.

### An application's behavior doesn't change with the policy

Check for application overrides, a different configuration file, bundled
libraries, and processes that haven't restarted. Library linkage alone doesn't
identify which of these applies.

### An algorithm is listed but the handshake fails

A listed algorithm can be excluded by service configuration or unsupported by
the peer. Check the permitted algorithms and configuration at both ends of the
connection.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Manage security updates](./manage-common-vulnerabilities-exposures.md)
- [Certificate management](./certificate-management.md)
- [OpenSSL documentation](https://docs.openssl.org/master/man1/openssl-list/)
- [ML-KEM algorithm standard (NIST FIPS 203)](https://csrc.nist.gov/pubs/fips/203/final)
