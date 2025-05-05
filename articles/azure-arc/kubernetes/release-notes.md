---
title: "What's new with Azure Arc-enabled Kubernetes"
ms.date: 04/29/2025
ms.topic: concept-article
description: "Learn about the latest releases of Arc-enabled Kubernetes."
---

# What's new with Azure Arc-enabled Kubernetes

Azure Arc-enabled Kubernetes is updated on an ongoing basis. To stay up to date with the most recent developments, this article provides you with information about recent releases of the [Azure Arc-enabled Kubernetes agents](conceptual-agent-overview.md).

When any of the Arc-enabled Kubernetes agents are updated, all of the agents in the `azure-arc` namespace are incremented with a new version number, so that the version numbers are consistent across agents. When a new version is released, all of the agents are upgraded together to the newest version (whether or not there are functionality changes in a given agent), unless you [disabled automatic upgrades](agent-upgrade.md) for the cluster.

We generally recommend using the most recent versions of the agents. The [version support policy](agent-upgrade.md#version-support-policy) covers the most recent version and the two previous versions (N-2).

## Version 1.25.0 (April 2025)

- Fixes to MSI-Adapter response to improve retrieval of JWT tokens by extensions and upstream libraries
- Make agent update state mutually exclusive to avoid conflicts
- Upgrade connected proxy agent
- Security vulnerability fixes

## Version 1.24.4 (March 2025)

- Security vulnerability fixes
- Various bug fixes
- Support for [version-managed extensions](managed-extensions.md)

## Version 1.23.3 (February 2025)

- Updates to improve MSI adapter support
- Updates to improve disconnected scenario
- Security vulnerability fixes
- Various bug fixes

## Older agent releases

## Version 1.22.4 (January 2025)

- Fixed issue with resource health reporting
- Allow special characters in password for proxy server
- Security vulnerability fixes


### Version 1.21.10 (November 2024)

- Container images and charts are now signed
- Security vulnerability fixes

### Version 1.20.10 (September 2024)

- Functionality enhancements
- Security vulnerability fixes

### Version 1.19.x (September 2024)

- Various enhancements and bug fixes

### Version 1.18.x (July 2024)

- Fixed `logCollector` pod restarts
- Updated to Microsoft Go v1.22.5
- Other bug fixes

### Version 1.17.x (June 2024)

- Upgraded to use [Microsoft Go 1.22 to be FIPS compliant](https://github.com/microsoft/go/blob/microsoft/main/eng/doc/fips/README.md#tls-with-fips-compliant-settings)

### Version 1.16.x (May 2024)

- Migrated to use Microsoft Go w/ OpenSSL and fixed some vulnerabilities

### Version 1.15.3 (March 2024)

- Various enhancements and bug fixes

### Version 1.14.5 (December 2023)

- Migrated automatic upgrade to use latest Helm release

### Version 1.13.4 (October 2023)

- Various enhancements and bug fixes

### Version 1.13.1 (September 2023)

- Various enhancements and bug fixes

### Version 1.12.5 (July 2023)

- Alpine base image powering our Arc agent containers has been updated from 3.7.12 to 3.18.0

### Version 1.11.7 (May 2023)

- Updates to enable users that belong to more than 200 groups in cluster connect scenarios

### Version 1.11.3 (April 2023)

- Updates to base image of Arc-enabled Kubernetes agents to address security CVE

## Next steps

- Learn how to [enable or disable automatic agent upgrades](agent-upgrade.md).
- Learn how to [connect a Kubernetes cluster to Azure Arc](quickstart-connect-cluster.md).
