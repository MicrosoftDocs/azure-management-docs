---
title: "Monitor and troubleshoot cert-manager for Arc-enabled Kubernetes (preview)"
ms.date: 03/03/2026
ms.topic: how-to
description: "Learn how to monitor the health of the cert-manager for Azure Arc-enabled Kubernetes (preview) extension and troubleshoot problems."
# Customer intent: As a customer using cert-manager for Azure Arc-enabled Kubernetes, I want to understand how to monitor the health of the extension and troubleshoot problems, so that my workloads remain secure and compliant.
---

# Monitor and troubleshoot cert-manager for Azure Arc-enabled Kubernetes (preview)

This article explains how to monitor extension status, check the health of cert-manager and trust-manager, and troubleshoot problems related to cert-manager for Azure Arc-enabled Kubernetes (preview).

## Confirm successful extension installation

In the Azure portal, go to your Arc-enabled Kubernetes cluster. In the service menu, under **Settings**, select **Extensions**. You should see the cert-manager for Azure Arc-enabled Kubernetes extension, using either the name you provided or **microsoft.certmanagement**.

Ensure that the extension status is **Installed** or **Succeeded**. If the status shows **Failed**, try reinstalling the extension, or check your Azure Activity log for possible errors.

## Confirm that pods and components are running

First, run this kubectl command to list the pods in the `cert-manager` namespace:

```bash
kubectl get pods -n cert-manager
```

You should see the following pods running:

- `cert-manager`: The main cert-manager controller.
- `cert-manager-webhook`: Used for Certificate CRD webhooks.
- `cert-manager-cainjector`: Helper for injecting CA into webhooks.
- `trust-manager`: The trust bundle manager.

You might also see additional pods. For example, you might see output similar to the following example:

```output
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-xxxxxxxx-xxxxx               2/2     Running   0          1m
cert-manager-xxxxxxxx-xxxxx               2/2     Running   0          1m
cert-manager-cainjector-xxxxxxxx-xxxxx    2/2     Running   0          1m
cert-manager-cainjector-xxxxxxxx-xxxxx    2/2     Running   0          1m
cert-manager-webhook-xxxxxxxx-xxxxx       2/2     Running   0          1m
cert-manager-webhook-xxxxxxxx-xxxxx       2/2     Running   0          1m
cert-manager-webhook-xxxxxxxx-xxxxx       2/2     Running   0          1m
trust-manager-xxxxxxxx-xxxxx              2/2     Running   0          1m
```

If the cert-manager and trust-manager pods aren't running:

- Run `kubectl describe pod <name> -n cert-manager` to see events. For example, a webhook pod might fail if it can't obtain a certificate for itself due to network policies or firewall blocking the necessary connectivity.
- Check container logs and look for any errors:

   ```bash
   kubectl logs deployment/cert-manager -n cert-manager
   kubectl logs deployment/cert-manager-webhook -n cert-manager
   kubectl logs deployment/trust-manager -n cert-manager
   ```

After confirming that pods are running, run the following kubectl command to confirm that the Custom Resource Definitions (CRDs) for cert-manager and trust-manager are installed:

```bash
kubectl get crds | grep cert-manager
```

This command should list CRDs such as `certificates.cert-manager.io`, `issuers.cert-manager.io`, `clusterissuers.cert-manager.io`, and `bundles.trust.cert-manager.io`. For example:

```output

bundles.trust.cert-manager.io                          2025-08-14T15:11:27Z
certificaterequests.cert-manager.io                    2025-08-14T15:11:27Z
certificates.cert-manager.io                           2025-08-14T15:11:27Z
challenges.acme.cert-manager.io                        2025-08-14T15:11:27Z
clusterissuers.cert-manager.io                         2025-08-14T15:11:27Z
issuers.cert-manager.io                                2025-08-14T15:11:27Z
orders.acme.cert-manager.io                            2025-08-14T15:11:27Z
```

These CRDs indicate that the API for managing certificates and trust bundles is available in your cluster, and you can start issuing and managing certificates.

## Troubleshoot certificates not being issued

If a certificate isn't getting issued (secret not created, or status of certificate remains **False**), check the following:

- Run `kubectl describe certificate <name>` to see events. You might see a message explaining the reason, such as "Issuer not ready" or "Waiting on certificate approval."
- Ensure the referenced Issuer/ClusterIssuer exists and is ready by running `kubectl get issuer,clusterissuer -A` to list issuers. If the issuer isn't ready, run `kubectl describe` to get more information. For example, this could be caused by a missing or misnamed backing secret for a Certificate A issuer, or an Automatic Certificate Management Environment (ACME) issuer failing an HTTP-01 challenge.
- If using an ACME issuer such as Let's Encrypt, run  `kubectl get challenge,order -A` for ACME challenge objects and describe them for details (which could show problems such as "DNS record not found for DNS-01" or "Ingress not reachable for HTTP-01").
- Check cert-manager controller logs by running `kubectl logs deploy/cert-manager -n cert-manager` around the time of the request. Logs might include warnings explaining why a certificate couldn't be issued.

## Troubleshoot trust bundles not propagating

If trust bundles aren't propagating as expected, check the following:

- Run `kubectl describe bundle <name> -n <ns>` and check the status section. For example, you might see that the source secret or configmap couldn't be found.
- Confirm that you used the correct namespace and name for the source secret or configmap specified in the bundle.
- Run `kubectl logs deploy/trust-manager -n cert-manager` to check the trust-manager logs for messages regarding bundles.
- Verify if the ConfigMap in target namespaces is created or updated. If not, trust-manager might not have the necessary permissions. By default, the extension sets up the needed ClusterRole that allows trust-manager to create ConfigMaps cluster-wide, but it's possible that the permissions were modified.

## Resolve common errors in cert-manager for Azure Arc-enabled Kubernetes (preview)

### Error due to resource already existing in your cluster

This error occurs when you try to install the extension on a cluster that already has cert-manager or trust-manager components installed.

To resolve this error, you can either uninstall the existing cert-manager and trust-manager components from your cluster before installing the extension, or choose a different cluster that doesn't have these components installed. For more information, see [Migrate from open source cert-manager and trust-manager](cert-manager-deploy.md#migrate-from-open-source-cert-manager-and-trust-manager).

### Certificate signed by unknown authority

If you see "x509: certificate signed by unknown authority" errors in your application logs, your application doesn't trust the issuer that signed a certificate. Use trust-manager to distribute the relevant certification authority (CA), or configure the application to trust the CA.

For example, if using a self-signed ClusterIssuer, ensure that all clients trust that self-signed root.

### Issuer is not ready

If you see "Issuer is not ready (Error: secret not found)", fix the `spec.ca.secretName` value in the issuer to point to the correct secret that contains the CA's key pair.

### ACME HTTP-01 challenge failed

If you use an ACME issuer and see am HTTP-01 challenge failed error, ensure that an appropriate ingress controller is in place and that the HTTP challenge ingress can be reached. Normally, cert-manager creates a temporary ingress to serve a token at `http://<yourdomain>/.well-known/acme-challenge/`. If that doesn't happen, or if your cluster can't be reached, the challenge fails. If so, consider using DNS-01 challenge (requires access to DNS provider API) or use another method.

To check challenge resource status, run `kubectl describe challenge <name>` to look for reasons, such as a 404 error on the URL or DNS TXT not found.

### Certificate renewed but secret not updated in pod

Kubernetes updates volume mounts with new secret content, but some processes might not notice. For example, Java applications might need a restart or a specific signal to reload keystores.

To resolve this, update your application so that it's prepared to reload certificates if necessary. Some applications use sidecars or cert-reloader processes for this purpose.