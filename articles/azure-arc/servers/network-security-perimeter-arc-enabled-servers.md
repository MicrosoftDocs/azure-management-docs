---
title: Configure Network Security Perimeter for Azure Arc-enabled servers
description: Learn about a proposed workflow to control public access to the Azure Arc-enabled servers data plane by using Network Security Perimeter.
ms.topic: how-to
ms.date: 11/10/2026
---

# Configure network security perimeter for Azure Arc-enabled servers

> [!IMPORTANT]
> To evaluate Network Security Perimeter for Arc-enabled Servers, use the following link: [aka.ms/arc-servers-nsp](https://aka.ms/arc-servers-nsp)

Azure Network Security Perimeter (NSP) establishes a logical network boundary around supported Azure platform-as-a-service (PaaS) resources. It controls public network access through explicit access rules and provides access logs for auditing and troubleshooting.

This article describes the proposed configuration for Azure Arc-enabled servers: create a perimeter and profile first, create an Azure Arc private link scope, define inbound access rules, associate the scope with the profile, and then associate servers with the scope.

> [!NOTE]
> In this article, **inbound** means traffic arriving at the **Azure Arc service data plane** from an agent or another client. It doesn't mean inbound connections to the operating system of an Arc-enabled server. NSP isn't a replacement for a host firewall, a network firewall, or server remote-access controls.

## Understand the configuration

| Component | Purpose |
| --- | --- |
| Azure Arc private link scope | Groups the Arc-enabled servers that use the scope. The source design uses this resource as the NSP association target. |
| Network security perimeter | Defines the logical boundary for supported PaaS resources. |
| NSP profile | Holds the access rules applied to associated resources. |
| Resource association | Links a resource to an NSP profile and specifies its access mode. |
| Inbound IP access rule | Allows requests to the protected public service endpoint from specified source IP addresses or CIDR prefixes. |
| Diagnostic setting | Routes supported NSP access logs to a configured destination. |

For Azure Arc-enabled servers, enforced mode denies public access unless an inbound IP prefix access rule permits the request. Perimeter membership doesn't automatically grant access, and permitted requests still require authentication and authorization.

Private endpoint traffic isn't restricted by NSP public access rules. Creating a private link scope doesn't itself create a private endpoint or configure routing and DNS.

For more information, see [Network Security Perimeter concepts](/azure/private-link/network-security-perimeter-concepts.md).

## Limitations

Azure Arc-enabled servers supports only **inbound IP prefix access rules** for Network Security Perimeter. The following limitations apply:

- **No intra-perimeter access:** Associating Azure Arc with a perimeter doesn't automatically allow access from other resources in the same perimeter.
- **No outbound access rules:** Arc's NSP integration doesn't support outbound rules or control outbound access from Arc-enabled servers.
- **No other inbound rule types:** Configure inbound access using source IP addresses or CIDR prefixes only.

Inbound rules apply to requests arriving at the Azure Arc service data plane, not to inbound connections to the server's operating system. Use the public egress IP addresses that Azure Arc sees, including any proxy, firewall, or NAT addresses.

## Prerequisites

Before using the proposed Arc-specific workflow, confirm that you have:

- Product team approval for Arc NSP support, including availability, supported regions and clouds, feature registration, minimum agent version, and any limitations.
- An Azure subscription and permissions to create or update the Arc private link scope, perimeter, profile, access rules, and resource association. Confirm the exact Arc NSP role requirements before publication.
- A new, unused Azure Arc private link scope for initial evaluation. Don't apply this draft to a scope that already serves production machines.
- The public egress IP addresses or CIDR prefixes that your servers use to reach Azure Arc. If traffic passes through a proxy, firewall, or NAT device, identify the source address that's visible to the Azure service.
- Existing connectivity to Azure Arc dependencies. An NSP rule doesn't configure a server's outbound firewall, DNS, proxy, or access to Microsoft Entra ID and Azure Resource Manager.
- Azure CLI version **2.75.0 or later**, with the `connectedmachine` and `nsp` extensions, for the current CLI examples.

If you also use private endpoints, complete the separate networking and DNS requirements in [Use Azure Private Link with Azure Arc-enabled servers](private-link-security.md). Don't assume that the private endpoint prerequisites apply unchanged to the proposed public-endpoint NSP scenario.

## Configuration sequence

Use the following sequence for a new scope:

1. Create a network security perimeter and profile.
2. Create an Azure Arc private link scope.
3. Define the required inbound access rules.
4. Associate the private link scope with the profile and configure enforcement.
5. Associate servers with the private link scope.

Create the **perimeter and profile first**, before creating the private link scope. 

## Configure by using the Azure portal

The following portal workflow incorporates the requested perimeter-first sequence. Confirm the Arc-specific portal experience before publication.

### 1. Create a perimeter and profile

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Search for **Network security perimeters**.
3. Select **Create**.
4. Enter the subscription, resource group, perimeter name, and location.
5. Define inbound IP prefix access rules for the public egress IP addresses or CIDR prefixes that your servers use to reach Azure Arc.
6. Complete the creation workflow and create or select the profile that holds your Arc access rules.

### 2. Create an Azure Arc private link scope

1. Search for **Azure Arc Private Link Scopes**.
2. Select **Create**.
3. On the **Basics** tab, select the subscription and resource group.
4. On the **Networking** tab, select **Secure by Perimeter** for the Public Network Access setting.
5. Select your NSP's subscription and resource group, and then select your NSP and related profile.
6. Select **Review + create**, wait for validation, and select **Create**.

Don't associate servers yet. If your deployment also requires private connectivity, create and configure its private endpoint and DNS separately.

### 3. Associate servers with the scope

#### Existing Arc-enabled servers

1. Open the Azure Arc private link scope.
2. Select **Azure Arc resources**, and then select **+ Add**.
3. Select the servers to associate.
4. Select **Select** to save the association.

#### New servers

1. Open **Machines - Azure Arc** and select **Add/Create** > **Add a machine**.
2. Choose **Add a single server** or **Add multiple servers**, and select **Generate script**.
3. Select the server subscription, resource group, region, and operating system.
4. Select the appropriate connectivity method and the private link scope.
5. For multiple servers, select an appropriate service principal. See [Connect machines by using a service principal](onboard-service-principal.md).
6. Add the required tags, review the summary, and download the onboarding script.
7. Run the script with administrator or root privileges and review the onboarding result.

## Enable and review access logs

NSP supports diagnostic access logs for allowed and denied requests. Logging requires explicit configuration; creating the perimeter alone doesn't establish a log destination.

Configure diagnostic settings on the perimeter and choose the supported log categories and destination described in [Diagnostic logs for Network Security Perimeter](/azure/private-link/network-security-perimeter-diagnostic-logs).

General NSP categories include `NspPublicInboundPerimeterRulesAllowed` and `NspPublicInboundPerimeterRulesDenied`. Confirm which categories Arc emits before promising Arc-specific logging coverage. Use the resulting logs to investigate missing prefixes, unexpected sources, and the effect of enforcement.

## Troubleshoot

| Symptom | What to check |
| --- | --- |
| Arc private link scope isn't available for association | Confirm Arc resource-type support, regional availability, and any required enablement. Don't infer support from the generic NSP CLI. |
| Agent loses connectivity after enforcement | Verify the actual public egress address, inbound rules, scope association, propagation, and required outbound access to other Azure dependencies. |
| Server doesn't appear in the scope picker | Check subscription and region against the source constraints; confirm applicable Arc NSP requirements. |
| Portal onboarding offers only Private endpoint for scope selection | Confirm the supported onboarding method for the intended public-endpoint NSP topology. |
| No access logs appear | Check diagnostic settings, selected categories, destination configuration, and Arc log support. |
| A private endpoint request isn't denied by a public NSP rule | Private endpoint traffic isn't restricted by public NSP rules. Verify the actual network path. |
