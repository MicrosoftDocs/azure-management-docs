---
title: Azure Container Storage enabled by Azure Arc using Azure Arc Jumpstart
description: Learn about Azure Arc Jumpstart and Azure Container Storage enabled by Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: overview
ms.date: 03/12/2025

# Customer intent: As an IT professional, I want to deploy Azure Container Storage on an AKS Edge Essentials instance, so that I can efficiently manage and store data from real-time AI analysis of video feeds in an edge computing environment.
---

# Azure Arc Jumpstart and Azure Container Storage enabled by Azure Arc

Azure Container Storage enabled by Azure Arc partnered with [Azure Arc Jumpstart](https://azurearcjumpstart.com/) to produce both a new Arc Jumpstart scenario and Azure Arc Jumpstart Drops, furthering the capabilities of edge computing solutions. This partnership led to an innovative scenario in which a computer vision AI model detects defects in bolts from real-time video streams, with the identified defects securely stored using Azure Container Storage on an AKS Edge Essentials instance. This scenario showcases the powerful integration of Azure Arc with AI and edge storage technologies.

Additionally, Azure Container Storage contributed to Azure Arc Jumpstart Drops, a curated collection of resources that simplify deployment and management for developers and IT professionals. These tools, including Kubernetes files and scripts, are designed to streamline edge storage solutions and demonstrate the practical applications of Microsoft's cutting-edge technology.

## Azure Arc Jumpstart scenario using Azure Container Storage enabled by Azure Arc

Azure Container Storagey collaborated with the [Azure Arc Jumpstart](https://azurearcjumpstart.com/) team to implement a scenario in which a computer vision AI model detects defects in bolts by analyzing video from a supply line video feed streamed over Real-Time Streaming Protocol (RTSP). The identified defects are then stored in a container within a storage account using Azure Container Storage.

In this automated setup, Azure Container Storage is deployed on an [AKS Edge Essentials](/azure/aks/hybrid/aks-edge-overview) single-node instance, running in an Azure virtual machine. An Azure Resource Manager template is provided to create the necessary Azure resources and configure the **LogonScript.ps1** custom script extension. This extension handles AKS Edge Essentials cluster creation, Azure Arc onboarding for the Azure VM and AKS Edge Essentials cluster, and Azure Container Storage deployment. Once AKS Edge Essentials is deployed, Azure Container Storage is installed as a Kubernetes service that exposes a CSI driven storage class for use by applications in the Edge Essentials Kubernetes cluster.

For more information, see the following articles:

- [Jumpstart scenario: Fault Detection with Azure Container Storage on AKS Edge Essentials single-node deployment](https://jumpstart.azure.com/azure_arc_jumpstart/azure_edge_iot_ops/aks_edge_essentials_single_acsa)
- [Watch the Jumpstart scenario on YouTube](https://youtu.be/Qnh2UH1g6Q4).

## Azure Arc Jumpstart Drops for Azure Container Storage enabled by Azure Arc

[Jumpstart Drops](https://aka.ms/jumpstartdrops) is a curated online collection of tools, scripts, and other assets that simplify the daily tasks of developers, IT, OT, and day-2 operations professionals. Jumpstart Drops is designed to showcase the power of Microsoft's products and services and promote mutual support and knowledge sharing among community members.

Azure Container Storage created Jumpstart Drops as part of another collaboration with [Azure Arc Jumpstart](https://azurearcjumpstart.com/):

- [Cloud Ingest Edge Volume on a Single Node Ubuntu K3s Cluster with an SFTP Front End](https://jumpstart.azure.com/azure_jumpstart_drops?drop=Azure%20Container%20Storage%20enabled%20by%20Azure%20Arc%20SFTP&fs=true) : This example can be used to install Azure Container Storage enabled by Azure Arc to provide a ReadWriteMany Cloud Ingest Edge Volume on an Ubuntu system with K3s and an SFTP front end.
- [Create an Azure Container Storage enabled by Azure Arc Edge Volumes with CloudSync](https://jumpstart.azure.com/azure_jumpstart_drops?drop=Create%20an%20Azure%20Container%20Storage%20enabled%20by%20Azure%20Arc%20Edge%20Volumes%20with%20CloudSync&fs=true) : This example can be used to install Azure Container Storage enabled by Azure Arc to provide a Cloud Backed ReadWriteMany Edge Volume on an Ubuntu system with K3s.
- [Create an Azure Container Storage enabled by Azure Arc Edge Volumes Local FS instance](https://jumpstart.azure.com/azure_jumpstart_drops?drop=Create%20an%20Azure%20Container%20Storage%20enabled%20by%20Azure%20Arc%20Edge%20Volumes%20Local%20FS%20instance&fs=true): This example can be used to install Azure Container Storage to provide a local unbacked ReadWriteMany Edge Volume on an Ubuntu system with K3s.

## Next steps

- [Azure Container Storage enabled by Azure Arc overview](overview.md)
- [AKS Edge Essentials overview](/azure/aks/hybrid/aks-edge-overview)
