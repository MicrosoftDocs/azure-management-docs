---
title: Build hardened software on Azure Linux
description: Understand the compile-time hardening flags that Azure Linux RPM build macros apply, apply them to your own builds, and check the protections in a built binary.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Build hardened software on Azure Linux

Azure Linux 4 packages are compiled with hardening flags that make memory
corruption bugs harder to exploit. The same flags are available to you through
the RPM build macros, so software you build can get the same protections.

## Default compiler and linker hardening flags

The `azurelinux-rpm-config` package defines the compiler and linker flags for
RPM builds. Hardened builds are on by default (`%_hardened_build` is `1`), with
`_FORTIFY_SOURCE` level 3, position-independent executables, and full
read-only relocations. Build notes for `annocheck` are also on by default.

The flags apply to builds that use the RPM macros. Running `gcc` directly
doesn't apply them.

| Flag | Protection |
| ---- | ---------- |
| `-D_FORTIFY_SOURCE=3` | Adds compile-time and runtime buffer size checks to common C library functions, such as `memcpy` and `strcpy`. |
| `-D_GLIBCXX_ASSERTIONS` | Adds bounds checks to C++ standard library containers and strings. |
| `-fstack-protector-strong` | Places a canary value on the stack of functions with local arrays or address-taken variables, and stops the process when an overflow changes it. |
| `-fstack-clash-protection` | Probes large stack allocations, so the stack can't skip past the guard page into other memory. |
| `-fcf-protection` | Instruments code for Intel Control-flow Enforcement Technology (CET), indirect branch tracking, and shadow stacks on processors that support it. |
| `-Werror=format-security` | Fails the build when a format function uses a non-literal format string with no arguments. |
| `-fPIE` and `-pie` | Builds position-independent executables, so address space layout randomization applies to the program as well as its libraries. |
| `-Wl,-z,relro` and `-Wl,-z,now` | Resolves all symbols at startup and makes the relocation tables read-only, which blocks overwrites of the global offset table. |

## Prerequisites

- An Azure Linux virtual machine (VM) or build environment.
- A user account with `sudo` privileges.

The compiler, the RPM build tools, the build configuration, and `readelf`
aren't installed on the general-purpose image. Install them before you run the
commands on this page. `readelf` is installed with the compiler toolchain:

```bash
sudo dnf install -y gcc rpm-build azurelinux-rpm-config
```

Without these packages, `rpm --eval` returns the macro names instead of flags,
and `readelf` isn't found.

## Verify hardening flags in a build

Read the flags from the macros, and check the protections in the binaries you
build.

### Read the build flags

Read the macros that control hardening. These commands require the packages
from [Prerequisites](#prerequisites):

```bash
rpm --eval '%{_hardened_build}'
rpm --eval '%{_fortify_level}'
rpm --eval '%{build_cflags}'
rpm --eval '%{build_ldflags}'
```

The first two commands return:

```output
1
3
```

`%{build_cflags}` and `%{build_cxxflags}` include the compiler flags in the
table. `%{build_ldflags}` includes `-Wl,-z,relro` and `-Wl,-z,now`. The PIE
flags come from the spec files that the macros reference under
`/usr/lib/rpm/redhat/`.

### Check a packaged binary

Use `readelf` from `binutils` to check the protections in a binary. `readelf`
requires the packages from [Prerequisites](#prerequisites). This example
checks the OpenSSH server:

```bash
readelf -h /usr/sbin/sshd | grep Type
readelf -d /usr/sbin/sshd | grep -E 'BIND_NOW|FLAGS_1'
readelf -n /usr/sbin/sshd | grep -o 'x86 feature: IBT, SHSTK'
```

```output
  Type:                              DYN (Position-Independent Executable file)
 0x000000000000001e (FLAGS)              BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
x86 feature: IBT, SHSTK
```

- `DYN (Position-Independent Executable file)` and `PIE` confirm a
  position-independent executable.
- `BIND_NOW` and `NOW` confirm that symbols are resolved at startup. Together
  with a `GNU_RELRO` program header, this gives full read-only relocations.
- `IBT, SHSTK` confirms that the binary is marked as compatible with indirect
  branch tracking and shadow stacks.

## Apply hardening flags to your builds

Build your software with the same flags, and confirm the result with
`annocheck`.

### Build an RPM package

`rpmbuild` passes the flags to the compiler through the `%build` macros, such
as `%configure`, `%cmake`, and `%meson`, and through `CFLAGS`, `CXXFLAGS`, and
`LDFLAGS`. Keep the macros in your spec file, and don't overwrite those
variables with your own values. Check the compiler command lines in the build
log to confirm the flags.

### Build outside RPM

For a build that doesn't use `rpmbuild`, export the flags before you configure or compile the project. These commands require the `gcc` and `azurelinux-rpm-config` packages from [Prerequisites](#prerequisites). Without `azurelinux-rpm-config`, `rpm --eval` returns macro names instead of flags.

```bash
export CFLAGS="$(rpm --eval '%{build_cflags}')"
export CXXFLAGS="$(rpm --eval '%{build_cxxflags}')"
export LDFLAGS="$(rpm --eval '%{build_ldflags}')"
```

Build systems that read these variables, such as Autotools, CMake, and Meson, pass them to the compiler. For a direct compiler call, pass them on the command line. The following example compiles a C source file named `hello.c` into a program named `hello`. Replace both names with the source file and program name for your project:

```bash
gcc $CFLAGS $LDFLAGS -o hello hello.c
```

### Check a binary with annocheck

`annocheck` reads the build notes that the macros record and reports each hardening test. Install it and check your binary. The following example checks the `hello` program from the previous section. Replace `hello` with your program name:

```bash
sudo dnf install -y annobin-annocheck
annocheck hello
```

A binary built with the macro flags passes:

```output
Hardened: hello: PASS.
```

The same source compiled with plain `gcc` fails tests such as `bind-now`,
`cf-protection`, and `pie`. Run `annocheck --verbose hello` for the
details of each test.

### Change a hardening setting for one package

Keep the defaults unless a package can't build or run with them. Scope any
change to the package's spec file, and record the reason in a comment.

| Spec file line | Effect |
| -------------- | ------ |
| `%undefine _hardened_build` | Removes the PIE spec files, `-fstack-protector-strong`, and `-Wl,-z,now`. |
| `%define _fortify_level 2` | Uses `_FORTIFY_SOURCE` level 2 instead of 3. |
| `%undefine _annotated_build` | Removes the `annocheck` build notes. `annocheck` can't verify the binary afterward. |

To preview a change before you edit the spec file, pass it to `rpm`. This
command requires the `azurelinux-rpm-config` package from
[Prerequisites](#prerequisites); without it, the command returns no output:

```bash
rpm --define '_fortify_level 2' --eval '%{build_cflags}' | grep -o 'FORTIFY_SOURCE=[0-9]'
```

```output
FORTIFY_SOURCE=2
```

## Troubleshoot hardened builds

### rpm --eval returns the macro name instead of flags

The build configuration isn't installed. Install it:

```bash
sudo dnf install -y azurelinux-rpm-config
```

### A binary fails the annocheck pie or bind-now test

The build didn't use the linker flags. Check that the build system passes
`LDFLAGS`, and that the project doesn't override them. Compare the link command
in the build log with `rpm --eval '%{build_ldflags}'`.

### A program stops with "buffer overflow detected"

`_FORTIFY_SOURCE` found a write past the end of a buffer at runtime. Fix the
code that the core dump or the stack trace identifies. Lowering the fortify
level hides the bug without fixing it.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Inspect kernel hardening](./kernel-hardening.md)
- [Secure language package managers](./language-package-managers.md)
- [GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
