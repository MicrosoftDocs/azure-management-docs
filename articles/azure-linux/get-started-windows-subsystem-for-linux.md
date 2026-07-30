---
title: Get started with Azure Linux 4.0 on Windows Subsystem for Linux (WSL)
description: Learn about resources you can use to quickly learn and start using Azure Linux on Windows Subsystem for Linux (WSL) for development, testing, and troubleshooting.
author: farhakareem
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: get-started
ms.date: 07/25/2026
---

# Get started with Azure Linux 4.0 on Windows Subsystem for Linux (WSL)

This guide helps you install Azure Linux on Windows Subsystem for Linux (WSL) and start using it as a local Linux environment for development, testing, and troubleshooting without needing a virtual machine (VM) or an Azure subscription.

Azure Linux on WSL uses the same command-line tools and behavior as Azure Linux running in Azure, so you can use it for day-to-day work and to reproduce issues locally.

## Prerequisites

- Windows 10 (22H2) or Windows 11.
- WSL 2 installed and enabled. To install or enable, see [How to install Linux on Windows with WSL](/windows/wsl/install).
- An internet connection.
- Permissions to install WSL distributions.

## Install Azure Linux 4.0 on WSL

1. Open **Windows Terminal** (no admin required).
1. Follow the [GitHub README guide](https://github.com/microsoft/azurelinux/blob/4.0/README.md) to install Azure Linux.
1. Launch Azure Linux:

    ```bash
    wsl -d AzureLinux-4
    ```

On first launch, you're prompted to create a Linux username and password. Use these credentials only inside the Azure Linux environment.

## Verify the installation

After signing in, validate that Azure Linux is installed and running as expected.

```bash
cat /etc/os-release
```

You see output indicating **Azure Linux** and the installed version number.

At this point, Azure Linux is fully installed and ready to use.

## Update packages (recommended)

After installation, update your system packages to ensure you have the latest fixes and security updates.

```bash
sudo dnf update -y
```

## Use Visual Studio Code with Azure Linux (optional)

> [!NOTE]
> Visual Studio Code isn't required, but you can use it with WSL for editing, debugging, and terminal workflows.

1. Install [Visual Studio Code](https://code.visualstudio.com/) on Windows.
1. Install the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension.
1. From an Azure Linux terminal, open a folder in VS Code.

    ```bash
    code .
    ```

For more information, see the [Visual Studio Code documentation](https://code.visualstudio.com/Docs).

## Related content

- [Overview of Azure Linux on Windows Subsystem for Linux (WSL)](./windows-subsystem-for-linux-overview.md)
