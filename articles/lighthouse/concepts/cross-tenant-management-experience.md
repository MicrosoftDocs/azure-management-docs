---
title: Cross-tenant management experiences
description: Azure Lighthouse enables and enhances cross-tenant experiences in many Azure services.
ms.date: 06/18/2024
ms.topic: concept-article
---

# Cross-tenant management experiences

As a service provider, you can use [Azure Lighthouse](../overview.md) to manage your customers' Azure resources from within your own Microsoft Entra tenant. Many common tasks and services can be performed across these managed tenants.

> [!TIP]
> Azure Lighthouse can also be used [within an enterprise which has multiple Microsoft Entra tenants of its own](enterprise.md) to simplify cross-tenant administration.

## Understanding tenants and delegation

A Microsoft Entra tenant is a representation of an organization. It's a dedicated instance of Microsoft Entra ID that an organization receives when they create a relationship with Microsoft by signing up for Azure, Microsoft 365, or other services. Each Microsoft Entra tenant is distinct and separate from other Microsoft Entra tenants, and has its own tenant ID (a GUID). For more information, see [What is Microsoft Entra ID?](/entra/fundamentals/whatis)

Typically, in order to manage Azure resources for a customer, service providers must sign in to the Azure portal using an account associated with that customer's tenant. In this scenario, an administrator in the customer's tenant must create and manage user accounts for the service provider.

With Azure Lighthouse, the onboarding process specifies users in the service provider's tenant who are assigned roles to delegated subscriptions and resource groups in the customer's tenant. These users can then sign in to the Azure portal, using their own credentials, and work on resources belonging to all of the customers to which they have access. Users in the managing tenant can see all of these customers by visiting the [My customers](../how-to/view-manage-customers.md) page in the Azure portal. They can also work on resources directly within the context of that customer's subscription, either in the Azure portal or via APIs.

Azure Lighthouse provides flexibility to manage resources for multiple customers without having to sign in to different accounts in different tenants. For example, a service provider may have two customers with different responsibilities and access levels. Using Azure Lighthouse, authorized users can sign in to the service provider's tenant and access all of the delegated resources across these customers, according to the [roles they've been assigned](tenants-users-roles.md) for each delegation.

![Diagram showing resources for two customers managed through one service provider tenant.](../media/azure-delegated-resource-management-service-provider-tenant.jpg)

## APIs and management tool support

You can perform management tasks on delegated resources in the Azure portal, or you can use APIs and management tools such as Azure CLI and Azure PowerShell. All existing APIs can be used on delegated resources, as long as the functionality is supported for cross-tenant management and the user has the appropriate permissions.

The Azure PowerShell [Get-AzSubscription cmdlet](/powershell/module/Az.Accounts/Get-AzSubscription) shows the `TenantId` for the managing tenant by default. The `HomeTenantId` and `ManagedByTenantIds` attributes for each subscription allow you to identify whether a returned subscription belongs to a managed tenant or to your managing tenant.

Similarly, Azure CLI commands such as [az account list](/cli/azure/account#az-account-list) show the `homeTenantId` and `managedByTenants` attributes. If you don't see these values when using Azure CLI, try clearing your cache by running `az account clear` followed by `az login --identity`.

In the Azure REST API, the [Subscriptions - Get](/rest/api/resources/subscriptions/get) and [Subscriptions - List](/rest/api/resources/subscriptions/list) commands include `ManagedByTenant`.

> [!NOTE]
> In addition to tenant information related to Azure Lighthouse, tenants shown by these APIs may also reflect partner tenants for Azure Databricks or Azure managed applications.

We also provide APIs that are specific to performing Azure Lighthouse tasks. For more info, see the **Reference** section.

## Enhanced services and scenarios

Most Azure tasks and services can be used with delegated resources across managed tenants, assuming the appropriate roles are granted. Below are some of the key scenarios where cross-tenant management can be especially effective.

[Azure Arc](../../azure-arc/index.yml):

- Manage hybrid servers at scale - [Azure Arc-enabled servers](../../azure-arc/servers/overview.md):
  - [Onboard servers](../../azure-arc/servers/learn/quick-enable-hybrid-vm.md) to delegated customer subscriptions and/or resource groups in Azure
  - Manage Windows Server or Linux machines outside Azure that are connected to delegated subscriptions
  - Manage connected machines using Azure constructs, such as Azure Policy and tagging
  - Ensure the same set of [policies are applied](../../azure-arc/servers/learn/tutorial-assign-policy-portal.md) across customers' hybrid environments
  - Use Microsoft Defender for Cloud to [monitor compliance across customers' hybrid environments](/azure/defender-for-cloud/quickstart-onboard-machines?pivots=azure-arc)
- Manage hybrid Kubernetes clusters at scale - [Azure Arc-enabled Kubernetes](../../azure-arc/kubernetes/overview.md):
  - [Connect Kubernetes clusters](../../azure-arc/kubernetes/quickstart-connect-cluster.md) to delegated subscriptions and/or resource groups
  - [Use GitOps](../../azure-arc/kubernetes/tutorial-use-gitops-flux2.md) to deploy configurations to connected clusters
  - Perform management tasks such as [enforcing policies across connected clusters](/azure/governance/policy/concepts/policy-for-kubernetes#install-azure-policy-extension-for-azure-arc-enabled-kubernetes)

[Azure Automation](/azure/automation/):

- Use Automation accounts to access and work with delegated resources

[Azure Backup](/azure/backup/):

- Back up and restore customer data using Azure Backup. Currently, the following Azure workloads are supported: Azure Virtual Machines (Azure VM), Azure Files, SQL Server on Azure VMs, SAP HANA on Azure VMs. Workloads which leverage [Backup vault](/azure/backup/backup-vault-overview) (such as Azure Database for PostgreSQL, Azure Blob, Azure Managed Disk, and Azure Kubernetes Services) currently aren't fully supported.
- View data for all delegated customer resources in [Backup center](/azure/backup/backup-center-overview)
- Use the [Backup Explorer](/azure/backup/monitor-azure-backup-with-backup-explorer) to help view operational information of backup items (including Azure resources not yet configured for backup) and monitoring information (jobs and alerts) for delegated subscriptions. The Backup Explorer is currently available only for Azure VM data.
- Use [Backup reports](/azure/backup/configure-reports) across delegated subscriptions to track historical trends, analyze backup storage consumption, and audit backups and restores.

[Azure Blueprints](/azure/governance/blueprints/overview):

- Use Azure Blueprints to orchestrate the deployment of resource templates and other artifacts (requires [additional access](https://www.wesleyhaakman.org/preparing-azure-lighthouse-customer-subscriptions-for-azure-blueprints/) to prepare the customer subscription)

[Azure Cost Management + Billing](/azure/cost-management-billing/):

- From the managing tenant, CSP partners can view, manage, and analyze pre-tax consumption costs (not inclusive of purchases) for customers who are under the Azure plan. The cost is based on retail rates and the Azure role-based access control (Azure RBAC) access that the partner has for the customer's subscription. Currently, you can view consumption costs at retail rates for each individual customer subscription based on Azure RBAC access.

[Azure Key Vault](/azure/key-vault/general/):

- Create Key Vaults in customer tenants
- Use a managed identity to create Key Vaults in customer tenants

[Azure Kubernetes Service (AKS)](/azure/aks/):

- Manage hosted Kubernetes environments and deploy and manage containerized applications within customer tenants
- Deploy and manage clusters in customer tenants
- [Use Azure Monitor for containers](/azure/aks/monitor-aks) to monitor performance across customer tenants

[Azure Migrate](/azure/migrate/):

- Create migration projects in the customer tenant and migrate VMs

[Azure Monitor](/azure/azure-monitor/):

- View alerts for delegated subscriptions, with the ability to view and refresh alerts across all subscriptions
- View activity log details for delegated subscriptions
- [Log analytics](/azure/azure-monitor/logs/workspace-design#multiple-tenant-strategies): Query data from remote workspaces in multiple tenants (note that automation accounts used to access data from workspaces in customer tenants must be created in the same tenant)
- Create, view, and manage [alerts](/azure/azure-monitor/alerts/alerts-create-new-alert-rule) in customer tenants
- Create alerts in customer tenants that trigger automation, such as Azure Automation runbooks or Azure Functions, in the managing tenant through webhooks
- Create [diagnostic settings](/azure/azure-monitor/essentials/diagnostic-settings) in workspaces created in customer tenants, to send resource logs to workspaces in the managing tenant
- For SAP workloads, [monitor SAP Solutions metrics with an aggregated view across customer tenants](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/using-azure-lighthouse-and-azure-monitor-for-sap-solutions-to/ba-p/1537293)
- For Azure AD B2C, [route sign-in and auditing logs](/azure/active-directory-b2c/azure-monitor) to different monitoring solutions

[Azure Networking](/azure/networking/fundamentals/networking-overview):

- Deploy and manage [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) and virtual network interface cards (vNICs) within managed tenants
- Deploy and configure [Azure Firewall](/azure/firewall/overview) to protect customers’ Virtual Network resources
- Manage connectivity services such as [Azure Virtual WAN](/azure/virtual-wan/virtual-wan-about), [Azure ExpressRoute](/azure/expressroute/expressroute-introduction), and [VPN Gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways)
- Use Azure Lighthouse to support key scenarios for the [Azure Networking MSP Program](/azure/networking/networking-partners-msp)

[Azure Policy](/azure/governance/policy/):

- Create and edit policy definitions within delegated subscriptions
- Deploy policy definitions and policy assignments across multiple tenants
- Assign customer-defined policy definitions within delegated subscriptions
- Customers see policies authored by the service provider alongside any policies they've authored themselves
- Can [remediate deployIfNotExists or modify assignments within the managed tenant](../how-to/deploy-policy-remediation.md)
- Note that viewing compliance details for non-compliant resources in customer tenants is not currently supported

[Azure Resource Graph](/azure/governance/resource-graph/):

- See the tenant ID in returned query results, allowing you to identify whether a subscription belongs to a managed tenant

[Azure Service Health](/azure/service-health/):

- Monitor the health of customer resources with Azure Resource Health
- Track the health of the Azure services used by your customers

[Azure Site Recovery](/azure/site-recovery/):

- Manage disaster recovery options for Azure virtual machines in customer tenants (note that you can't use `RunAs` accounts to copy VM extensions)

[Azure Virtual Machines](/azure/virtual-machines/):

- Use virtual machine extensions to provide post-deployment configuration and automation tasks on Azure VMs
- Use boot diagnostics to troubleshoot Azure VMs
- Access VMs with serial console
- Integrate VMs with Azure Key Vault for passwords, secrets, or cryptographic keys for disk encryption by using [managed identity through policy](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/create-keyvault-secret), ensuring that secrets are stored in a Key Vault in the managed tenants
- Note that you can't use Microsoft Entra ID for remote login to VMs

[Microsoft Defender for Cloud](/azure/defender-for-cloud/):

- Cross-tenant visibility
  - Monitor compliance with security policies and ensure security coverage across all tenants' resources
  - Continuous regulatory compliance monitoring across multiple tenants in a single view
  - Monitor, triage, and prioritize actionable security recommendations with secure score calculation
- Cross-tenant security posture management
  - Manage security policies
  - Take action on resources that are out of compliance with actionable security recommendations
  - Collect and store security-related data
- Cross-tenant threat detection and protection
  - Detect threats across tenants' resources
  - Apply advanced threat protection controls such as just-in-time (JIT) VM access
  - Harden network security group configuration with Adaptive Network Hardening
  - Ensure servers are running only the applications and processes they should be with adaptive application controls
  - Monitor changes to important files and registry entries with File Integrity Monitoring (FIM)
- Note that the entire subscription must be delegated to the managing tenant; Microsoft Defender for Cloud scenarios are not supported with delegated resource groups

[Microsoft Sentinel](/azure/sentinel/multiple-tenants-service-providers):

- Manage Microsoft Sentinel resources [in customer tenants](/azure/sentinel/multiple-tenants-service-providers)
- [Track attacks and view security alerts across multiple tenants](https://techcommunity.microsoft.com/t5/azure-sentinel/using-azure-lighthouse-and-azure-sentinel-to-monitor-across/ba-p/1043899)
- [View incidents](/azure/sentinel/multiple-workspace-view) across multiple Microsoft Sentinel workspaces spread across tenants

Support requests:

- [Open support requests from **Help + support**](../../azure-portal/supportability/how-to-create-azure-support-request.md#getting-started) in the Azure portal for delegated resources (selecting the support plan available to the delegated scope)
- Use the [Azure Quota API](/rest/api/reserved-vm-instances/quotaapi) to view and manage Azure service quotas for delegated customer resources

## Current limitations

With all scenarios, be aware of the following current limitations:

- Requests handled by Azure Resource Manager can be performed using Azure Lighthouse. The operation URIs for these requests start with `https://management.azure.com`. However, requests that are handled by an instance of a resource type (such as Key Vault secrets access or storage data access) aren't supported with Azure Lighthouse. The operation URIs for these requests typically start with an address that is unique to your instance, such as `https://myaccount.blob.core.windows.net` or `https://mykeyvault.vault.azure.net/`. The latter are also typically data operations rather than management operations.
- Role assignments must use [Azure built-in roles](/azure/role-based-access-control/built-in-roles). All built-in roles are currently supported with Azure Lighthouse, except for Owner or any built-in roles with [`DataActions`](/azure/role-based-access-control/role-definitions#dataactions) permission. The User Access Administrator role is supported only for limited use in [assigning roles to managed identities](../how-to/deploy-policy-remediation.md#create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant).  Custom roles and [classic subscription administrator roles](/azure/role-based-access-control/classic-administrators) are not supported. For more information, see [Role support for Azure Lighthouse](tenants-users-roles.md#role-support-for-azure-lighthouse).
- For users in the managed tenant, role assignments made through Azure Lighthouse aren't shown under Access Control (IAM) or with CLI tools such as `az role assignment list`. These assignments are only visible in the Azure portal in the **Delegations** section of Azure Lighthouse, or through the Azure Lighthouse API.
- While you can onboard subscriptions that use Azure Databricks, users in the managing tenant can't launch Azure Databricks workspaces on a delegated subscription.
- While you can onboard subscriptions and resource groups that have resource locks, those locks won't prevent actions from being performed by users in the managing tenant. [Deny assignments](/azure/role-based-access-control/deny-assignments) that protect system-managed resources (system-assigned deny assignments), such as those created by Azure managed applications or Azure Blueprints, do prevent users in the managing tenant from acting on those resources. However, users in the customer tenant can't create their own deny assignments.
- Delegation of subscriptions across a [national cloud](/azure/active-directory/develop/authentication-national-cloud) and the Azure public cloud, or across two separate national clouds, is not supported.

## Next steps

- Onboard your customers to Azure Lighthouse, either by [using Azure Resource Manager templates](../how-to/onboard-customer.md) or by [publishing a private or public managed services offer to Azure Marketplace](../how-to/publish-managed-services-offers.md).
- [View and manage customers](../how-to/view-manage-customers.md) by going to **My customers** in the Azure portal.
- Learn more about [Azure Lighthouse architecture](architecture.md). 
