---
title: "What's new with Azure Arc resource bridge"
ms.date: 03/19/2025
ms.topic: concept-article
description: "Learn about the latest releases of Azure Arc resource bridge."
# Customer intent: "As a cloud operations manager, I want to stay informed about the latest updates and features for the Arc resource bridge, so that I can ensure my deployment is secure, efficient, and compliant with the evolving platform requirements."
---

# What's new with Azure Arc resource bridge

To stay up to date with the most recent developments, this article provides you with information about recent releases of the Arc resource bridge Azure CLI extension, `az arcappliance`.  

The [version support policy](overview.md#supported-versions) for Arc resource bridge generally covers version(s) released within the last 6 months or within the latest n-3 versions, whichever is more recent. Even if a version is within the version support policy (n-3), the appliance should be upgraded at least once every six months. This is to ensure the internal components and certificates are refreshed. You can check your appliance version and the version release date for an estimate on the last upgrade date. When a patch version is released, the upgrade path might skip the minor version and directly upgrade to the patch version. In such cases, the supported versions (n-3) exclude the skipped minor version and include the patch version instead.

## Version 1.6.0 (Sept 2025)

- Support version: n
- Appliance: 1.6.0
- CLI extension: 1.6.0
- Kubernetes: 1.31.5
- Mariner: 3.0.20250402

### Arc resource bridge platform

- [PREVIEW] Node Identity feature added
- Enabled managed identity at the appliance VM layer
- [PREVIEW] Arc Gateway feature added for Arc-enabled VMware
- KVAIO cleanup and optimizations for cloud login or RBAC failures
- Improvements to Validate, CreateConfig, and error messages
- Pass network profile even when proxy is not enabled
- Bump Kubernetes SDK to 32.0.1


## Version 1.5.0 (June 2025)

- Support version: n-1
- Appliance: 1.5.0
- CLI extension: 1.5.0
- Kubernetes: 1.30.4
- Mariner: 3.0.20250402

### Arc resource bridge platform

- Cloud logs collection feature added: Enables automatic log collection and upload to cloud on deployment failure

- Prevent non-essential debug file operations from blocking critical functionality
- ARB container image signed and removed old unsigned cached images
- Added validation to prevent network overlaps with the K8s Pod CIDR 10.244.0.0/16
- Added validation to warn for network overlaps with the K8s Service CIDR 10.96.0.0/12
- Added validation to prevent the usage of Proxy URLs ending in .local.
- Addressed bug in vSphere Cluster Client Set command creating conflicts.
- Downloadsdk dyanmic parts - reducing the number of concurrent downloads with each retry.
- Added kms-plugin token rotation to credential rotation


## Version 1.4.1 (February 2025)

- Support version: n-2
- Appliance: 1.4.0
- CLI extension: 1.4.0
- Kubernetes: 1.30.4
- Mariner: 3.0.20250102

### Bug fixes

- fix for compatibility with Azure CLI v2.70.0. From this version forward, Azure CLI version needs to be 2.70.0 or higher.

> [!NOTE]
> This patch version of the Azure CLI extension `az arcappliance` doesn't change the appliance version. Therefore, `az arcappliance` CLI extension 1.4.1 and 1.4.0 both have the same appliance version, 1.4.0.

## Version 1.4.0 (February 2025)

- Support version: n-3
- Appliance: 1.4.0
- CLI extension: 1.4.0
- Kubernetes: 1.30.4
- Mariner: 3.0.20250102

### Arc-enabled SCVMM

- Validate command - Add custom time-outs

### Arc resource bridge platform

- Enhanced telemetry for error type categorization
- Support for US Gov Virginia/Fairfax region

## Version 1.3.1 (December 2024)

> [!NOTE]
> This `az arcappliance` Azure CLI extension requires Azure CLI v2.69.0 or below. It isn't compatible with Azure CLI v2.70.0 or higher.

- Appliance: 1.3.1
- CLI extension: 1.3.1
- Kubernetes: 1.29.4
- Mariner: 2.0.20241029

### Arc-enabled SCVMM

- CreateConfig CLI command - Improve prompt messages, reorder Library Share input prompt
- CreateConfig CLI command - Display Library Share, Cloud Names, and IP Pools inputs in alphabetical order
- Image Provisioning from remote machine - Decompress Vhdx disk space error message improvement
- Add retry and error message improvement for SCVMM createClient
- Validate VLAN ID check error message improvement
- Add TSG link in error message - validate checks, prep-createclient, createVM

### Arc resource bridge platform

- Error category framework update

### Bug fixes

- Azure Stack HCI CVE fix

## Version 1.3.0 (October 2024)

- Support version: skipped, upgrades go directly to patch version 1.3.1
- Appliance: 1.3.0
- CLI extension: 1.3.0
- SFS release: 0.1.34.10926
- Kubernetes: 1.29.4
- Mariner: 2.0.20240609

### Arc-enabled SCVMM

- Validation - fail if user isn't part of an Admin User Group like DomainAdmins
- Conditional Validation on Gateway IP for SCVMM IP Pool Scenario and sshkeygen removal
- Silently clean appliance VM resources like HW profiles, ISO Files, and VM templates in delete command
- CAPVMM update to 1.1.19
- SCVMM Image Provisioning Decompress Mariner Vhdx disk space error message improvement
- SCVMM appliance deployment failing in Deploy due to IPPool missing access to HG

### Arc-enabled VMware vSphere

- Remove root folder privilege validations from vSphere
- extra validations on the canary image

### Arc resource bridge platform

- New error additional info field to add more context to errors
- Add ACR image pull test suite
- Add time-out for API Server Endpoint
- Added DNSError category

### Bug fixes

- CVE fixes

## Version 1.2.0 (July 2024)

- Appliance: 1.2.0
- CLI extension: 1.2.0
- SFS release: 0.1.32.10710
- Kubernetes: 1.28.5
- Mariner: 2.0.20240609

### Arc-enabled SCVMM

- `CreateConfig`: Improve prompt messages and reorder networking prompts for the custom IP range scenario
- `CreateConfig`: Validate Gateway IP input against specified IP range for the custom IP range scenario
- `CreateConfig`: Add validation to check infra configuration capability for HA VM deployment. If HA isn't supported, reprompt users to proceed with standalone VM deployment

### Arc-enabled VMware vSphere

- Improve prompt messages in createconfig for VMware
- Validate proxy scheme and check for required `no_proxy` entries

### Features

- Reject double commas (`,,`) in `no_proxy` string
- Add default folder to createconfig list
- Add conditional Fairfax URLs for US Gov Virginia support
- Add new error codes

### Bug fixes

- Fix for openSSH [CVE-2024-63870](https://github.com/advisories/GHSA-2x8c-95vh-gfv4)

## Version 1.1.1 (April 2024)

- Appliance: 1.1.1
- CLI extension: 1.1.1
- SFS release: 0.1.26.10327
- Kubernetes: 1.27.3
- Mariner: 2.0.20240301

### Arc-enabled SCVMM

- Add quotes for resource names

### Azure Stack HCI

- HCI auto rotation logic on upgrade

### Features

- Updated log collection with describe nodes
- Error message enhancement for failure to reach Arc resource bridge VM
- Improve troubleshoot command error handling with scoped access key
- Longer time-out for individual pod pulls
- Updated `execute` command to allow passing in a kubeconfig
- Catch `<>` in no_proxy string
- Add validation to see if connections from the client machine are proxied
- Diagnostic checker enhancement - Add default gateway and dns servers check to telemetry mode
- Log collection enhancement

### Bug fixes

- HCI MOC image client fix to set storage container on catalog

## Version 1.1.0 (April 2024)

- Appliance: 1.1.0
- CLI extension: 1.1.0
- SFS release: 0.1.25.10229
- Kubernetes: 1.27.3
- Mariner: 2.0.20240223

### Arc-enabled SCVMM

- Use same `vmnetwork` key for HG and Cloud (`vmnetworkid`)
- SCVMM - Add fallback for VMM IP pool with support for IP range in appliance network, add `--vlanid` parameter to accept `vlanid`
- Non-interactive mode for SCVMM `troubleshoot` and `logs` commands
- `Createconfig` command uses styled text to warn about saving config files instead of standard logger
- Improved handling and error reporting for time-outs while provisioning/deprovisioning images from the cloud fabric
- Verify template and snapshot health after provisioning an image, and clean up files associated to the template on image deprovision failures
- Missing VHD state handing in SCVMM
- SCVMM `validate` and `createconfig` fixes

### Arc-enabled VMware vSphere

- SSD storage validations added to VMware vSphere in telemetry mode to check if the ESXi host backing the resource pool has any SSD-backed storage
- Improve missing privilege error message, and show some privileges in error message
- Validate host ESXi version and provide a concrete error message for placement profile
- Improve message for no datacenters found, and display default folder
- Surface VMware error when finder fails during validate
- Verify template health and fix it during image provision

### Features

- `deploy` command - diagnostic checker enhancements that add retries with exponential backoff to proxy client calls
- `deploy` command - diagnostic checker enhancement: adds storage performance checker in telemetry mode to evaluate the storage performance of the VM used to deploy the appliance
- `deploy` command - Add time-out for SSH connection: New error message: "Error: Timeout occurred due to management machine being unable to reach the appliance VM IP, 192.168.0.11. Ensure that the requirements are met: `https://aka.ms/arb-machine-reqs: dial tcp 192.168.0.11:22: connect: connection timed out`"
- `validate` command - The appliance deployment now fails if Proxy Connectivity and No Proxy checks report any errors

### Bug fixes

- SCVMM ValueError fix - fallback option for VMM IP Pools with support for Custom IP Range based Appliance Network

## Version 1.0.18 (February 2024)

- Appliance: 1.0.18
- CLI extension: 1.0.3
- SFS release: 0.1.24.10201
- Kubernetes: 1.26.6
- Mariner: 2.0.20240123

### Fabric/Private cloud provider

- SCVMM `createconfig` command improvements - retry until valid Port and FQDN provided
- SCVMM and VMware - Validate control plane IP address; add reprompts
- SCVMM and VMware - extend `deploy` command time-out from 30 to 120 minutes

### Features

- `deploy` command - diagnostic checker enhancement: proxy checks in telemetry mode

### Product

- Reduction in CPU requests
- ETCD preflight check enhancements for upgrade

### Bug fixes

- Fix for clusters impacted by the `node-ip` being set as `kube-vip` IP issue
- Fix for SCVMM cred rotation with the same credentials

## Version 1.0.17 (December 2023)

- Appliance: 1.0.17
- CLI extension: 1.0.2
- SFS release: 0.1.22.11107
- Kubernetes: 1.26.6
- Mariner: 2.0.20231106

### Fabric/Private cloud provider

- SCVMM `createconfig` command improvements
- Azure Local - extend `deploy` command time-out from 30 to 120 minutes
- All private clouds - enable provider credential parameters to be passed in each command
- All private clouds - basic validations for select `createconfig` command inputs
- VMware - basic reprompts for select `createconfig` command inputs

### Features

- `deploy` command - diagnostic checker enhancement - improve `context` error messages

### Bug fixes

- Fix for `context` error always being returned as `Deploying`

### Known bugs

- Arc resource bridge upgrade shows appliance version as upgraded, but status shows upgrade failed

## Version 1.0.16 (November 2023)

- Appliance: 1.0.16
- CLI extension: 1.0.1
- SFS release: 0.1.21.11013
- Kubernetes: 1.25.7
- Mariner: 2.0.20231004

### Fabric/Private cloud provider

- SCVMM image provisioning and upgrade fixes
- VMware vSphere - use full inventory path for networks
- VMware vSphere error improvement for denied permission
- Azure Stack HCI - enable default storage container

### Features

- `deploy` command - diagnostic checker enhancement - add `azurearcfork8s.azurecr.io` URL

### Bug fixes

- vSphere credential issue
- Don't set storage container for non-`arc-appliance` catalog image provision requests
- Monitoring agent not installed issue

## Version 1.0.15 (September 2023)

- Appliance: 1.0.15
- CLI extension: 1.0.0
- SFS release: 0.1.20.10830
- Kubernetes: 1.25.7
- Mariner: 2.0.20230823

### Fabric/Infrastructure

- `az arcappliance` CLI commands now only support static IP deployments for VMware and SCVMM
- For test purposes only, Arc resource bridge on Azure Stack HCI may be deployed with DHCP configuration
- Support for using canonical region names
- Removal of VMware vSphere 6.7 fabric support (vSphere 7 and 8 are both supported)

### Features

- (new) `get-upgrades` command- fetches the new upgrade edge available for a current appliance cluster
- (new) `upgrade` command - upgrades the appliance to the next available version (not available for SCVMM)
- (update) `deploy` command - In addition to `deploy`, this command now also calls `create` command. `Create` command is now optional.
- (new) `get-credentials` command - now allows fetching of SSH keys and kubeconfig, which are needed to run the `logs` command from a different machine than the one used to deploy Arc resource bridge
- Allowing usage of `config-file` parameter for `get-credentials` command
(new) Troubleshoot command - help debug live-site issues by running allowed actions directly on the appliance using a JIT access key

### Bug fix

- IPClaim premature deletion issue vSphere static IP

## Next steps

- Learn more about [Arc resource bridge](overview.md).
- Learn how to [upgrade Arc resource bridge](upgrade.md).
