---
title: "Secure your workloads in Azure Arc-enabled Kubernetes"
ms.date: 06/06/2025
ms.topic: concept-article
description: "Guidance for securing workloads running on Azure Arc-enabled Kubernetes clusters, including application and container security."
---

# Secure your workloads

Securing your workloads involves building, deploying, and running containers in ways that help reduce risk and align with industry standards. This article provides guidance on container security, pod security standards, and workload identity, helping you protect your applications from supply chain risks, unauthorized access, and runtime threats within and outside your clusters.

## Follow a secure container lifecycle as you acquire, catalog, and build your containers

Follow the [Microsoft Containers Secure Supply Chain framework](/azure/security/container-secure-supply-chain/articles/container-secure-supply-chain-implementation/containers-secure-supply-chain-overview) as you acquire, catalog, and build your workloads. This framework helps you better protect against untrusted sources, avoid compromised dependencies, and scan for vulnerabilities. 

Track code provenance by obtaining or generating a Software Bill of Materials (SBOM). You can also configure [Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-containers-introduction#vulnerability-assessment) to help perform a vulnerability assessment of images in your container registry – see the [support matrix](/azure/defender-for-cloud/support-matrix-defender-for-containers?tabs=extva%2Cazurert%2Cazurespm%2Cawsnet#vulnerability-assessment-va-features) for which registries are supported at what level (preview or general availability).

More generally, we recommend you follow the [Security Development Lifecycle](https://www.microsoft.com/securityengineering/sdl) as you develop your workload application software.

See also [our guidance](conceptual-secure-your-operations.md#follow-a-secure-container-lifecycle-as-you-deploy-and-run-containers-with-azure-policy-for-kubernetes) on how to follow the Secure Supply Chain framework as you deploy and run your workloads.

### References
* [CIS Kubernetes Benchmark - Sections 1, 2, and 4](https://www.cisecurity.org/benchmark/kubernetes)
* [NIST Application Container Security Guide - Section 4.5.1-3](https://csrc.nist.gov/pubs/sp/800/190/final)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Implement continuous vulnerability scanning"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Prepare your container images for hardened Kubernetes environments

To help promote the execution of container images with least privileges at runtime, author your container images to support nonroot execution of workloads.

Use distroless and purpose-built container images that narrowly satisfy the necessary runtime components. You can choose base images for your workloads from a reputable vendor such as from the [Microsoft Artifact Registry](https://mcr.microsoft.com/). Narrowing your runtime dependencies helps both reduce the potential attack surface on ancillary components and the maintenance burden resulting from security updates and bug fixes. Use multi-stage builds to further reduce the footprint of your runtime images. 

Use [container signing](/azure/container-registry/container-registry-tutorial-sign-trusted-ca) to help ensure your workload images aren't accidentally or maliciously tampered with. Azure Pipelines and GitHub Actions provide various tooling in support of container signing. You can integrate container signing with your chosen Public Key Infrastructure (PKI).

### References
* [NSA Kubernetes Hardening Guidance – "Nonroot containers and rootless container engine" and æBuild secure container images"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Build Phase"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Follow pod security standards

Follow the Kubernetes [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) for your pods, aiming for the restricted policy level where possible, and the baseline level otherwise. For example, your containers should run as a nonroot user and shouldn’t mount host volumes, unless it's strictly necessary for their operation. See [our suggestions](conceptual-secure-your-operations.md#follow-a-secure-container-lifecycle-as-you-deploy-and-run-containers-with-azure-policy-for-kubernetes) about how you can help enforce these standards. Not all Microsoft-supplied extensions are themselves able to meet the most restricted policy level, due to the privileged operations they perform. They may require extra capabilities to be permitted when deployed.

In addition, consider [setting memory limits and CPU requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for each pod (though not CPU limits which can lead to undesirable throttling). Also consider resource quotas for each namespace. These limits and quotas can prevent broader Denial of Service (DoS) attacks from a container that is compromised or otherwise misbehaving.

### References
* [Pod Security Standards | Kubernetes](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
* [Sections 5.2 of the CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)  
* [NSA Kubernetes Hardening Guidance – "Pod security enforcement" and "Resource Policies"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* [Kubernetes Security - OWASP Cheat Sheet Series – "Apply security context" and "Limiting resource usage in a cluster"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)

## Enforce extra Linux security standards

Consider using [extra Linux security hardening frameworks](https://kubernetes.io/docs/concepts/security/linux-kernel-security-constraints/) that offer further protections. SELinux or AppArmor require and enforce a more precise declaration of the access that each workload has to specific resources such as files and ports. These declarations go beyond the standard Linux permissions model. Further, [seccomp](https://kubernetes.io/docs/reference/node/seccomp/) restricts the system calls that each workload can make, and a default seccomp profile is required at the Restricted level of the Pod Security Standards. All these security hardening features come with default settings, but you can typically tailor things further to your specific application. For example, you can define a custom seccomp profile that allows only the precise syscalls your pods need.

### References
* [Section 4.4.3 of the NIST Application Container Security Guide](https://csrc.nist.gov/pubs/sp/800/190/final)
* [NSA Kubernetes Hardening Guidance – "Kubernetes Pod Security"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

## Use workload identity for accessing Azure resources

Your workloads should use [workload identity federation](/azure/azure-arc/kubernetes/workload-identity) to help securely access your resources in Azure, such a storage account. This approach helps avoid the need for you to create and distribute separate secrets to authenticate the Entra ID identities used for authorization with Azure RBAC. Instead, this federation approach enables to you to obtain a token for an Entra ID identity in the cloud directly from a Kubernetes service account token. (Service accounts are Kubernetes’ built-in identity for workloads.)

Your cluster’s service account token issuer creates and signs these service account tokens. The issuer’s private key is therefore a fundamental secret. Access to it should be restricted and it should be regularly rotated.

If you’re running AKS enabled by Azure Arc on Azure Local, then this access control is automatically taken care of. Only the control planes nodes that run the API server (which includes the token issuer) can access the keys. The keys expire after 90 days, and they're rotated automatically every 45 days.

If you connect your own cluster via Arc-enabled Kubernetes, then evaluate if your vendor’s product offers similar capabilities. Check if they restrict access to the service account token issuer key and if they rotate it regularly.

## Configure TLS encryption and authentication within/to/from workloads

Use TLS for all connections that your workloads make, both inside and outside of the cluster. Using TLS may require you to generate certificates and distribute trust bundles for use in establishing these connections. 

For TLS connections within a cluster, evaluate using a service mesh such as [Istio](https://istio.io/latest/about/service-mesh/) to maintain the TLS connections for you. A service mesh such as Istio, or an application runtime such as DAPR, can help generate its own self-signed workload certificates. For extra security, consider [configuring a cluster-local intermediate CA](https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/) for use in signing these workload certificates. If you do so, then also consider issuing the certificate for this intermediate CA using your Public Key Infrastructure (PKI), where the root certificate itself remains secured in an offline HSM-backed vault.

For ingress TLS connections into the cluster, evaluate using one of the many products such as load balancers and ingress/gateway controllers that can help you securely manage incoming traffic. You may need to generate your own certificates for these endpoints to use. You may also need to distribute trust bundles for these certificates to external clients that need to authenticate their connection into the cluster. Again, consider issuing these certificates from your PKI. You can use [cert-manager](https://cert-manager.io/) to request these certs: here’s [how this request works](https://kubernetes.github.io/ingress-nginx/user-guide/tls/#automated-certificate-management-with-cert-manager) for the NGINX ingress controller. And you can use [trust-manager](https://cert-manager.io/docs/trust/trust-manager/) to help you distribute trust in them. Or, if you only need a few certificates, then you might create and rotate them manually in Azure Key Vault and sync them as Kubernetes secrets using the [Azure Key Vault Secret Store extension for Kubernetes (preview)](/azure/azure-arc/kubernetes/secret-store-extension) ("SSE").

Once generated, these certificates and accompanying private keys are stored as secrets in the Kubernetes secret store. See this [guidance on protecting these secrets](conceptual-secure-your-data.md#protect-the-kubernetes-secrets-store). 

You may also use service account tokens for TLS, instead of certificates for mTLS, for client-side authentication of connections within the cluster. If so, we recommend avoiding the use of the "default" service account for each namespace. Instead, create a dedicated service account identity for each separate workload or component. Doing so enables a least privilege approach as you configure authorization rules [for your services](conceptual-secure-your-workloads.md#configure-authorization-rules-for-accessing-workloadsservices) or [for your API server using Kubernetes RBAC](conceptual-secure-your-operations.md#control-who-can-deploy-to-your-cluster-with-role-based-access-control-rbac).

### References
* [Kubernetes Security - OWASP Cheat Sheet Series – "Implementing centralized policy management"](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)
* [NSA Kubernetes Hardening Guidance – "Protecting Pod servicer account tokens"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

## Configure authorization rules for accessing workloads/services

Aim to control traffic between your workloads by setting authorization rules about which workloads can send requests to your Kubernetes workloads (services).

If you’re using a service mesh such as [Istio](https://istio.io/latest/about/service-mesh/), it can issue identity credentials to each workload automatically. You can then use these identities in [configuring authorization policies](https://istio.io/latest/docs/reference/config/security/authorization-policy/) that help restrict access to Kubernetes services only to the specified calling workloads. 

Alternatively, if you’re not using a service mesh, you can configure your own credentials (certificates or service account tokens) and deploy a dedicated authorization engine such as [OPA](https://www.openpolicyagent.org/). OPA can be used with its [Envoy plug-in](https://www.openpolicyagent.org/docs/latest/envoy-introduction/) to avoid the need to modify your workloads.

Also [consider enforcing traffic restrictions at the network layer](conceptual-secure-your-network.md#configure-kubernetes-network-policy-to-control-access-tofrom-your-workloads).

## Maintain and monitor workload telemetry and plug it into a security management (SIEM) solution

Consider sending your workload telemetry to a monitoring solution, where it can be analyzed for potential security issues (as well was for product performance or debugging issues). 

You can deploy [Azure Monitor extensions](/azure/azure-monitor/containers/kubernetes-monitoring-enable) to your edge cluster. These extensions automatically send Prometheus metrics to your Azure Monitor workspace and/or Container Insights logs to your Log Analytics workspace. Prometheus metrics can be collected and analyzed at scale with Grafana dashboards. Container Insights logs and performance data can be collected and analyzed using prebuilt views and workbooks. As you set up this analysis, be aware of the [best practices for monitoring clusters](/azure/azure-monitor/containers/best-practices-containers) that cover reliability, cost optimization, performance, and security.

Further, the [Azure Monitor pipeline at edge (preview)](/azure/azure-monitor/essentials/edge-pipeline-configure?tabs=Portal) extension offers other flexible options for telemetry collection in edge clusters. Its capabilities include scalable routing from edge to cloud, local data caching, and delayed cloud syncing, which helps ensure reliable monitoring even in network-segmented or offline environments. It can also receive telemetry data from a wider variety of remote sources and other agents using OpenTelemetry Protocol (OTLP) and syslog formats.

Once you have logs flowing into a Log Analytics workspace, you can also enable [Microsoft Sentinel](/azure/sentinel/overview?tabs=defender-portal) for  cyberthreat detection, investigation, response, and proactive hunting.

### References
* [NSA Kubernetes Hardening Guidance – "Logging"](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

## Next steps

- Learn how to [secure your operations](conceptual-secure-your-operations.md)
- Return to the top of this [security book](conceptual-security-book.md)