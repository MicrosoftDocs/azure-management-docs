---
title: Configure Machine to Manage Arc-Enabled Kubernetes Cluster
description: "Learn how to configure a Windows machine with Azure CLI, kubectl, and Helm to manage Azure Arc-enabled Kubernetes clusters using a sample script."
author: cwatson-cat
ms.author: cwatson
ms.topic: reference #Don't change
ms.date: 04/24/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to configure a Windows machine with the necessary tools to manage Azure Arc-enabled Kubernetes clusters, so that I can efficiently deploy and manage Edge RAG workloads.
---

# Script to configure machine to manage Azure Arc-enabled Kubernetes cluster 

To communicate with and configure an Azure Arc-enabled Kubernetes cluster for Edge RAG, set up any Windows machine with tools like Azure CLI, kubectl, and Helm. The machine needs to be able to reach the Kubernetes cluster.

## Sample script

Run this sample script to set up a Windows machine to do the following tasks:

1. Create a folder.
1. Download Azure CLI, Helm, kubectl, and Azure CLI extensions aksarc and Kubernetes-extension.
1. Sign in to Azure using the Azure CLI with the subscription and tenant ID.
1. Connect to the Azure Arc-enabled Kubernetes cluster.
1. Save credentials for kubectl.

```powershell
# Fill the subscription and resource group information 

$sub = ""  # Add subscription guid 

$tenantid = " " # Add tenant id 

$rg = " " # Add resourcegroup name 

$k8scluster =  "<AKS Arc cluster name>" # Add name of the AKS ARC cluster 

$rootfolder = "C:\microsoft.arc.rag" # Define the root folder 

 
function Trace-Output { param([string]$message) Write-Host "$(Get-Date -Format 'HH:mm:ss') - $message" -ForegroundColor Yellow } 

 
# Create the root folder if it doesn't exist 

if (-Not (Test-Path -Path $rootfolder)) { 

    Trace-Output "Creating root folder at $rootfolder"  

    New-Item -ItemType Directory -Path $rootfolder 

} else { 

    Trace-Output "Root folder already exists at $rootfolder"  

} 

 
cd $rootfolder 

# Set the path to the AZ CLI installer 

$azCliInstaller = "$rootfolder\AzureCLI.msi" 

$logFile = "$rootfolder\AzureCLIInstall.log" 

# Set AZ CLI into path 

# Define the correct installation path 

$azCliPath = "C:\Program Files\Microsoft SDKs\Azure\CLI2\wbin" 

 
# Download and install AZ CLI 64-bit if it doesn't exist 

if (!(Test-Path $azCliPath) ) { 

    Trace-Output "Downloading and installing Azure CLI 64-bit"  

    Invoke-WebRequest -Uri "https://aka.ms/installazurecliwindowsx64" -OutFile $azCliInstaller 

    $argStr = "/I $azCliInstaller /quiet /l*v $logFile" 

    Start-Process msiexec.exe -Wait -ArgumentList $argStr 

    Remove-Item .\AzureCLI.msi -Force 

} else { 

    Trace-Output "Azure CLI is already installed" 

} 

# Check if the path exists, if not add AZ cli into the path. 

if (Test-Path $azCliPath) { 

 

    if ($env:Path -notlike "*$azCliPath*") { 

        Trace-Output "Azure CLI path added to the PATH environment variable." 

        $env:Path += ";$azCliPath;" 

    } else { 

        Trace-Output "Azure CLI path is already in the PATH environment variable." 

    } 

 
    # Get the current PATH environment variable 

    $currentPath = [System.Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::Machine) 

 
    # Check if the Azure CLI path is already in the PATH variable 

    if ($currentPath -notlike "*$azCliPath*") {         

        # Add the Azure CLI path to the PATH variable 

        $newPath = "$currentPath;$azCliPath" 

        [System.Environment]::SetEnvironmentVariable("Path", $newPath, [System.EnvironmentVariableTarget]::Machine) 

        Trace-Output "Azure CLI path added to Machine level PATH environment variable." 

    } else { 

        Trace-Output "Azure CLI path is already in Machine level PATH environment variable." 

    } 

} else { 

    Trace-Output "Azure CLI installation path not found." 

    return 

} 

 
# Install AZ CLI extensions if not installed 

$extensions = @("aksarc", "k8s-extension") 

foreach ($extension in $extensions) { 

$extensionInstalled = az extension show --name $extension -o none 2>$null 
 

if ($LASTEXITCODE -ne 0) { 

       Trace-Output "Installing Azure CLI extension: $extension" 

        az extension add --name $extension 

    } else { 

        Trace-Output "Azure CLI extension $extension is already installed"  

    } 

} 

 

# Set the paths for kubectl and helm 

$kubectlPath = "$rootfolder\kubectl.exe" 

$helmPath = "$rootfolder\helm.exe" 

 

# Download kubectl if it doesn't exist 

if (-Not (Test-Path -Path $kubectlPath)) { 

    Trace-Output "Downloading kubectl" 

    Invoke-WebRequest -Uri "https://dl.k8s.io/release/v1.32.0/bin/windows/amd64/kubectl.exe" -OutFile $kubectlPath 

} else { 

    Trace-Output "kubectl is already downloaded" 

} 

 

# Download helm if it doesn't exist 

if (-Not (Test-Path -Path $helmPath)) { 

    Trace-Output "Downloading helm" 

    Invoke-WebRequest -Uri "https://get.helm.sh/helm-v3.17.2-windows-amd64.zip" -OutFile "$rootfolder\helm.zip" 

    Expand-Archive -Path "$rootfolder\helm.zip" -DestinationPath $rootfolder 

    Move-Item -Path "$rootfolder\windows-amd64\helm.exe" -Destination $helmPath 

    Remove-Item -Path "$rootfolder\helm.zip" 

    Remove-Item -Path "$rootfolder\windows-amd64" -Recurse 

} else { 

    Trace-Output "helm is already downloaded" 

} 

 

# Add the root folder to the system PATH if not already added 

$envPath = [System.Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::Machine) 

if (-Not $envPath.Contains($rootfolder)) { 

    $env:Path += ";$rootfolder;" 

    Trace-Output "Adding $rootfolder to system PATH" 

    [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$rootfolder", [System.EnvironmentVariableTarget]::Machine) 

} else { 

    Trace-Output "$rootfolder is already in the system PATH" 

} 

 
# login to azure using device code and set subscription 

az login --use-device-code -t $tenantId 

az account set -s $sub 

az account show 
 

# Save aksarc cluster credentials into kubeconfig for kubectl and helm to work 

Trace-Output "aks cluster name = $k8scluster " 

az aksarc get-credentials --name $k8scluster --resource-group $rg --admin  --only-show-errors
```

## Related content

- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
- [Use cluster connect to securely connect to Azure Arc-enabled Kubernetes clusters](/azure/azure-arc/kubernetes/cluster-connect?tabs=azure-cli)