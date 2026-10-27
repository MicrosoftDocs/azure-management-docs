---
title: Resolve Azure Advisor recommendations for Azure Arc-enabled servers
description: Learn how to resolve Azure Advisor reliability recommendations for disconnected and expired Azure Arc-enabled servers.
ms.date: 09/09/2026
ms.topic: how-to
# Customer intent: As an Azure Arc administrator, I want to resolve Azure Advisor recommendations for unhealthy Arc-enabled servers, so that I can restore connectivity and avoid managed identity expiration.
---

# Resolve Azure Advisor recommendations for Azure Arc-enabled servers

Azure Advisor analyzes Azure Arc-enabled servers and recommends actions that can improve their reliability. Use this article to understand and resolve recommendations for disconnected or expired servers.

The following Azure Advisor recommendations are available for Azure Arc-enabled servers:

| Recommendation                                                                                                                                                | Severity | Condition                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------------------------- |
| [Restore connectivity for 30+ day disconnected Arc-enabled server](#restore-connectivity-for-30-day-disconnected-arc-enabled-server)                          | Medium   | The server is disconnected for 30 through 44 days.                                  |
| [Reconnect Arc-enabled server before the managed identity certificate expires](#reconnect-arc-enabled-server-before-the-managed-identity-certificate-expires) | High     | The server is disconnected for at least 45 days and the certificate is still valid. |
| [Re-onboard expired Arc-enabled server](#re-onboard-expired-arc-enabled-server)                                                                               | High     | The server's status is **Expired**, or its managed identity certificate is expired.      |

## Restore connectivity for 30+ day disconnected Arc-enabled server

Azure shows this recommendation when a server remains disconnected for 30 through 44 days. Azure didn't receive a heartbeat from the server for an extended period. If you don't restore connectivity, the server's managed identity credential might expire, and Azure management capabilities might stop working.

To restore connectivity:

1. Sign in to the server.
1. Run the following command to [check the agent status and dependent services](azcmagent-show.md):

   ```console
   azcmagent show
   ```

1. Confirm that the agent services are running. If a service isn't running, start or restart it, and then run `azcmagent show` again.
1. Run the following command to [test connectivity to the required Azure Arc endpoints](azcmagent-check.md):

   ```console
   azcmagent check
   ```

1. Resolve any failed connectivity checks. Review the [network requirements](network-requirements.md) and, if the server uses a proxy, [verify the agent proxy configuration](manage-agent-proxy-settings.md).
1. If the problem continues, run the following command to [collect the full agent log bundle](azcmagent-logs.md):

   ```console
   azcmagent logs --full
   ```

   Use the logs to investigate the failure. For help with common errors, see [Troubleshoot Azure Connected Machine agent connection problems](troubleshoot-agent-onboard.md#agent-connection-issues-to-service).

1. In the Azure portal, confirm that the server's status returns to **Connected**.

To receive an alert when a server becomes disconnected, [create a Resource Health alert](troubleshoot-connectivity.md#disconnected-server-alerts).

## Reconnect Arc-enabled server before the managed identity certificate expires

Azure shows this recommendation when the server is disconnected for at least 45 days but the certificate is still valid.

Restore connectivity immediately so that the agent can renew its managed identity credential and avoid expiration. The certificate is valid for 90 days, and the agent attempts to renew it when 45 or fewer days of validity remain. For more information, see [Managed identity security overview](security-identity-authorization.md#microsoft-entra-id-managed-identity).

To reconnect the server before the certificate expires:

1. Sign in to the server.
1. Run the following command to [check the agent status and dependent services](azcmagent-show.md):

   ```console
   azcmagent show
   ```

1. Confirm that the agent services are running. If a service isn't running, start or restart it, and then run `azcmagent show` again.
1. Run the following command to [test connectivity to the required Azure Arc endpoints](azcmagent-check.md):

   ```console
   azcmagent check
   ```

1. Resolve any failed connectivity checks. Review the [network requirements](network-requirements.md) and, if the server uses a proxy, [verify the agent proxy configuration](manage-agent-proxy-settings.md).
1. If the problem continues, run the following command to [collect the full agent log bundle](azcmagent-logs.md):

   ```console
   azcmagent logs --full
   ```

   Use the logs to investigate the failure. For help with common errors, see [Troubleshoot Azure Connected Machine agent connection problems](troubleshoot-agent-onboard.md#agent-connection-issues-to-service).

1. In the Azure portal, confirm that the server's status returns to **Connected**.

## Re-onboard expired Arc-enabled server

Azure shows this recommendation when the server's status is **Expired** or its managed identity certificate has expired. An expired credential can't renew automatically, and Azure Arc management capabilities are unavailable until you reconnect the server with onboarding credentials.

First, confirm whether the server should remain managed by Azure Arc.

- If the server should remain Arc-enabled, [disconnect the agent and reconnect the server by using an onboarding credential](security-identity-authorization.md#microsoft-entra-id-managed-identity). To generate and run a new onboarding script, see [Connect hybrid machines to Azure using a deployment script](onboard-portal.md).
- If the server is retired, [remove its stale Azure resource](uninstall-agent.md#remove-stale-server-resources). If you still have access to the server, [remove VM extensions, disconnect the server, and uninstall the agent](uninstall-agent.md).

After re-onboarding a server, confirm in the Azure portal that its status is **Connected**.

## Next steps

- Learn how [Azure Arc-enabled servers operate in disconnected scenarios](troubleshoot-connectivity.md).
- Review the [Azure Connected Machine agent overview](agent-overview.md).
