---
title: Monitor and Manage Virtual Machines from the Azure mobile app
description: Use the Azure mobile app to monitor and manage virtual machines.
ms.date: 02/21/2025
ms.topic: how-to
# Customer intent: As a cloud administrator, I want to monitor and manage Azure virtual machines via a mobile app, so that I can ensure service availability and respond to incidents while on the go.
---

# Manage virtual machines in the Azure mobile app

With the Azure mobile app you can monitor, manage, and troubleshoot your virtual machines (VMs) while you're on the go, all from your mobile device. 

Common use cases for the virtual machines feature include: 
- **On-the-go management**: If a VM becomes unresponsive, quickly restart it to maintain service availability and minimize downtime.
- **Incident response**: When you receive an Azure alert, you can easily check the health of your VM.
- **Access control**: Manage user permissions for a VM while you're on the go.
- **Monitoring and status checks**: Check in on your VM's health, performance metrics, and activity.

## Navigate to your virtual machine

For on-the-go access to your virtual machine, you'll first navigate to it.

1. Open the Azure mobile app and sign in.
2. Select **Virtual Machines** from the home screen or search for a specific VM in the search bar.
3. Tap on the VM.

From the VM's page, you have options to:
- Monitor
  - View VM status like running, stopped, and deallocated.
  - Check CPU, memory disk, and network usage through real-time metrics.
  - Access logs and recent activity.
  - View diagnostic logs and troubleshoot incidents remotely.

  :::image type="content" source="media/virtual-machines/virtual-machines-details.jpg" alt-text="Screenshot showing the VM details view in the Azure mobile app.":::

- Manage
  - Start, connect to, or restart the VM.
  - Stop or hibernate a VM.
  - Configure auto-shutdown settings.
  - Access VM properties, including OS details and configurations.

:::image type="content" source="media/virtual-machines/virtual-machines-autoshutdown.jpg" alt-text="Screenshot showing the VM autoshutdown view in the Azure mobile app.":::

> [!TIP] 
> Integrate with [Azure Monitor](/azure/azure-monitor/vm/vminsights-enable) to get access to deeper insights in the Azure mobile app. 

## Next steps

- Learn more about the [Azure mobile app](overview.md).
- Download the Azure mobile app for free from the [Apple App Store](https://aka.ms/ReferAzureIOSMSLearnMobileAppDocs), [Google Play](https://aka.ms/azureapp/android/doc) or [Amazon App Store](https://aka.ms/azureapp/amazon/doc).
