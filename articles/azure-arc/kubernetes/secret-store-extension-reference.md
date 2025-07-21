---
title: Reference for Azure Key Vault Secret Store Extension
description: A reference guide for the Azure Key Vault Secret Store Extension, documenting the possibilities allowed in each of SSE's configuration resources.
ms.date: 08/01/2025
---

## Arc Extension Configuration parameters

Configuration parameters can be set when the SSE arc extension instance is created, or can be updated later. Use ```--configuration-settings <setting>=<value>``` with ```az k8s-extension create ...``` or ```az k8s-extension update ...``` to create or update an SSE instance respectively.

SSE accepts the following arc extension configuration parameters:

   | Parameter name                    | Description                         | Default value                         |
   |---------------------------------|-------------------------------------------------------------------------------|----------------------------------------------|
   | `rotationPollIntervalInSeconds`          | Specifies how quickly the SSE checks or updates the secret it's managing.       | `3600` (1 hour)                                             |

## SecretSync configuration

Soon

## SecretProviderClass configuration

Soon