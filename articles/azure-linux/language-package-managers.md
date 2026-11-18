---
title: Secure language package managers on Azure Linux
description: Understand the default sources and settings for pip, npm, Go, and Cargo on Azure Linux, and configure them to reduce supply-chain risk.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/30/2026
---

# Secure language package managers on Azure Linux

Azure Linux 4 packages are signed and installed with DNF. Language package
managers, such as pip, npm, Go modules, and Cargo, download code from public
registries instead. That code doesn't pass through Azure Linux package signing,
so its integrity depends on how you configure each tool.

## Default pip, npm, Go, and Cargo sources and settings

The language package managers aren't installed in the general-purpose image.
They're available from the Azure Linux repository in the `python3-pip`,
`nodejs-npm`, `golang`, and `cargo` packages.

| Tool | Default source | Defaults that affect supply-chain risk |
| ---- | -------------- | -------------------------------------- |
| pip | Python Package Index (PyPI) | No system configuration file. The Python installation has no `EXTERNALLY-MANAGED` marker, so `sudo pip install` isn't blocked. It installs into `/usr/local/lib/python3.<minor>/site-packages`, which takes precedence over packages installed by DNF. |
| npm | `https://registry.npmjs.org/` | `/etc/npmrc` sets the global prefix to `/usr/local`. Package install scripts run (`ignore-scripts=false`), `npm audit` runs on install, and TLS certificate checks are on. |
| Go | `https://proxy.golang.org,direct` | `/usr/lib/golang/go.env` sets `GOSUMDB=sum.golang.org`, so downloaded modules are checked against the checksum database. It also sets `GOTOOLCHAIN=local`, so the `go` command doesn't download a different toolchain. |
| Cargo | crates.io | No system configuration file. A build can update `Cargo.lock` when it doesn't match `Cargo.toml`, unless you pass `--locked`. |

Two of these defaults leave supply-chain controls to you:

- **npm runs package install scripts.** `ignore-scripts` is `false`, so a
  dependency's `preinstall`, `install`, and `postinstall` scripts run with the
  permissions of the user who runs `npm install`. See
  [Install npm dependencies from a lock file](#install-npm-dependencies-from-a-lock-file).
- **Python has no `EXTERNALLY-MANAGED` marker.** pip doesn't stop you from
  installing packages into the system Python environment. See
  [Isolate Python packages](#isolate-python-packages).

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.
- The package manager you use, installed with DNF. For example:

  ```bash
  sudo dnf install -y python3-pip nodejs-npm golang cargo
  ```

## Verify pip, npm, Go, and Cargo supply-chain settings

Check the effective settings for each tool before you depend on them. Each
command requires the matching package from [Prerequisites](#prerequisites).
On a VM without it, the shell reports `command not found`.

### pip

List the configuration files pip reads and any values they set. This command
requires the `python3-pip` package:

```bash
pip3 config debug
```

Expected result on a new VM when you run the command as `root`:

```output
env_var:
env:
global:
  /etc/xdg/pip/pip.conf, exists: False
  /etc/pip.conf, exists: False
site:
  /usr/pip.conf, exists: False
user:
  /root/.pip/pip.conf, exists: False
  /root/.config/pip/pip.conf, exists: False
```

No global or user file exists. Check for packages installed outside DNF, which
take precedence over packages installed by DNF:

```bash
ls /usr/local/lib/python3.*/site-packages
```

Expected result when no packages were installed outside DNF:

```output
ls: cannot access '/usr/local/lib/python3.*/site-packages': No such file or directory
```

### npm

Read the registry and the settings that control install scripts. These
commands require the `nodejs-npm` package:

```bash
npm config get registry
npm config get ignore-scripts
cat /etc/npmrc
```

Expected result for the default configuration:

```output
https://registry.npmjs.org/
false
prefix=/usr/local
python=/usr/bin/python3
update-notifier=false
```

### Go

Read the module source, checksum database, and toolchain settings. This
command requires the `golang` package:

```bash
go env GOPROXY GOSUMDB GOTOOLCHAIN
```

Expected result for the default configuration:

```output
https://proxy.golang.org,direct
sum.golang.org
local
```

With `GOTOOLCHAIN=local`, a module that requires a newer Go version fails
instead of downloading a toolchain:

```output
go: go.mod requires go >= <version> (running go <installed-version>; GOTOOLCHAIN=local)
```

Install a newer `golang` package when one is available, instead of changing
this setting.

### Cargo

Check for configuration files that change the registry or sources:

```bash
ls ~/.cargo/config.toml .cargo/config.toml 2>/dev/null
```

No output means Cargo uses crates.io.

## Pin and verify language package dependencies

Pin dependencies, verify their integrity, and keep language packages out of the
system Python and Node.js installations.

### Isolate Python packages

Install Python packages in a virtual environment instead of with `sudo pip`:

```bash
python3 -m venv <environment-path>
<environment-path>/bin/pip install -r requirements.txt
```

To stop pip from installing outside a virtual environment, set
`require-virtualenv` in the global configuration:

```bash
sudo pip3 config --global set global.require-virtualenv true
```

pip then refuses installs outside a virtual environment:

```output
ERROR: Could not find an activated virtualenv (required).
```

If a package was already installed with `sudo pip`, remove it to restore the
version installed by DNF:

```bash
sudo pip3 uninstall <package-name>
```

### Require hashes for Python dependencies

With hash-checking mode, pip installs only packages that match the hashes in
your requirements file:

```bash
<environment-path>/bin/pip install --require-hashes -r requirements.txt
```

When a requirement has no hash, pip refuses to install and prints the hash of
the downloaded file, for example:

```output
ERROR: Hashes are required in --require-hashes mode, but they are missing from some requirements.
```

Get hashes from a trusted source, such as a lock file generated in a controlled
environment, rather than copying them from a failed install.

### Install npm dependencies from a lock file

Commit `package-lock.json`, and install with `npm ci`, which installs exactly
the versions in the lock file and fails if `package.json` doesn't match it:

```bash
npm ci
```

To stop package install scripts from running, set `ignore-scripts` in the
global configuration:

```bash
sudo npm config set ignore-scripts true --location=global
```

The setting is written to `/etc/npmrc`. Some packages need their install
scripts to build native code. Test your application, and run the scripts for
those packages explicitly.

### Keep Go module verification on

Keep `GOSUMDB` and `GOTOOLCHAIN=local`. For private modules, exclude only your
own module paths from the public proxy and checksum database:

```bash
go env -w GOPRIVATE=<module-path-prefix>
```

`go env -w` writes to your user Go configuration file and overrides
`/usr/lib/golang/go.env`. Don't set `GOSUMDB=off`, `GONOSUMDB`, or
`GOINSECURE` for public modules.

> [!NOTE]
> The Azure Linux `go` command accepts a non-local `GOTOOLCHAIN` value only
> when `MS_GOTOOLCHAIN_ALLOW_NON_LOCAL=1` is set. Toolchains that `go`
> downloads aren't built or updated by Azure Linux.

### Build Rust projects from the lock file

Commit `Cargo.lock` for applications, and build with `--locked` so that Cargo
fails instead of changing dependency versions:

```bash
cargo build --locked
```

To use a private registry or a mirror, configure source replacement as
described in the
[Cargo source replacement documentation](https://doc.rust-lang.org/cargo/reference/source-replacement.html).

### Use an internal mirror

To control which package versions your builds can download, point each tool at
an internal mirror or artifact feed that you manage:

| Tool | Setting |
| ---- | ------- |
| pip | `sudo pip3 config --global set global.index-url <mirror-url>` |
| npm | `sudo npm config set registry <mirror-url> --location=global` |
| Go | `go env -w GOPROXY=<mirror-url>` |
| Cargo | Source replacement in `.cargo/config.toml` |

## Troubleshoot pip, npm, and Go package installs

### A system tool fails after sudo pip install

A package in `/usr/local/lib/python3.<minor>/site-packages` takes precedence
over the version installed by DNF, for every Python program on the VM. Find the
package that Python loads:

```bash
python3 -c 'import <module-name>; print(<module-name>.__file__)'
```

If the path is under `/usr/local`, remove the package with
`sudo pip3 uninstall <package-name>`.

### npm install fails after setting ignore-scripts

A dependency needs its install script, for example to compile native code.
Run the scripts for that package explicitly, or install with
`npm ci --ignore-scripts=false` for a reviewed lock file.

### go build fails with GOTOOLCHAIN=local

The module requires a newer Go version than the installed `golang` package.
Update the package with `sudo dnf upgrade golang`, or use a module version
that supports the installed Go version.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Manage security updates](./manage-common-vulnerabilities-exposures.md)
- [Build hardened software](./build-hardening.md)
- [Package repositories](./package-repositories.md)
- [pip secure installs](https://pip.pypa.io/en/stable/topics/secure-installs/)
- [Go module authentication](https://go.dev/ref/mod#authenticating)
