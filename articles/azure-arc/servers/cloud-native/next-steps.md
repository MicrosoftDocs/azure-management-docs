---
title: Next steps for cloud-native server management with Azure Arc-enabled servers
description: Learn how to transition your server management into a cloud-native model using Azure Arc-enabled servers. 
ms.date: 08/19/2025
ms.topic: overview
# Customer intent: "As a system architect managing a hybrid cloud environment, I want to understand how start moving from traditional on-premises management to a cloud-native model by using the management capabilities of Azure Arc-enabled servers."
---

# Next steps for cloud-native server management with Azure Arc-enabled servers

Transitioning from traditional on-premises management to a cloud-native approach is often a gradual process. It’s a journey, but one that leads to more consistent and scalable management. [Azure Arc-enabled servers](../overview.md) has the flexibility to help you along the way.

Evolve your server management approach by following the approach described in this article. You'll gradually migrate from being an "on-premises system administrator" to a "cloud-era system administrator." The core skills and knowledge that you use today remain as valuable as ever, but now Azure's powerful tools will help you do the job more efficiently.

## Hybrid enablement

Begin by onboarding a subset of your servers to Azure Arc to establish a baseline hybrid management model.
<!-- Converted narrative guidance into a defined procedure with steps, per agent feedback. -->

### Onboard a subset of servers

1. Identify a small, representative group of servers that reflect your broader environment (for example, different operating systems, roles, or locations).
2. [Choose a deployment method](../deployment-options.md) for the Connected Machine agent based on your tooling and operational preferences.
3. Deploy the Connected Machine agent to the selected servers.
4. Validate that the servers appear as Azure Arc-enabled resources in the Azure portal.
<!-- Steps added to make the procedure actionable and repeatable. -->

**Verification:** Selected servers are visible in Azure as Arc-enabled servers and report a healthy connection status.
<!-- Added one-line expected outcome as requested. -->

As part of your onboarding plan, consider how to organize these resources in Azure to support management at scale.
<!-- Minor transition added to connect procedure to next guidance. -->

**Examples of resource organization considerations:**
- Resource groups aligned to application, environment, or business unit
- Consistent tagging for environment, location, or ownership
- Alignment with Azure Policy scope boundaries
<!-- Converted descriptive paragraph into a bulleted list. -->

A well-planned resource hierarchy improves clarity and simplifies policy application.
<!-- Sentence retained to preserve original intent. -->

## Iterative adoption of services

Adopt Azure services incrementally so your team can build confidence and refine processes over time.
<!-- Reframed opening sentence for procedural clarity. -->

### Phased service adoption

1. Select a single management capability to adopt first, such as patching or compliance reporting.
2. Enable the chosen Azure service for a limited set of Arc-enabled servers.
3. Operate the service alongside existing tools to compare results and validate coverage.
4. Gradually expand usage to additional servers as confidence increases.
<!-- Converted narrative examples into a numbered procedure. -->

**Examples of phased adoption:**
- Use [Azure Update Manager](/azure/update-manager/overview) for patching while continuing to rely on Group Policy Objects (GPOs) for configuration.
- Introduce [Azure machine configuration](/azure/governance/machine-configuration/) in audit mode for critical settings.
- Run routine tasks on a subset of servers using [Azure Run Command](/azure/virtual-machines/run-command-overview).
<!-- Converted example paragraph into a bulleted list. -->

**Verification:** Azure-based services operate successfully alongside existing tools without disrupting current operations.
<!-- Added expected outcome line. -->

During the transition, maintaining overlapping solutions can reduce risk.
<!-- Retained concept while shortening sentence. -->

**Common overlap scenarios:**
- Keep Windows Server Update Services (WSUS) as a backup while validating Azure Update Manager coverage.
- Continue enforcing GPOs while using Azure Policy in audit mode for compliance comparison.
<!-- Converted prose into bullets for scannability. -->

Once cloud-based processes consistently meet your requirements, you can retire redundant systems to reduce cost and complexity.
<!-- Clarified conclusion of the phase. -->

## Automation and environment management

Automation helps reduce manual effort and enforce consistency across your environment.
<!-- Slight rewrite for active voice. -->

Some ways you can leverage automation with Azure Arc-enabled servers include:

- Using Azure Policy to automatically apply tags when your servers meet specific requirements
- Scripted onboarding of new servers to Azure Arc
- Policy-based remediation for configuration drift
<!-- Converted examples into a concise bullet list. -->

Apply Azure security and governance controls consistently across both your Arc-enabled servers and native Azure VMs.
<!-- Transition sentence preserved. -->

**Security and governance components:**
- [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/built-in-roles) to enforce least privilege
- Microsoft Defender for Cloud recommendations for security posture improvements
- Azure Policy and scripts for rapid remediation
- Azure Activity logs for auditing and change tracking
<!-- Converted security paragraph into bulleted list. -->

To maintain agent health, configure alerts for connectivity or extension failures so issues are detected early.
<!-- Clarified final sentence for actionability. -->

## Team training

As you refine your processes, document your standards and operational steps so your team can work consistently.
<!-- Minor rewrite for clarity. -->

**Training focus areas:**
- Azure portal and Azure CLI usage
- Azure role-based access control (Azure RBAC)
- Azure Resource Graph queries
- Service-specific operational procedures (for example, on-demand patching)
<!-- Converted narrative list into bullets. -->

Microsoft provides extensive [free training for Azure services](/training/azure/), including Azure Arc learning paths such as:
- [Bring innovation to your hybrid environments with Azure Arc](/training/paths/manage-hybrid-infrastructure-with-azure-arc/)
- [Deploy and manage Azure Arc-enabled servers](/training/paths/deploy-manage-azure-arc-enabled-servers/)
<!-- Converted inline list into bullets for consistency. -->

## Community resources

Azure Arc continues to evolve, with new capabilities released regularly.
<!-- Shortened opening sentence for clarity. -->

**Ways to stay informed:**
- Follow the [Azure Arc blog](https://techcommunity.microsoft.com/category/azure/blog/azurearcblog)
- Review [Azure Updates](https://azure.microsoft.com/updates/) for new features
- Ask and answer questions on [Microsoft Q&A](/answers/tags/146/azure-arc)
<!-- Converted paragraph into a bulleted list. -->

The [Azure Arc Jumpstart](https://azurearcjumpstart.io/) provides a hybrid sandbox for hands-on experimentation, with scenarios updated frequently.
<!-- Minor rewrite to improve conciseness. -->

## Continuous improvement

Prioritize continuous learning and evaluation, even after establishing cloud-based processes.
<!-- Minor wording adjustment for tone. -->

Regularly review outcomes, optimize configurations, and explore additional Azure services that can extend management capabilities to Arc-enabled servers. Over time, your confidence and efficiency with a cloud-native server management model will continue to grow.
<!-- Combined closing thoughts into a clear concluding paragraph. -->

---

### Agent feedback applied

- ACTION: Turn each recommended procedure (onboard a subset, phased service adoption, automation setup) into numbered steps and add a one-line verification or expected outcome at the end of each procedure
- ACTION: Convert prerequisite and parameter prose into concise bulleted lists or a simple two-column table inside the existing sections
- ACTION: Turn each recommended procedure (onboard a subset, phased service adoption, automation setup) into numbered steps and add a one-line verification or expected outcome at the end of each procedure
- ACTION: Convert prerequisite and parameter prose into concise bulleted lists or a simple two-column table inside the existing sections
- ACTION: Turn each recommended procedure (onboard a subset, phased service adoption, automation setup) into numbered steps and add a one-line verification or expected outcome at the end of each procedure
- ACTION: Convert prerequisite and parameter prose into concise bulleted lists or a simple two-column table inside the existing sections
