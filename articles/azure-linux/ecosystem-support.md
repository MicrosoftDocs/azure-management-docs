---
title: Azure Linux Partner Solutions
description: Learn about Azure Linux partner solutions for build, test, deployment, configuration, and monitoring of your applications running on Azure Linux.
ms.author: schaffererin
author: kavyamsft
ms.service: microsoft-linux
ms.topic: concept-article
ms.custom: linux-related-content
ms.date: 05/13/2026
---

# Azure Linux partner solutions

Microsoft collaborates with partners to ensure your build, test, deployment, configuration, and monitoring of your applications perform optimally with Azure Linux.

The third party partners featured in this article have introduction guides to help you start using their solutions with your applications running on Azure Linux.

## Validated solutions

The following partners have validated their solutions on Azure Linux. A partner not being listed doesn't necessarily mean their solution is incompatible. If you're a partner interested in validating your solution on Azure Linux please reach out to: AzureLinuxPartners@microsoft.com.

| Partner | Category | Azure Linux Container Host | Azure Container Linux (ACL) | Azure Linux VM and VMSS |
| ------- | -------- | -------------------------- | -------------------------- | ------------------------ |
| [Advantech](#advantech) | DevOps | Validated ✅ | | |
| [Akuity](#akuity) | DevOps | Validated ✅ | | |
| [Anchore](#anchore) | DevOps, Security | Validated ✅ | | |
| [Buoyant](#buoyant) | Networking, Observability, Security | Validated ✅ | | |
| [Catalogic](#catalogic) | Storage, Migration | Validated ✅ | | |
| [Corent](#corent) | Config Management | Validated ✅ | | |
| [CrowdStrike](#crowdstrike) | Security | Validated ✅ | | |
| [Dynatrace](#dynatrace) | Observability | Validated ✅ | | |
| [Elastic](#elastic) | Observability, Security | Validated ✅ | Validated ✅ | |
| [Hashicorp](#hashicorp) | DevOps | Validated ✅ | | |
| [Isovalent](#isovalent) | Networking, Observability, Security | Validated ✅ | | |
| [Kong](#kong) | DevOps, Security | Validated ✅ | | |
| [Kubecost](#kubecost) | DevOps, Observability | Validated ✅ | | |
| [NetApp](#netapp) | DevOps | Validated ✅ | | |
| [New Relic](#new-relic) | Observability | Validated ✅ | | |
| [Palo Alto Networks](#palo-alto-networks) | Security | Validated ✅ | | |
| [Qualys](#qualys) | Security | Validated ✅ | | |
| [Solo.io](#soloio) | Networking, Observability, Security | Validated ✅ | | |
| [Sysdig](#sysdig) | DevOps, Security, Config Management | Validated ✅ | | |
| [Tetrate](#tetrate) | Networking, Security | Validated ✅ | | |
| [Tigera](#tigera-inc) | Networking, Observability, Security | Validated ✅ | | |
| [TrendAI](#trendai) | Security | Validated ✅ | | |
| [Upwind](#upwind) | Security | Validated ✅ | | |
| [Veeam](#veeam) | Storage | Validated ✅ | | |
| [VictoriaMetrics](#victoriametrics) | DevOps, Observability, Storage, Monitoring | Validated ✅ | | |
| [Wiz](#wiz) | Security | Validated ✅ | | |

> [!NOTE]
> Azure Linux 4.0 is now in **Public Preview** and is strictly limited to evaluation and testing purposes. It is not suitable for production use.

## DevOps

DevOps streamlines the delivery process, improves collaboration across teams, and enhances software quality, ensuring swift, reliable, and continuous deployment of your applications.

### Advantech

#### iFactoryEHS

:::image type="icon" source="./media/ecosystem-support/advantech.png":::

| Solution | Categories |
|----------|------------|
| iFactoryEHS | DevOps |

The right EHS management system can strengthen organizations behind the scenes and enable them to continuously put their best foot forward. iFactoryEHS solution is designed to help manufacturers manage employee health, improve safety, and analyze environmental footprints while ensuring operational continuity.

For more information, see [Advantech & iFactoryEHS](https://page.advantech.com/en/global/solutions/ifactory/ifactory_ehs).

#### FactoryOEE

:::image type="icon" source="./media/ecosystem-support/factoryoee.png":::

| Solution | Categories |
|----------|------------|
| FactoryOEE | DevOps |

Advantech FactoryOEE is an innovative factory management tool that enhances production efficiency and focuses on energy consumption and carbon emissions.

<details> <summary> See more </summary><br>

Through advanced data analytics, it monitors energy usage in real-time, reducing carbon footprint. We're committed to creating an efficient, eco-friendly production model, optimizing your factory for peak performance while minimizing environmental impact. Choose FactoryOEE for the perfect balance of efficiency and sustainability.

</details>

For more information, see [Advantech & FactoryOEE solutions](https://wise-paas.advantech.com/marketplace/product/advantech.FactoryOEE) and [Advantech on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/advantech.ifactory_tpm?tab=Overview).

### Akuity

:::image type="icon" source="./media/ecosystem-support/akuity.png":::

| Solution | Categories |
|----------|------------|
| Akuity Platform | DevOps |

The Akuity Platform is a managed solution for Argo CD from the creators of Argo open source project.

<details> <summary> See more </summary><br>

Argo Project is a suite of open source tools for deploying and running applications and workloads on Kubernetes. It extends the Kubernetes APIs and unlocks new and powerful capabilities in application deployment, container orchestration, event automation, progressive delivery, and more.

Akuity is rooted in Argo, extending its capabilities and using the same familiar user interface. The platform solves real-life DevOps use cases using battle-tested patterns packaged into a product with the best possible developer experience.

</details>

For more information, see [Akuity Solutions](https://akuity.io/).

### Anchore

:::image type="icon" source="./media/ecosystem-support/anchore.png":::

| Solution | Categories |
|----------|------------|
| Anchore | DevOps <br> Observability <br> Security |

Anchore is a software bill of materials (SBOM) powered software supply chain management solution designed for a cloud-native world.

<details> <summary> See more </summary><br>

It provides continuous visibility into supply chain security risks. Anchore takes a developer-friendly approach that minimizes friction by embedding automation into development toolchains to generate SBOMs and accurately identify vulnerabilities, malware, misconfigurations, and secrets for faster remediation.

We're passionate about protecting software supply chains by making it easier for developers and security teams to deliver secure cloud-native software. Together, we've built a platform and open source tools that help organizations secure the software they build without compromising velocity.

</details>

For more information, see [Anchore solutions](https://anchore.com/).

### Hashicorp

:::image type="icon" source="./media/ecosystem-support/hashicorp.png" border="gray":::

| Solution | Categories |
|----------|------------|
| Terraform | DevOps |

At HashiCorp, we believe infrastructure enables innovation, and we're helping organizations to operate that infrastructure in the cloud.

<details> <summary> See more </summary><br>

Our suite of multicloud infrastructure automation products, built on projects with source code freely available at their core, underpin the most important applications for the largest enterprises in the world. As part of the once-in-a-generation shift to the cloud, organizations of all sizes, from well-known brands to ambitious start-ups, rely on our solutions to provision, secure, connect, and run their business-critical applications so they can deliver essential services, communications tools, and entertainment platforms worldwide.

</details>

For more information, see [Hashicorp solutions](https://hashicorp.com/) and [Hasicorp on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/hashicorp-4665790.terraform-azure-saas?tab=overview).

### Kong

:::image type="icon" source="./media/ecosystem-support/kong.png":::

| Solution | Categories |
|----------|------------|
| Kong Connect | DevOps <br> Security |

Kong Konnect is the unified cloud-native API lifecycle platform to optimize any environment. It reduces operational complexity, promotes federated governance, and provides robust security by seamlessly managing Kong Gateway, Kong Ingress Controller and Kong Mesh with a single management console, delivering API configuration, portal, service catalog, and analytics capabilities.

<details> <summary> See more </summary><br>

A unified Konnect control plane empowers businesses to:

- Define a collection of API Data Plane Nodes that share the same configuration.
- Provide a single control plane to catalog, connect to, and monitor the status of all control planes and instances and manage group configuration.
- Browse APIs, reference documentation, test endpoints, and create applications using specific APIs through a customizable and unified API portal for developers.
- Create a single source of truth by cataloging all services with the Service Hub.
- Access key statistics, monitor vital signs, and spot patterns in real time to see how your APIs and gateways are performing.
- Deliver a fully Kubernetes-centric operational lifecycle model through the integration of DevOps-ready config-driven API management layer and KIC's unrivaled runtime performance.

Kong's extensive ecosystem of community and enterprise plugins delivers critical functionality, including authentication, authorization, rate limiting, request enforcement, and caching, without increasing API platform's footprint.

</details>

For more information, see [Kong Solutions](https://konghq.com/) and [Kong on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/konginc1581527938760.kong-enterprise?tab=Overview).

### Kubecost

:::image type="icon" source="./media/ecosystem-support/kubecost.png":::

| Solution | Categories |
|----------|------------|
| Kubecost Kubernetes Cost Monitoring | DevOps <br> Observability |

Kubecost provides real-time cost visibility and insights for teams using Kubernetes.

<details> <summary> See more </summary><br>

We uncover patterns that create overspending on infrastructure and help teams prioritize where to focus optimization efforts. By identifying root causes for negative patterns, customers using Kubecost save 30-50% or more of their Kubernetes cloud infrastructure costs. Today, Kubecost empowers more than 10,000 teams across companies of all sizes to monitor and reduce costs, while balancing cost, performance, and reliability. Kubecost is tightly integrated with the open-source cloud-native ecosystem and built for engineers and developers first, making it easy to drive adoption within your organization.

</details>

For more information, see [Kubecost Solutions](https://www.kubecost.com/) and [IBM Kubecost on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/ibm-usa-ny-armonk-hq-6275750-ibmcloud-asperia.ibm_kubecost_enterprise?tab=Overview).

### NetApp

:::image type="icon" source="./media/ecosystem-support/spotbynetapp.png":::

| Solution | Categories |
|----------|------------|
| Ocean | DevOps |

Spot Ocean allows organizations to effectively manage their containers' infrastructure at scale, transparently and with minimal effort.

<details> <summary> See more </summary><br>

Ocean ensures cloud-native applications always get continuously optimized infrastructure that's balanced for performance, availability, and cost.

Spot Ocean continuously analyzes how containers use the underlying infrastructure, and automatically scales compute resources to maximize utilization and availability with an optimal blend of spot VMs, reserved instances, savings plans, and pay-as-you-go compute resources.

With Spot Ocean, users gain:

- Automation and multicloud management: Reduce heavy lift infrastructure management efforts and increase operational efficiency
- Cost Optimization: control and significantly reduce infrastructure cost
- Availability: optimize uptime by predicting and automatically addressing resource needs and instance interruptions

</details>

For more information, see [Spot By NetApp Solutions](https://spot.io/product/ocean/).

### Sysdig

:::image type="icon" source="./media/ecosystem-support/sysdig.png":::

| Solution | Categories |
|----------|------------|
| Sysdig Secure | DevOps <br> Security <br> Config Management |

Sysdig is a leader in cloud security powered by runtime insights.

<details> <summary> See more </summary><br>

Built on open source Falco, Sysdig detects threats in real time across cloud environments—helping teams uncover hidden risks and stop attacks. With full-stack visibility into workloads, identities, and services, Sysdig goes beyond traditional scanning to correlate live signals and prioritize what matters most. Whether securing containers, Kubernetes, cloud services, or serverless apps, Sysdig helps teams act with speed and confidence. By reducing alert fatigue, prioritizing vulnerabilities, accelerating response, and consolidating point tools, Sysdig empowers Azure customers to innovate securely at cloud speed. From prevention to defense, Sysdig helps you secure every second.

</details>

For more information, see [Sysdig Solutions](https://sysdig.com/ecosystem/microsoft-azure/) and [Sysdig on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/sysdig.sysdig-enterprise-azure).

## Networking

Ensure efficient traffic management, enhanced security, and optimal network performance.

### Buoyant

:::image type="icon" source="./media/ecosystem-support/buoyant.png":::

| Solution | Categories |
|----------|------------|
| Managed Linkerd with Buoyant Cloud | Networking <br> Security <br> Observability |

Managed Linkerd with Buoyant Cloud automatically keeps your Linkerd control plane data plane up to date with latest versions, and handles installs, trust anchor rotation, and more.

For more information, see [Buoyant Solutions](https://buoyant.io/cloud) and [Buoyant on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/buoyantinc1658330371653.buoyant?tab=Overview).

### Isovalent

:::image type="icon" source="./media/ecosystem-support/isovalent.png":::

| Solution | Categories |
|----------|------------|
| Isovalent Enterprise for Cilium | Networking <br> Security <br> Observability |

Isovalent Enterprise for Cilium provides advanced network policy capabilities, including DNS-aware policy, L7 policy, and deny policy, enabling fine-grained control over network traffic for micro-segmentation and improved security.

<details> <summary> See more </summary><br>

Isovalent also provides multi-cluster connectivity via Cluster Mesh, seamless networking and security across multiple clouds, including public cloud providers like AWS, Azure, and Google Cloud Platform, as well as on-premises environments. With free service-to-service communication and advanced load balancing, Isovalent makes it easy to deploy and manage complex microservices architectures.

The Hubble flow observability + User Interface feature provides real-time network traffic flow and policy visualization, as well as a powerful User Interface for easy troubleshooting and network management. Tetragon provides advanced security capabilities such as protocol enforcement, IP and port allowlists, and automatic application-aware policy generation to protect against the most sophisticated threats. Tetragon is built on eBPF, enabling scaling to meet the needs of the most demanding cloud-native environments with ease.

Isovalent provides enterprise-grade support from their experienced team of experts, ensuring that any issues are resolved in a timely and efficient manner. Additionally, professional services help organizations deploy and manage Cilium in production environments.

</details>

For more information, see [Isovalent Solutions](https://isovalent.com/blog/post/isovalent-azure-linux/) and [Isovalent on Azure Marketplace](https://marketplace.microsoft.com/product/isovalentinc1662143158090.isovalent-cilium-enterprise-offer1?tab=Overview).

### Solo.io

:::image type="icon" source="./media/ecosystem-support/soloio.png":::

| Solution | Categories |
|----------|------------|
| Gloo Mesh Core | Networking <br> Security <br> Observability |

Gloo Mesh Core works with community Istio out of the box. You get instant insights into your Istio environment through a custom dashboard.

<details> <summary> See more </summary><br>

Observability pipelines let you analyze many data sources that you already have. You can even automate installing and upgrading Istio with the Gloo lifecycle manager.

</details>

| Solution | Categories |
|----------|------------|
| Solo distribution of Istio | Networking <br> Security <br> Observability |

The Solo distribution of Istio is a hardened Istio enterprise image, which maintains n-4 support for CVEs and other security fixes. The image support timeline is longer than the community Istio support timeline, which provides n-1 support with an additional six weeks of extended time to upgrade the n-2 version to n-1.

For more information, see [Solo Solutions](https://www.solo.io/) and [Solo Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/SoloGloo.gloo-saas-accelerator?tab=Overview).

### Tigera, Inc

:::image type="icon" source="./media/ecosystem-support/tigera.png":::

| Solution | Categories |
|----------|------------|
| Calico | Networking <br> Security <br> Observability |

Fully managed, active security platform with full-stack observability for containers and Kubernetes.

<details> <summary> See more </summary><br>

Calico Cloud enables organizations to prevent attacks using Zero Trust and to detect, troubleshoot, and automatically mitigate exposure risks from security breaches across multicloud and hybrid deployments. Calico Cloud is built on Calico Open Source, the most widely adopted container networking and security solution. It supports multiple data planes, including eBPF, Windows, and Linux.

</details>

For more information, see [Tigera Solutions](https://www.tigera.io/) and [Tigera on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/tigerainc1620235671643.calicocloudsaas?tab=overview).

## Observability

Observability provides deep insights into your systems, enabling rapid issue detection and resolution to enhance your application's reliability and performance.

### Dynatrace

:::image type="icon" source="./media/ecosystem-support/dynatrace.png":::

| Solution | Categories |
|----------|------------|
| Dynatrace Azure Monitoring | Observability |

Fully automated, AI-assisted observability across Azure environments Dynatrace is a single source of truth for your cloud platforms, allowing you to monitor the health of your entire Azure infrastructure.

For more information, see [Dynatrace Solutions](https://www.dynatrace.com/technologies/azure-monitoring/) and [Dynatrace on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/dynatrace.dynatrace_portal_integration?tab=Overview).

### Elastic

:::image type="icon" source="./media/ecosystem-support/elastic.png":::

| Solution | Categories |
|----------|------------|
| Elastic | Observability <br> Security |

Elastic, the Search AI Company, enables everyone to securely harness search powered AI to find the answers they need in real-time using all their data, at scale. Elastic's solutions for search, observability and security are built on the Elastic Search AI Platform, the development platform used by thousands of companies, including more than 50% of the Fortune 500.

For more information, see [Elastic Solutions](https://www.elastic.co/), [Elastic Observability on Azure Marketplace](https://marketplace.microsoft.com/product/saas/elastic.ec-azure-observability?ocid=azure-linux-os-guard-obs), and [Elastic Security on Azure Marketplace](https://marketplace.microsoft.com/product/saas/elastic.ec-azure-security?ocid=azure-linux-os-guard-sec).

### New Relic

:::image type="icon" source="./media/ecosystem-support/new-relic.png":::

| Solution | Categories |
|----------|------------|
| New Relic Azure Monitoring | Observability |

The New Relic Intelligent Observability Platform helps businesses eliminate interruptions in digital experiences.

<details> <summary> See more </summary><br>

New Relic is the only AI-driven platform to unify and pair telemetry data to provide clarity over your entire digital estate. We move your problem solving past proactive to predictive by processing the right data at the right time to maximize value and control costs.

</details>

For more information, see [New Relic Solutions](https://newrelic.com/solutions/partners/azure) and [New Relic on Azure Marketplace](https://marketplace.microsoft.com/product/newrelicinc1635200720692.newrelic_liftr_payg_2025?tab=Overview).

## Security

Ensure the integrity and confidentiality of applications and foster trust and compliance across your infrastructure.

### CrowdStrike

:::image type="icon" source="./media/ecosystem-support/crowdstrike.png":::

| Solution | Categories |
|----------|------------|
| Falcon Cloud Security   | Security   |

Powered by the CrowdStrike Security Cloud and world-class AI, the CrowdStrike Falcon&reg; platform leverages real-time indicators of attack, threat intelligence, evolving adversary tradecraft and enriched telemetry from across the enterprise to deliver hyper-accurate detections, automated protection and remediation, elite threat hunting and prioritized observability of vulnerabilities.

<details> <summary> See more </summary><br>

Purpose-built in the cloud with a single lightweight-agent architecture, the Falcon platform delivers rapid and scalable deployment, superior protection and performance, reduced complexity and immediate time-to-value.

</details>

For more information, see [CrowdStrike](https://crowdstrike.com).

### Palo Alto Networks

:::image type="icon" source="./media/ecosystem-support/palo-alto-networks.png":::

| Solution | Categories |
|----------|------------|
| Prisma Cloud Compute Edition | Security |

Prisma Cloud Compute Edition by Palo Alto Networks securely accelerates your time-to-market with support for Azure Linux for AKS and enhanced Kubernetes container security. Gain full lifecycle cloud workload protection (CWP) for hosts, containers, serverless functions, web applications, and APIs.

<details> <summary> See more </summary><br>

Protect against Layer 7 and OWASP Top 10 threats with Prisma Cloud security. Proactively reduce risk, detect vulnerabilities, and protect your applications. Agentless architecture options are also available for frictionless vulnerability scanning and risk assessment.

With Prisma Cloud by Palo Alto Networks you get always on, real-time app visibility and control to eliminate blind spots, reduce alerts, provide security guidance, and accelerate innovation.

</details>

For more information, see [Palo Alto Networks Solutions](https://www.paloaltonetworks.com/prisma/environments/azure) and [Prisma Cloud Compute Edition on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/paloaltonetworks.pcce_twistlock?tab=Overview).

### Qualys

:::image type="icon" source="./media/ecosystem-support/qualys.png":::

#### Qualys Cloud Agent

| Solution | Categories |
|----------|------------|
| Qualys Cloud Agent | Security |

The Qualys Cloud Agent is a lightweight, modular software agent that enables continuous, real-time security and compliance monitoring across various environments, including SaaS platforms, on-premises systems, and cloud infrastructures.

<details> <summary> See more </summary><br>

It supports a wide range of operating systems and architectures, such as Windows, Linux, macOS, AIX, Solaris, and specialized systems like AWS Bottlerocket. The agent provides functionalities such as vulnerability management, patching, endpoint protection, and file integrity monitoring, allowing customers to activate and deactivate capabilities based on their needs. Designed for ease of use, it facilitates seamless integration and management of security protocols across complex IT landscapes.

</details>

For more information, see [Qualys Cloud Agent Solutions](https://www.qualys.com/cloud-agent/).

#### Qualys Container Security

| Solution | Categories |
|----------|------------|
| Qualys Container Security | Security |

Qualys K8s and the Container Security solution provide proactive, preventive, and reactive security for containerized applications.

<details> <summary> See more </summary><br>

It integrates into your DevOps workflows, offering continuous real-time security and compliance throughout the containerized application lifecycle. Key features include:

- **Vulnerability management**: Identifies vulnerabilities in container images, registries, and running containers, prioritizes them, and helps mitigate the most critical vulnerabilities first.
- **Runtime protection**: eBPF-based runtime security monitors and protects containers in real-time, detecting and responding to malicious activities.
- **Compliance**: Ensures that Kubernetes configurations and container images adhere to best practices and compliance standards, preventing misconfigurations that might lead to security breaches.
- **File integrity monitoring**: Monitors changes to critical files within containers to detect and respond to unauthorized modifications.
- **Secret and malware detection**: Detects secrets and malware on the left side before container images are deployed in runtime, ensuring security from the development phase.

</details>

For more information, see [Qualys Container Security Solutions](https://www.qualys.com/apps/container-security/).

### Tetrate

:::image type="icon" source="./media/ecosystem-support/tetrate.png":::

| Solution | Categories |
|----------|------------|
| Tetrate Istio Distro (TID) | Security <br> Networking |

Tetrate Istio Distro (TID) is a simple, safe enterprise-grade Istio distro, providing the easiest way of installing, operating, and upgrading.

<details> <summary> See more </summary><br>

TID enforces fetching certified versions of Istio and enables only compatible versions of Istio installation. It includes a FIPS-compliant flavor, delivers platform-based Istio configuration validations by integrating validation libraries from multiple sources, uses various cloud provider certificate management systems to create Istio CA certs that are used for signing service mesh managed workloads, and provides multiple additional integration points with cloud providers.

</details>

For more information, see [Tetrate documentation](https://docs.tetrate.io/) and [Tetrate on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/tetrate1598353087553.tetrateistio?tab=overview).

### TrendAI

:::image type="icon" source="./media/ecosystem-support/trendai.png":::

| Solution | Categories |
|----------|------------|
| TrendAI | Security |

TrendAI Vision One™ Cloud Security is proactive security for cloud‑native and containerized workloads, helping organizations reduce risk as they run applications on Azure Kubernetes Service.

<details> <summary> See more </summary><br>

It gives teams a single, clear view of risk across container and runtime activity in AKS environments, helping them prioritize what matters most. Instead of adding more dashboards, it brings signals together and turns them into clear, actionable insight. TrendAI Vision One Cloud Security is designed to reduce friction from code to runtime, helping engineering and security teams stay aligned as Kubernetes environments scale and change. This allows organizations to manage risk more confidently while continuing to move quickly.

</details>

For more information, see [TrendAI documentation](https://www.trendmicro.com/en_gb/business/products/hybrid-cloud/cloud-one-container-image-security.html).

### Upwind

:::image type="icon" source="./media/ecosystem-support/upwind.png":::

| Solution | Categories |
|----------|------------|
| Upwind | Security |

Upwind secures your cloud deployments, configurations, and applications through a runtime fabric that provides real-time visibility from the inside out. Use Upwind to get a live map of your network and application topology, prioritize fixes based on real usage, and detect threats as they happen.

For more information, see [Upwind documentation](https://www.upwind.io/) and [Upwind on Azure Marketplace](https://marketplace.microsoft.com/product/saas/upwindsecurityinc1754856292483.upwind-security?tab=Overview).

### Wiz

:::image type="icon" source="./media/ecosystem-support/wiz.png":::

| Solution | Categories |
|----------|------------|
| WIZ Cloud Infrastructure Security Platform | Security |

Wiz is the unified cloud security platform for cloud security and development teams that includes prevention, active detection, and response. Use Wiz solution to reduce risk and gain unmatched visibility, accurate prioritization, and business agility.

For more information, see [Wiz Solutions](https://wiz.io/) and [Wiz on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/wizinc1627338511749.wiz-azure-marketplace?tab=overview).

## Storage

Storage enables standardized and seamless storage interactions, ensuring high application performance and data consistency.

### Veeam

:::image type="icon" source="./media/ecosystem-support/veeam.png":::

| Solution | Categories |
|----------|------------|
| Kasten K10 by Veeam | Storage |

Veeam Kasten is the #1 Kubernetes data management product, providing an easy-to-use, scalable, and secure system for backup and restore, mobility, DR, and ransomware protection.

For more information, see [Veeam Solutions](https://www.veeam.com/solutions/alliance-partners/kubernetes.html) and [Veeam on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/veeam.kasten_k10_by_veeam_byol?tab=overview).

## Config management

Automate and standardize the system settings across your environments to enhance efficiency, reduce errors, and ensuring system stability and compliance.

### Corent

:::image type="icon" source="./media/ecosystem-support/corent.png":::

| Solution | Categories |
|----------|------------|
| Corent MaaS | Config Management |

Corent MaaS provides scanning to identify workloads that can be containerized, and automatically containerizes on AKS.

For more information, see [Corent Solutions](https://www.corenttech.com/SurPaaS_MaaS_Product.html).

## Migration

Migrate workloads to Azure Linux Container Host on AKS with confidence.

### Catalogic

:::image type="icon" source="./media/ecosystem-support/catalogic.png":::

| Solution | Categories |
|----------|------------|
| CloudCasa by Catalogic | Migration <br> Storage |

CloudCasa by Catalogic is a Kubernetes backup, recovery, and migration solution that is fully compatible with AKS, as well as all other major Kubernetes distributions and managed services.

<details> <summary> See more </summary><br>

Install the CloudCasa agent and let it do all the hard work of protecting and recovering your cluster resources and persistent data from human error, security breaches, and service failures, including providing the business continuity and compliance that your business requires.

From a single dashboard, CloudCasa makes cross-cluster, cross-tenant, cross-region, and cross-cloud recoveries easy. Recovery and migration from backups includes recovering an entire cluster along with your vNETs, add-ons, load balancers and more. During recovery, users can migrate to Azure Linux, and migrate storage resources from Azure Disk to Azure Container Storage.

CloudCasa can also centrally manage Azure Backup or Velero backup installations across multiple clusters and cloud providers, with migration of resources to different environments.

</details>

For more information, see [CloudCasa by Catalogic Solutions](https://cloudcasa.io/) and [CloudCasa by Catalogic on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/catalogicsoftware1625626770507.cloudcasa-aks-app).

## Monitoring

Monitor your applications and infrastructure to ensure optimal performance and reliability.

### VictoriaMetrics

:::image type="icon" source="./media/ecosystem-support/victoriametrics.png":::

| Solution | Categories |
|----------|------------|
| VictoriaMetrics | DevOps <br> Observability <br> Storage <br> Monitoring |

VictoriaMetrics is a high performance, cost effective, and scalable open-source time series database and monitoring solution. This solution enables users to build a monitoring platform with minimal operational burden and without scalability issues.

<details> <summary> See more </summary><br>

You can use VictoriaMetrics to monitor services running in AKS with Azure Linux, the applications running in AKS with Azure Linux, and the underlying infrastructure. The enterprise version of VictoriaMetrics provides extra features for securing your monitoring setup, such as OIDC authentication and access control.

</details>

## Related content

- [Supported Azure services in Azure Linux](./supported-azure-services.md)
- [Package management on Azure Linux](./package-management-overview.md)
- [Azure Linux deployment options](./deployment-options.md)
