---
title: "What's new with Azure Arc resource bridge"
ms.date: 09/09/2026
ms.topic: concept-article
description: "Learn about the latest releases of Azure Arc resource bridge."
# Customer intent: "As a cloud operations manager, I want to stay informed about the latest updates and features for the Arc resource bridge, so that I can ensure my deployment is secure, efficient, and compliant with the evolving platform requirements."
---

# What's new with Azure Arc resource bridge

To stay up to date with the most recent developments, this article provides you with information about recent releases of the Arc resource bridge Azure CLI extension, `az arcappliance`.

The [version support policy](overview.md#supported-versions) for Arc resource bridge generally covers versions released within the last six months or within the latest n-3 versions, **whichever is more recent**. Even if a version is within the version support policy (n-3), manually upgrade the appliance at least once every six months. This schedule ensures the internal components and certificates are refreshed. You can check your appliance version and the version release date for an estimate on the last upgrade date. When a patch version is released, the upgrade path might skip the minor version and directly upgrade to the patch version. In such cases, the supported versions (n-3) exclude the skipped minor version and include the patch version instead.

## Version 1.8.0 (July 2026)

- Support version: n
- Appliance: 1.8.0
- CLI extension: 1.8.0
- Kubernetes: 1.33.5
- Mariner: 3.0.20260517

### Arc resource bridge platform

- Kubernetes service CIDR range reduced to 10.96.0.0/24 for new deployments.
- Specific error code returned for proxy-update conflicts with an in-flight upgrade or agent-update operation.
- Diagnostic checker includes CertificateExpiration checker for kubeadm certs. New health check that flags upcoming or expired kubeadm-managed certificates.
- Enable Azure Arc Gateway for Arc-enabled VMware vSphere (Preview) for new deployments.



## Version 1.7.0 (Dec 2025)

- Supported version: n-1 (extended support)
- Appliance: 1.7.0
- CLI extension: 1.7.0
- Kubernetes: 1.32.6
- Mariner: 3.0.20251030

### Arc resource bridge platform

- Update the network proxy settings with the new command: `az arcappliance configuration proxy update`. This command requires a resource bridge that's deployed or upgraded to 1.7.0. This command supports only Azure Local and VMware. If an ARB upgrade fails because of incorrect or outdated network proxy settings, run a proxy update to fix the values. If a proxy update operation fails, retry it and ensure it succeeds before performing any other operation, including retrying upgrade.
- New CLI command to show the local resource bridge configuration: `az arcappliance configuration show`. For ARM configuration, continue to use: `az arcappliance show`.
- Deployment blocks configuration settings that overlap with Service CIDR (10.96.0.0/12).
- Returns an error for VMware credentials that contain an invalid character (single quote).
- New user management key is downloaded by using the `az arcappliance get-credentials` CLI command. Use this key to update network proxy settings. This key downloads only for resource bridges on version 1.7.0.
- The `--config-file` argument is now optional for the `az arcappliance create` command. If you don't provide it, pass these arguments: `--name`, `--resource-group`, and `--location`.
- The `--config-file` argument is now optional for the `az arcappliance delete` command. If you don't provide it, pass these arguments based on private cloud type:
  - Azure Local: `--name`, `--resource-group`.
  - VMware: `--name`, `--resource-group`, `--datacenter`, `--datastore`, `--folder`.
  - SCVMM: `--name`, `--resource-group`.
- The `--config-file` argument is now optional for the `az arcappliance upgrade` command. If you don't provide it, pass these arguments: `--name`, `--resource-group`, and `--kubeconfig`. Retrieve the kubeconfig by using the CLI command `az arcappliance get-credentials`. This command supports only Azure Local and VMware.

## Version 1.6.0 (Sept 2025)

- Support version: out of support
- Appliance: 1.6.0
- CLI extension: 1.6.0
- Kubernetes: 1.31.5
- Mariner: 3.0.20250402

### Arc resource bridge platform

- [PREVIEW] Node Identity feature added.
- Enabled managed identity at the appliance VM layer.
- [PREVIEW] Arc Gateway feature added for Arc-enabled VMware.
- KVAIO cleanup and optimizations for cloud authentication or RBAC failures.
- Improvements to Validate, CreateConfig, and error messages.
- Pass network profile even when proxy isn't enabled.
- Bump Kubernetes SDK to 1.32.0.1.

## Version 1.5.0 (June 2025)

- Support version: out of support
- Appliance: 1.5.0
- CLI extension: 1.5.0
- Kubernetes: 1.30.4
- Mariner: 3.0.20250402

### Arc resource bridge platform

- Added cloud logs collection feature: Automatically collects logs and uploads them to the cloud when deployment fails.
- Prevent nonessential debug file operations from blocking critical functionality.
- Sign ARB container image and remove old unsigned cached images.
- Add validation to prevent network overlaps with the K8s Pod CIDR 10.244.0.0/16.
- Add validation to warn about network overlaps with the K8s Service CIDR 10.96.0.0/12.
- Add validation to prevent the use of proxy URLs that end in .local.
- Fix bug in vSphere Cluster Client Set command that creates conflicts.
- Download SDK dynamic parts - reduce the number of concurrent downloads with each retry.
- Add kms-plugin token rotation to credential rotation.


## Version 1.4.1 (February 2025)

- Support version: out of support
- Appliance: 1.4.0
- CLI extension: 1.4.0
- Kubernetes: 1.30.4
- Mariner: 3.0.20250102

### Bug fixes

- Fix for compatibility with Azure CLI v2.70.0. From this version forward, Azure CLI version needs to be 2.70.0 or higher.

> [!NOTE]
> This patch version of the Azure CLI extension `az arcappliance` doesn't change the appliance version. Therefore, `az arcappliance` CLI extension 1.4.1 and 1.4.0 both have the same appliance version, 1.4.0.

## Version 1.4.0 (February 2025)

- Support version: out of support
- Appliance: 1.4.0
- CLI extension: 1.4.0
- Kubernetes: 1.30.4
- Mariner: 3.0.20250102

### Arc-enabled SCVMM

- Validate command - Add custom time-outs.

### Arc resource bridge platform

- Enhanced telemetry for error type categorization.
- Support for US Gov Virginia/Fairfax region.

## Version 1.3.1 (December 2024)

> [!NOTE]
> This `az arcappliance` Azure CLI extension requires Azure CLI v2.69.0 or lower. It isn't compatible with Azure CLI v2.70.0 or higher.
>

- Support version: out of support
- Appliance: 1.3.1
- CLI extension: 1.3.1
- Kubernetes: 1.29.4
- Mariner: 2.0.20241029

### Arc-enabled SCVMM

- CreateConfig CLI command - Improve prompt messages, reorder Library Share input prompt.
- CreateConfig CLI command - Display Library Share, Cloud Names, and IP Pools inputs in alphabetical order.
- Image provisioning from remote machine - Decompress Vhdx disk space error message improvement.
- Add retry and error message improvement for SCVMM createClient.
- Validate VLAN ID check error message improvement.
- Add TSG link in error message - validate checks, prep-createclient, createVM.

### Arc resource bridge platform

- Error category framework update.

### Bug fixes

- Azure Stack HCI CVE fix.

## Version 1.3.0 (October 2024)

- Support version: skipped, upgrades go directly to patch version 1.3.1
- Appliance: 1.3.0
- CLI extension: 1.3.0
- SFS release: 0.1.34.10926
- Kubernetes: 1.29.4
- Mariner: 2.0.20240609

### Arc-enabled SCVMM

- Validation - fail if user isn't part of an admin user group like DomainAdmins.
- Conditional validation on gateway IP for SCVMM IP pool scenario and sshkeygen removal.
- Silently clean appliance VM resources like HW profiles, ISO files, and VM templates in delete command.
- CAPVMM update to 1.1.19.
- SCVMM image provisioning decompress Mariner Vhdx disk space error message improvement.
- SCVMM appliance deployment failing in Deploy due to IPPool missing access to HG.

### Arc-enabled VMware vSphere

- Remove root folder privilege validations from vSphere.
- Extra validations on the canary image.

### Arc resource bridge platform

- New error extra info field to add more context to errors.
- Add ACR image pull test suite.
- Add timeout for API server endpoint.
- Added DNSError category.

### Bug fixes

- CVE fixes.

## Version 1.2.0 (July 2024)

- Appliance: 1.2.0
- CLI extension: 1.2.0
- SFS release: 0.1.32.10710
- Kubernetes: 1.28.5
- Mariner: 2.0.20240609

### Arc-enabled SCVMM

- `CreateConfig`: Improve prompt messages and reorder networking prompts for the custom IP range scenario.
- `CreateConfig`: Validate Gateway IP input against specified IP range for the custom IP range scenario.
- `CreateConfig`: Add validation to check infra configuration capability for HA VM deployment. If HA isn't supported, reprompt users to proceed with standalone VM deployment.

### Arc-enabled VMware vSphere

- Improve prompt messages in createconfig for VMware.
- Validate proxy scheme and check for required `no_proxy` entries.

### Features

- Reject double commas (`,,`) in `no_proxy` string.
- Add default folder to createconfig list.
- Add conditional Fairfax URLs for US Gov Virginia support.
- Add new error codes.

### Bug fixes

- Fix for openSSH [CVE-2024-63870](https://github.com/advisories/GHSA-2x8c-95vh-gfv4).

## Version 1.1.1 (April 2024)

- Appliance: 1.1.1
- CLI extension: 1.1.1
- SFS release: 0.1.26.10327
- Kubernetes: 1.27.3
- Mariner: 2.0.20240301

### Arc-enabled SCVMM

- Add quotes for resource names.

### Azure Stack HCI

- HCI auto rotation logic on upgrade.

### Features

- Update log collection with describe nodes.
- Error message enhancement for failure to reach Arc resource bridge VM.
- Improve troubleshoot command error handling with scoped access key.
- Longer timeout for individual pod pulls.
- Update `execute` command to allow passing in a kubeconfig.
- Catch `<>` in no_proxy string.
- Add validation to check if connections from the client machine are proxied.
- Diagnostic checker enhancement - Add default gateway and DNS servers check to telemetry mode.
- Log collection enhancement.

### Bug fixes

- HCI MOC image client fix to set storage container on catalog.

## Version 1.1.0 (April 2024)

- Appliance: 1.1.0
- CLI extension: 1.1.0
- SFS release: 0.1.25.10229
- Kubernetes: 1.27.3
- Mariner: 2.0.20240223

### Arc-enabled SCVMM

- Use same `vmnetwork` key for HG and Cloud (`vmnetworkid`).
- SCVMM - Add fallback for VMM IP pool with support for IP range in appliance network, add `--vlanid` parameter to accept `vlanid`.
- Non-interactive mode for SCVMM `troubleshoot` and `logs` commands.
- `Createconfig` command uses styled text to warn about saving config files instead of standard logger.
- Improved handling and error reporting for time-outs while provisioning or deprovisioning images from the cloud fabric.
- Verify template and snapshot health after provisioning an image, and clean up files associated with the template on image deprovision failures.
- Missing VHD state handling in SCVMM.
- SCVMM `validate` and `createconfig` fixes.

### Arc-enabled VMware vSphere

- SSD storage validations added to VMware vSphere in telemetry mode to check if the ESXi host backing the resource pool has any SSD-backed storage.
- Improved missing privilege error message, and show some privileges in error message.
- Validate host ESXi version and provide a clear error message for placement profile.
- Improve message for no datacenters found, and display default folder.
- Surface VMware error when finder fails during validate.
- Verify template health and fix it during image provision.

### Features

- `deploy` command - diagnostic checker enhancements that add retries with exponential backoff to proxy client calls.
- `deploy` command - diagnostic checker enhancement: adds storage performance checker in telemetry mode to evaluate the storage performance of the VM used to deploy the appliance.
- `deploy` command - Add timeout for SSH connection: New error message: "Error: Timeout occurred due to management machine being unable to reach the appliance VM IP, 192.168.0.11. Ensure that the requirements are met: `https://aka.ms/arb-machine-reqs: dial tcp 192.168.0.11:22: connect: connection timed out`".
- `validate` command - The appliance deployment now fails if Proxy Connectivity and No Proxy checks report any errors.

### Bug fixes

- SCVMM ValueError fix - fallback option for VMM IP Pools with support for Custom IP Range based Appliance Network.

## Version 1.0.18 (February 2024)

- Appliance: 1.0.18
- CLI extension: 1.0.3
- SFS release: 0.1.24.10201
- Kubernetes: 1.26.6
- Mariner: 2.0.20240123

### Fabric and private cloud provider

- SCVMM `createconfig` command improvements - retry until valid port and FQDN provided.
- SCVMM and VMware - Validate control plane IP address; add reprompts.
- SCVMM and VMware - extend `deploy` command timeout from 30 to 120 minutes.

### Features

- `deploy` command - diagnostic checker enhancement: proxy checks in telemetry mode.

### Product

- Reduction in CPU requests.
- ETCD preflight check enhancements for upgrade.

### Bug fixes

- Fix for clusters impacted by the `node-ip` being set as `kube-vip` IP issue.
- Fix for SCVMM cred rotation with the same credentials.

## Version 1.0.17 (December 2023)

- Appliance: 1.0.17
- CLI extension: 1.0.2
- SFS release: 0.1.22.11107
- Kubernetes: 1.26.6
- Mariner: 2.0.20231106

### Fabric/Private cloud provider

- SCVMM `createconfig` command improvements.
- Azure Local - extend `deploy` command timeout from 30 to 120 minutes.
- All private clouds - enable provider credential parameters to be passed in each command.
- All private clouds - basic validations for select `createconfig` command inputs.
- VMware - basic reprompts for select `createconfig` command inputs.

### Features

- `deploy` command - diagnostic checker enhancement - improve `context` error messages.

### Bug fixes

- Fix for `context` error always being returned as `Deploying`.

### Known bugs

- Arc resource bridge upgrade shows appliance version as upgraded, but status shows upgrade failed.

## Version 1.0.16 (November 2023)

- Appliance: 1.0.16
- CLI extension: 1.0.1
- SFS release: 0.1.21.11013
- Kubernetes: 1.25.7
- Mariner: 2.0.20231004

### Fabric/Private cloud provider

- SCVMM image provisioning and upgrade fixes.
- VMware vSphere - use full inventory path for networks.
- VMware vSphere error improvement for denied permission.
- Azure Stack HCI - enable default storage container.

### Features

- `deploy` command - diagnostic checker enhancement - add `azurearcfork8s.azurecr.io` URL.

### Bug fixes

- vSphere credential issue.
- Don't set storage container for non-`arc-appliance` catalog image provision requests.
- Monitoring agent not installed issue.

## Version 1.0.15 (September 2023)

- Appliance: 1.0.15
- CLI extension: 1.0.0
- SFS release: 0.1.20.10830
- Kubernetes: 1.25.7
- Mariner: 2.0.20230823

### Fabric/Infrastructure

- `az arcappliance` CLI commands now only support static IP deployments for VMware and SCVMM.
- For test purposes only, you can deploy Arc resource bridge on Azure Stack HCI with a DHCP configuration.
- Support for using canonical region names.
- Removal of VMware vSphere 6.7 fabric support (vSphere 7 and 8 are both supported).

### Features

- (new) `get-upgrades` command - fetches the new upgrade edge available for a current appliance cluster.
- (new) `upgrade` command - upgrades the appliance to the next available version (not available for SCVMM).
- (update) `deploy` command - In addition to `deploy`, this command now also calls `create` command. `Create` command is now optional.
- (new) `get-credentials` command - now allows fetching of SSH keys and kubeconfig, which are needed to run the `logs` command from a different machine than the one used to deploy Arc resource bridge.
- Allowing usage of `config-file` parameter for `get-credentials` command.
- (new) `troubleshoot` command - helps debug live-site issues by running allowed actions directly on the appliance using a JIT access key.

### Bug fix

- IPClaim premature deletion issue for vSphere static IP.

## Next steps

- Learn more about [Arc resource bridge](overview.md).
- Learn how to [upgrade Arc resource bridge](upgrade.md).
