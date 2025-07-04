---
title: Error Reference for Registry Health Checks
description: Error codes and possible solutions to problems found by running the az acr check-health diagnostic command in Azure Container Registry
ms.topic: reference
author: chasedmicrosoft
ms.author: doveychase
ms.service: azure-container-registry
ms.date: 10/31/2023
# Customer intent: As a system administrator, I want to understand error codes from the container registry health checks, so that I can quickly diagnose and resolve issues affecting Docker and Helm operations.
---
# Health check error reference

Following are details about error codes returned by the [az acr check-health][az-acr-check-health] command. For each error, possible solutions are listed.

For information about running `az acr check-health`, see [Check the health of an Azure container registry](container-registry-check-health.md).

## DOCKER_COMMAND_ERROR

This error means that Docker client for CLI couldn't be found. As a result, the following additional checks aren't run: finding Docker version, evaluating Docker daemon status, and running a Docker pull command.

*Potential solutions*: Install Docker client; add Docker path to the system variables.

## DOCKER_DAEMON_ERROR

This error means that the Docker daemon status is unavailable, or that it couldn't be reached using the CLI. As a result, Docker operations (such as `docker login` and `docker pull`) are unavailable through the CLI.

*Potential solutions*: Restart Docker daemon, or validate that it is properly installed.

## DOCKER_VERSION_ERROR

This error means that CLI wasn't able to run the command `docker --version`.

*Potential solutions*: Try running the command manually, make sure you have the latest CLI version, and investigate the error message.

## DOCKER_PULL_ERROR

This error means that the CLI wasn't able to pull a sample image to your environment.

*Potential solutions*: Validate that all components necessary to pull an image are running properly. Additionally, validate permissions and [troubleshoot registry login, authentication, and authorization](container-registry-troubleshoot-login-authn-authz.md). If you're using [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md) and authenticating with a Microsoft Entra identity (user, managed identity, or service principal), make sure the identity has permissions to the specific repository being accessed.

## HELM_COMMAND_ERROR

This error means that Helm client couldn't be found by the CLI, which precludes other Helm operations.

*Potential solutions*: Verify that Helm client is installed, and that its path is added to the system environment variables.

## HELM_VERSION_ERROR

This error means that the CLI was unable to determine the Helm version installed. This can happen if the Azure CLI version (or if the Helm version) being used is obsolete.

*Potential solutions*: Update to the latest Azure CLI version or to the recommended Helm version; run the command manually and investigate the error message.

## CMK_ERROR

This error means that the registry can't access the user-assigned or sysem-assigned managed identity used to configure registry encryption with a customer-managed key. The managed identity might have been deleted.  

*Potential solution*: To resolve the issue and rotate the key using a different managed identity, see steps to troubleshoot [the user-assigned identity](tutorial-troubleshoot-customer-managed-keys.md).

## CONNECTIVITY_DNS_ERROR

This error means that the DNS for the given registry login server was pinged but didn't respond, which means it is unavailable. This can indicate some connectivity issues. Alternatively, the registry might not exist, the user might not have the permissions on the registry (to retrieve its login server properly), or the target registry is in a different cloud than the one used in the Azure CLI.

*Potential solutions*: Validate connectivity; verify spelling of the registry, and that registry exists; verify that the user has the right permissions on it and that the registry's cloud is the same that is used in the Azure CLI.

## CONNECTIVITY_FORBIDDEN_ERROR

This error means that the challenge endpoint for the given registry responded with a 403 Forbidden HTTP status. This error means that users don't have access to the registry, most likely because of a virtual network configuration or because access to the registry's public endpoint isn't allowed. To see the currently configured firewall rules, run `az acr show --query networkRuleSet --name <registry>`.

*Potential solutions*: Remove virtual network rules, or add the current client IP address to the allowed list.

## CONNECTIVITY_CHALLENGE_ERROR

This error means that the challenge endpoint of the target registry didn't issue a challenge.

*Potential solutions*: Try again after some time. If the error persists, open an issue at https://aka.ms/acr/issues.

## CONNECTIVITY_AAD_LOGIN_ERROR

This error means that the challenge endpoint of the target registry issued a challenge, but the registry doesn't support Microsoft Entra authentication.

*Potential solutions*: Try a different way to authenticate, for example, with admin credentials. If users need  to authenticate using Microsoft Entra ID, open an issue at https://aka.ms/acr/issues.

## CONNECTIVITY_REFRESH_TOKEN_ERROR

This error means that the registry login server didn't respond with a refresh token, so access to the target registry was denied. This error can occur if the user doesn't have the right permissions on the registry or if the user credentials for the  Azure CLI are stale.

*Potential solutions*: Verify if the user has the right permissions on the registry; run `az login` to refresh permissions, tokens, and credentials. Additionally, validate permissions and [troubleshoot registry login, authentication, and authorization](container-registry-troubleshoot-login-authn-authz.md). If you're using [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md) and authenticating with a Microsoft Entra identity (user, managed identity, or service principal), make sure the identity has permissions to the specific repository being accessed.

## CONNECTIVITY_ACCESS_TOKEN_ERROR

This error means that the registry login server didn't respond with an access token, so that the access to the target registry was denied. This error can occur if the user doesn't have the right permissions on the registry or if the user credentials for the Azure CLI are stale.

*Potential solutions*: Verify if the user has the right permissions on the registry; run `az login` to refresh permissions, tokens, and credentials. Additionally, validate permissions and [troubleshoot registry login, authentication, and authorization](container-registry-troubleshoot-login-authn-authz.md). If you're using [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md) and authenticating with a Microsoft Entra identity (user, managed identity, or service principal), make sure the identity has permissions to the specific repository being accessed.

## CONNECTIVITY_SSL_ERROR

This error means that the client was unable to establish a secure connection to the container registry. This error generally occurs if you're running or using a proxy server.

*Potential solutions*: More information on working behind a proxy can be [found here](/cli/azure/use-cli-effectively).

## LOGIN_SERVER_ERROR

This error means that the CLI was unable to find the login server of the given registry, and no default suffix was found for the current cloud. This error can occur if the registry doesn't exist, if the user doesn't have the right permissions on the registry, if the registry's cloud and the current Azure CLI cloud do not match, or if the Azure CLI version is obsolete.

*Potential solutions*: Verify that the spelling is correct and that the registry exists; verify that user has the right permissions on the registry, and that the clouds of the registry and the CLI environment match; update Azure CLI to the latest version.

## NOTARY_VERSION_ERROR

This error means that the CLI isn't compatible with the currently installed version of Docker/Notary. Try downgrading your notary.exe version to a version earlier than 0.6.0 by replacing your Docker installation's Notary client manually to resolve this issue. You can also try downloading and installing a pre-compiled binary of Notary earlier than 0.6.0 for 64 bit Linux or macOS X from the Notary repository's releases page on GitHub. For windows download the .exe, place it in the(default path:  C:\ProgramFiles\Docker\Docker\resources\bin) and rename it to notary.exe. 

## CONNECTIVITY_TOOMANYREQUESTS_ERROR

This error means that the user has sent too many requests in a short period causing the authentication system to block further requests to prevent overload. This error occurs by reaching a configured limit in the user's registry service tier or environment. We recommend waiting for a moment before sending another request. This will allow the authentication system's block to lift and you can try sending a request again.  

## Next steps

For options to check the health of a registry, see [Check the health of an Azure container registry](container-registry-check-health.md).

See the [FAQ](container-registry-faq.yml) for frequently asked questions and other known issues about Azure Container Registry.





<!-- LINKS - internal -->
[az-acr-check-health]: /cli/azure/acr#az_acr_check_health
