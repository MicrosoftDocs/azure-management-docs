# Securing your workloads

## Follow a secure container lifecycle as you acquire, catalog, and build your containers

Follow the Microsoft Containers Secure Supply Chain framework as you acquire, catalog, and build your workloads.  This will help you better protect against untrusted sources, avoid compromised dependencies, and scan for vulnerabilities.  

As part of this, look to obtain or generate a Software Bill of Materials (SBOM) to track code provenance.  You can also configure [Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-containers-introduction#vulnerability-assessment) to help perform a vulnerability assessment of images in your container registry – see the [support matrix](/azure/defender-for-cloud/support-matrix-defender-for-containers?tabs=extva%2Cazurert%2Cazurespm%2Cawsnet#vulnerability-assessment-va-features) for which registries are supported at what level (preview or general availability).

More generally, we recommend you follow the [Security Development Lifecycle](https://www.microsoft.com/en-us/securityengineering/sdl) as you develop your workload application software.

See also section 3.3 below about how to follow the Secure Supply Chain framework as you deploy and run your workloads.

### References
Reference: Section 4.1 and 4.4.1 of the NIST Application Container Security Guide
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Implement continuous vulnerability scanning”

## Prepare your container images for hardened Kubernetes environments

To help promote the execution of container images with least privileges at runtime, author your container images following steps to support non-root execution of workloads.  Non-root container images allow for the configuration of hardened pod security standards.  See section 2.3 below.

Use distroless and purpose-built container images that narrowly satisfy the necessary runtime components.  You can choose base images for your workloads from a reputable vendor such as from the [Microsoft Artifact Registry](https://mcr.microsoft.com/).  Narrowing your runtime dependencies helps both reduce the potential attack surface on ancillary components as well as the maintenance burden for updates resulting from security and bug fixes.  Leverage multi-stage builds to further reduce the footprint of your runtime images.  

Use [container signing](/azure/container-registry/container-registry-tutorial-sign-trusted-ca) to help ensure your workload images are not accidentally or maliciously tampered with.  Azure Pipelines as well as GitHub Actions provide various tooling in support of container signing.  You can integrate container signing with your chosen Public Key Infrastructure (PKI).

### References
Reference: NSA Kubernetes Hardening Guidance – ‘“Non-root” containers and “rootless” container engine’ and ‘Build secuire container images’
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Build Phase”

## Follow pod security standards

Follow the Kubernetes [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) for your pods, aiming for the restricted policy level where possible, and the baseline level otherwise.  For example, your containers should run as a non-root user and shouldn’t mount host volumes, unless this is strictly necessary for their operation.  See section 3.3 below about how you can help enforce these standards.  Note that not all Microsoft-supplied extensions are themselves able to meet the most restricted policy level, due to the privileged operations they perform, and so they may require additional capabilities to be permitted when deployed.

In addition, consider [setting memory limits and CPU requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for each pod (though not CPU limits which can lead to undesirable throttling) and also consider resource quotas for each namespace.  This can prevent broader Denial of Service (DoS) attacks from a container that is compromised or otherwise misbehaving.

### References
Reference: Pod Security Standards | Kubernetes
Reference: Sections 5.2 of the CIS Kubernetes Benchmark.  
Reference: NSA Kubernetes Hardening Guidance – “Pod security enforcement” and “Resource Policies”
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Apply security context” and “Limiting resource usage in a cluster”

## Enforce additional Linux security standards

Consider using [additional Linux security hardening frameworks](https://kubernetes.io/docs/concepts/security/linux-kernel-security-constraints/) that offer additional protections.  SELinux or AppArmor require and enforce a more precise declaration of the access that each workload has to the specific resources (files, ports, etc) that it requires, going beyond the standard Linux permissions model.  And [seccomp](https://kubernetes.io/docs/reference/node/seccomp/) restricts the system calls that each workload can make, note that a default seccomp profile is required at the Restricted level of the Pod Security Standards (which were introduced above). Note that all these security hardening features come with default settings, but you can typically tailor things further to your specific application: for example, you can define a custom seccomp profile that allows only the precise syscalls your pods need.

### References
Reference: Section 4.4.3 of the NIST Application Container Security Guide
Reference:  NSA Kubernetes Hardening Guidance – “Kubernetes Pod Security”

## Use workload identity for accessing Azure resources

Your workloads should use [workload identity federation](/azure/azure-arc/kubernetes/workload-identity) to help securely access your resources in Azure, such a storage account.  This approach helps avoid the need for you to create and distribute separate secrets to authenticate the Entra ID identities used for authorization with Azure RBAC.  Instead, this federation approach enables to you to obtain a token for an Entra ID identity in the cloud directly from a Kubernetes service account token. (Service accounts are Kubernetes’ built-in identity for workloads.)

Service account tokens are created and signed by your cluster’s service account token issuer.  The issuer’s private key is therefore a fundamental secret. It should be only be accessible where it is needed to issue tokens and it should be regularly rotated.

If you’re running AKS enabled by Azure Arc on Azure Local, then this is automatically taken care of: the keys are indeed restricted to access only by the control planes nodes running the API server (which runs the token issuer), the keys they expire after 90 days, and they are rotated automatically every 45 days.

If you’ve connected your own cluster via Arc-enabled Kubernetes, then evaluate if your vendor’s product offers similar capabilities to restrict access to the service account token issuer key, and to rotate it.

## Configure TLS encryption and authentication within/to/from workloads

Use TLS for all connections that your workloads make, both inside and outside of the cluster.  This may require you to generate certificates and distribute trust bundles for use in establishing these connections.  

For TLS connections within a cluster, evaluate using a service mesh such as [Istio](https://istio.io/latest/about/service-mesh/) to maintain the TLS connections for you. A service mesh such as Istio, or an application runtime such as DAPR, can help generate its own self-signed workload certificates.   For additional security, consider [configuring a cluster-local intermediate CA](https://istio.io/latest/docs/tasks/security/cert-management/plugin-ca-cert/) for use in signing these workload certificates, and in turn consider issuing the certificate for this intermediate CA using your Public Key Infrastructure (PKI) where the root certificate itself remains secured in an offline HSM-backed vault.

For ingress TLS connections into the cluster, evaluate using one of the many products (load balancers, ingress/gateway controllers, etc) to help you securely manage incoming traffic.  You may need to generate your own certificates for these endpoints to use, and then distribute trust bundles for these certificates to external clients that need to authenticate the connection into the cluster.  Again, consider issuing these certificates from your PKI: you can use [cert-manager](https://cert-manager.io/) to request these certs (here’s [how this works](https://kubernetes.github.io/ingress-nginx/user-guide/tls/#automated-certificate-management-with-cert-manager) for the NGINX ingress controller) and [trust-manager](https://cert-manager.io/docs/trust/trust-manager/) can help you distribute trust in them.  Or, if you only need a small number of certificates, then you might create and rotate them manually in [Azure Key Vault and sync them as Kubernetes secrets using the Azure Key Vault Secret Store extension for Kubernetes (preview)](https://learn.microsoft.com/en-gb/azure/azure-arc/kubernetes/secret-store-extension) ("SSE").

Once generated, these certificates and accompanying private keys are stored as secrets in the Kubernetes secret store.  See section 4.2 below on protecting these secrets. 

You may also use service account tokens for TLS (instead of certificates for mTLS) for the client-side authentication of connections within the cluster or of egress connections outside of the cluster.  If so, it’s recommended to avoid the use of the ‘default’ service account for each namespace and to create a dedicated service account identity for each separate workload or component.  This enables a least privilege approach as you configure authorization rules for your services (see section 2.7 below) or for your API server using Kubernetes RBAC (see section 3.2 below).   You can also help protect the service account issuer key itself, as described in section 2.5 above.

### References
Reference: Kubernetes Security - OWASP Cheat Sheet Series – “Implementing centralized policy management”
Reference: NSA Kubernetes Hardening Guidance – “Protecting Pod servicer account tokens”

## Configure authorization rules for accessing workloads/services

Aim to control traffic between your workloads by setting authorization rules about which workloads can send requests to your Kubernetes workloads (services).  As discussed in section 2.6 above, you can configure credentials for these calling workloads as either client certificates or service account tokens.

If you’re using a service mesh such as [Istio](https://istio.io/latest/about/service-mesh/), it can issue these identity credentials to each workload automatically, and you can then use these identities in [configuring authorization policies](https://istio.io/latest/docs/reference/config/security/authorization-policy/) that help restrict access to Kubernetes services only to the specified calling workloads. 

Alternatively, if you’re not using a service mesh, you can consider deploying a dedicated authorization engine such as [OPA](https://www.openpolicyagent.org/) with, for examples, its [Envoy plug-in](https://www.openpolicyagent.org/docs/latest/envoy-introduction/).

Also consider enforcing traffic restrictions can also be created at the network layer: see section 5.1 below.

## Maintain and monitor workload telemetry and plug it into a security management (SIEM) solution

Consider sending your workload telemetry to a monitoring solution, where it can be analyzed for potential security issues (as well was for product performance or debugging issues). 

You can deploy [Azure Monitor extensions](/azure/azure-monitor/containers/kubernetes-monitoring-enable) to your edge cluster that will automatically send Prometheus metrics to your Azure Monitor workspace and/or Container Insights logs to your Log Analytics workspace.  Prometheus metrics can be collected and analyzed at scale with Grafana dashboards.  Container Insights logs and performance data can be collected and analyzed using prebuilt views and workbooks.  As you set this up, note the [best practices for monitoring clusters](/azure/azure-monitor/containers/best-practices-containers) that covers reliability, cost optimization, performance, as well as security.

Further, the [Azure Monitor pipeline at edge (preview)](/azure/azure-monitor/essentials/edge-pipeline-configure?tabs=Portal) extension offers additional flexible options for telemetry collection in edge clusters.  Its capabilities include scalable routing from edge to cloud, local data caching, and delayed cloud syncing, which helps ensure reliable monitoring even in network-segmented or offline environments.  It can also receive telemetry data from a wider variety of remote sources and other agents using OpenTelemetry Protocol (OTLP) and syslog formats.

Once you have logs flowing into a Log Analytics workspace, you can also enable [Microsoft Sentinel](/azure/sentinel/overview?tabs=defender-portal) for  cyberthreat detection, investigation, response, and proactive hunting.

### References
Reference: NSA Kubernetes Hardening Guidance – “Logging”