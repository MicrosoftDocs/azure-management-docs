---
title: Sample - Enable GPU for AKS on Azure Arc
description: "Learn about how to enable GPU on Azure Kubernetes Service (AKS) enabled by Azure Arc to meet the prerequisites for Edge RAG."
author: cwatson-cat
ms.author: cwatson
ms.topic: reference #Don't change
ms.date: 05/13/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to enable GPU support on Azure Kubernetes Service (AKS) through Azure Arc, so that I can prepare for deploying Edge RAG applications efficiently.
---


# Enabling GPU on Azure Kubernetes Service (AKS) enabled by Azure Arc

If you need to enable GPUs on the Azure Local cluster nodes to use for Edge RAG Preview, enabled by Azure Arc, we recommend using the sample script in this article. For more information, see [Complete Edge RAG deployment prerequisites](complete-prerequisites.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Sample

To enable GPU on Azure Kubernetes Service (AKS) enabled by Azure Arc, run the following sample code from any of the Azure Local hosts.

```powershell
$pnpScriptBlock= {
    $ErrorActionPreference= "Continue"
     $videoCards =  Get-PnpDevice  | where {$_.friendlyname -eq "3D Video Controller"}  
     $videoCards | select status, class, friendlyname, instanceid 
    
    if($videoCards.InstanceId.StartsWith("PCI\VEN_10DE").count -eq 0) {
    	 Write-Error "HOST $($env:computername) DOES NOT CONTAIN ANY NVIDIA GPUS, RETURNING..."
    	 return;
    	 } 
    
    $gpuCount = $videoCards | Where-Object { $_.InstanceId.StartsWith("PCI\VEN_10DE") } | Measure-Object | Select-Object -ExpandProperty Count
    	   if ($gpuCount -eq 0) {
    	        Write-Error "HOST $($env:computername) DOES NOT CONTAIN ANY NVIDIA GPUS, RETURNING..."
    	        return
    	    } else {
    	
    	        $nvidiagpus = $videoCards  | where {$_.InstanceId.StartsWith("PCI\VEN_10DE") -eq $true}
    	        Write-Host "HOST $($env:computername) CONTAINS $gpuCount NVIDIA GPU(S)"
    	    }
    	
    	    foreach($card in $nvidiagpus ) {
    	        # Ensure the status is either is either error or unknown
    	        Write-Host "Validating card $($card.InstanceId)" 
    	
    	        if($card.Status -ne "Unknown" -and $card.Status -ne "Error"   ) {
    	            Write-Error "card $($card.InstanceId) seems to have driver installed, please manually UNINSTALL NVIDIA driver"
    	            return;
    	        } 
    	
    	        if($card.Status -eq "Unknown") {
    	            Write-Host "card $($card.InstanceId) already in UNKNOWN status, moving on.."
    	            continue;
    	        } else {
    	            #Step 3: dismount the host driver from the host
    	            $deviceLocationPaths = Get-PnpDeviceProperty -KeyName DEVPKEY_Device_LocationPaths -InstanceId $card.instanceid    
    	            Write-Host "working on instanceid = [$($card.deviceid)], first data element= [$($deviceLocationPaths.Data[0])]"
    	            Write-Host "calling Disable-PnpDevice "
    	            Disable-PnpDevice -InstanceId $card.deviceid -Confirm:$false -Verbose
    	            Write-Host "calling Dismount-VMHostAssignableDevice"
    	            Dismount-VMHostAssignableDevice -LocationPath $deviceLocationPaths.Data[0] -Force -Verbose
    	
    	            Write-Host "Checking card status"
    	            $cardAfterDiscount = Get-PnpDevice -InstanceId $card.instanceid 
    	            if($cardAfterDiscount.Status -eq "Unknown") {
    	                Write-Host " [$($env:computername)] GPU status after dismount is in right state = $($cardAfterDiscount.Status )"
    	            } else {
    	                Write-Error "[$($env:computername)]  GPU status after dismount is should be 'Unknown' but is currently = $($cardAfterDiscount.Status ) -- returning"
    	                return;
    	            }
    	        }
    	    }
    	
    # Print the details after the 3d cards
    Get-PnpDevice  | Where-Object {$_.friendlyname -eq "3D Video Controller"} | Select-Object status, class, friendlyname, instanceid
    
    # Step 4: download and install the NVIDIA mitigation driver
    # This needs to be done only once for all cards
    Invoke-WebRequest -Uri "https://docs.nvidia.com/datacenter/tesla/gpu-passthrough/nvidia_azure_stack_inf_v2022.10.13_public.zip" -OutFile "nvidia_azure_stack_inf_v2022.10.13_public.zip"
    	    mkdir nvidia-mitigation-driver -Force
    	    Expand-Archive .\nvidia_azure_stack_inf_v2022.10.13_public.zip .\nvidia-mitigation-driver -Force
    	    Remove-Item .\nvidia_azure_stack_inf_v2022.10.13_public.zip -Force
    	    pushd .\nvidia-mitigation-driver
    	    pnputil /add-driver nvidia_azure_stack_A2_base.inf /install 
    	    pnputil /scan-devices 
    	    popd
    	
     # Print the details after the 3d cards
    $afterDriverInstall = Get-PnpDevice | where {$_.friendlyname -match "Nvidia"}
    $afterDriverInstall  | ft status, class, friendlyname, instanceid 
    	
    	    foreach($card in $afterDriverInstall ) {
    	        if($card.Status -eq "OK" -and $card.FriendlyName -eq "Nvidia A2_base - Dismounted"   ) {
    	            Write-Host " [$($env:computername)]  card $($card.InstanceId) is configured correctly" -ForegroundColor Green
    	
    	            (Get-MocNode -location MocLocation).properties.statuses.Info
    	
    	            write-host "Restarting Moc node agent, to pick up the latest changes"
    	            Restart-Service wssdagent  -Force -Verbose
    	
    	            start-sleep 5
    	            get-service wssdagent -Verbose
    	            
    	            write-host "Get moc status information after service restart"
    	            (Get-MocNode -location MocLocation).properties.statuses.Info
    	            continue;
    	        } else {
    	            Write-Error " [$($env:computername)]  card $($card.InstanceId) is NOT Configured properly" 
    	        }
    	    }
    	}
    	
    	$hcinodes = Get-ClusterNode
    	Write-Host "This HCI cluster has $($hcinodes.count) nodes `n$($hcinodes.Name -join "`n" )"
    	
    	foreach($node in $hcinodes.name)
    	{
    	    Write-Host "===============Processing host $node ================"
    	    $sess = New-PSSession -ComputerName $node
    	    Invoke-Command -Session $sess -ScriptBlock $pnpScriptBlock -verbose
    	
    	    $sess | Remove-PSSession
    	    Write-Host "===============Completed Processing host $node ================"
    	    Start-Sleep 2
    	}
    	
    	write-host "Wait 60 seconds for moc to refresh the status and print the status again" 
    	start-sleep 60
    	
    # This should show info about all the nodes
    write-host "Get moc status information - This should show the adapters on both the nodes"
   (Get-MocNode -location MocLocation).properties.statuses.Info
```

## Related content

- [What you need for Edge RAG](requirements.md)
- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)