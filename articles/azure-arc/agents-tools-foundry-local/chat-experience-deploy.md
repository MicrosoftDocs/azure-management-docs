---
title: Deploy and Validate Chat UI for Agents and Tools with Foundry Local
description: Deploy chat UI as an optional component for Agents and Tools with Foundry Local, then validate routing, auth mode, and streaming behavior.
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 05/22/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-assisted

#customer intent: As a platform engineer, I want to deploy and validate chat UI as an optional component so users can chat with my configured agent.
---

# Deploy and validate chat UI for Agents and Tools with Foundry Local

The chat UI is an optional web front end for Agents and Tools with Foundry Local. This article helps you deploy the chat UI workload and validate that browser requests reach `agents-runtime`.

## Prerequisites

Complete the following prerequisites before you deploy the chat UI.

- Review [Chat UI in Agents and Tools with Foundry Local](chat-experience.md) for the architecture and deployment context. Use [Chat UI configuration and API reference](chat-experience-reference.md) for configuration details and runtime behavior.
- Deploy Agents and Tools with Foundry Local and confirm `agents-runtime` is running in the `arc-rag` namespace. For verification steps, see [Verify deployment by mode](deploy.md#verify-deployment-by-mode).
- Complete data query setup in [Quickstart: Query your data with Agents and Tools with Foundry Local](quickstart-create-agent.md): ingest data, create and register a knowledge source, and link it to the default knowledge base.
- For connected deployments, complete [Configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md).
- Get your agent ID. For steps, see [Get your agent ID](#get-your-agent-id).
- Decide whether to host chat UI at the root path (`/`) or a shared subpath (for example, `/chat`).
- Decide on an identity mode: connected (Microsoft Entra ID sign-in) or disconnected or air-gapped (no Microsoft Entra sign-in).

### Get your agent ID

Use the default knowledge base ID (`KB_ID`) as your agent ID.

If needed, complete [Quickstart: Query your data with Agents and Tools with Foundry Local](quickstart-create-agent.md), then run this command from Step 4 to get the value:

```bash
KB_ID=$(curl -s "https://$CLUSTER/knowledge-bases?limit=1" \
  -H "Authorization: Bearer $TOKEN" | jq -r '.data[0].id')
echo "Knowledge Base: $KB_ID"
```

Use this `KB_ID` value for both:

- `VITE_AGENT_ID`
- `<agent-id>` placeholder in deployment commands

### Gather required deployment values

Before you set chat UI environment variables, make sure you know where each value comes from and where to set it.

| Value | Where to get it | Where to set it |
|---|---|---|
| `VITE_API_MODE=agents` | Fixed value for Agentic RAG mode. | Chat UI pod environment variables (Helm values that render the chat UI deployment, or direct Kubernetes env vars). |
| `VITE_API_URL=/runtime` | Your ingress path to `agents-runtime`. This article uses `/runtime/*` on the same host as chat UI. | Chat UI pod environment variables. |
| `VITE_AGENT_ID` | The agent identifier used by the runtime. Use the default knowledge base ID (`KB_ID`) from [Get your agent ID](#get-your-agent-id). | Chat UI pod environment variables. |
| `VITE_BASE_PATH` | Your chosen UI route path. Use `/` for root hosting or `/chat` for shared-subpath hosting. | Chat UI pod environment variables. |
| `VITE_AUTH_CLIENT_ID`, `VITE_AUTH_TENANT_ID` | Microsoft Entra app registration values from your auth setup. See [Configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md). | Chat UI pod environment variables (connected deployments only). |

For cluster domain and ingress host context, use the same domain that you configured during extension deployment and app registration. See [Deploy Agents and Tools with Foundry Local](deploy.md) and [Configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md).

## Choose a deployment method

Choose one of these methods:

- **kubectl commands:** Create and configure resources directly in the cluster without a manifest file.
- **Kubernetes manifest:** Use a single manifest file that contains Deployment, Service, and Ingress.

## Deploy chat UI

In this section, you deploy the chat UI front end, configure its runtime settings, and expose it so browser requests can reach both the UI and `agents-runtime`.

#### [kubectl commands](#tab/kubectl-commands)

Use this method when you want to create and configure chat UI directly from commands instead of managing a manifest file.

1. Create the chat UI deployment. Replace `<chat-ui-image>` with the container image repository for chat UI, and replace `<tag>` with the image tag or version to deploy:

    ```bash
    kubectl -n arc-rag create deployment chat-ui-frontend --image=<chat-ui-image>:<tag> --port=80
    ```

1. Set required runtime values. Replace `<agent-id>` with your default knowledge base ID (`KB_ID`):

    ```bash
    kubectl -n arc-rag set env deployment/chat-ui-frontend \
      VITE_API_MODE=agents \
      VITE_API_URL=/runtime \
      VITE_AGENT_ID=<agent-id> \
      VITE_BASE_PATH=/chat
    ```

    If you host at `/`, omit `VITE_BASE_PATH`.

1. Create the chat UI service:

    ```bash
    kubectl -n arc-rag expose deployment chat-ui-frontend --name=chat-ui-frontend --port=80 --target-port=80
    ```

1. Configure ingress routing for chat UI and runtime. Replace `<your-chat-ui-domain>` with your chat UI host name. If an ingress already exists for your environment, update that ingress instead of creating a new one.

    ```bash
    kubectl -n arc-rag create ingress arc-rag-ingress \
      --rule="<your-chat-ui-domain>/=chat-ui-frontend:80" \
      --rule="<your-chat-ui-domain>/runtime=agents-runtime:8081"
    ```

1. For connected deployments, set Microsoft Entra values. Replace `<entra-client-id>` and `<entra-tenant-id>` with your app registration values:

    ```bash
    kubectl -n arc-rag set env deployment/chat-ui-frontend \
      VITE_AUTH_CLIENT_ID=<entra-client-id> \
      VITE_AUTH_TENANT_ID=<entra-tenant-id>
    ```

1. For disconnected or air-gapped deployments, remove the Microsoft Entra values:

    ```bash
    kubectl -n arc-rag set env deployment/chat-ui-frontend VITE_AUTH_CLIENT_ID- VITE_AUTH_TENANT_ID-
    ```

#### [Kubernetes manifest](#tab/kubernetes-manifest)

Use one manifest file to keep runtime values, identity settings, and routing in one place.

1. Create `chat-ui-manifest.yaml`.
1. Add the following content and replace placeholder values.

   ```yaml
   # Chat UI deployment package (single file)
   # Includes Deployment + Service + Ingress.
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: chat-ui-frontend
     namespace: arc-rag
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: chat-ui-frontend
     template:
       metadata:
         labels:
           app: chat-ui-frontend
       spec:
         containers:
           - name: chat-ui-frontend
             image: <chat-ui-image>:<tag>
             imagePullPolicy: IfNotPresent
             ports:
               - containerPort: 80
   
             # Health endpoint for liveness/readiness checks
             livenessProbe:
               httpGet:
                 path: /health
                 port: 80
               initialDelaySeconds: 5
               periodSeconds: 30
             readinessProbe:
               httpGet:
                 path: /health
                 port: 80
               initialDelaySeconds: 3
               periodSeconds: 10
   
             env:
               # Required for Agentic RAG mode
               - name: VITE_API_MODE
                 value: "agents"
   
               # Required runtime path to agents-runtime through ingress
               - name: VITE_API_URL
                 value: "/runtime"
   
               # Required agent identifier from your setup flow
               - name: VITE_AGENT_ID
                 value: "<agent-id>"
   
               # Optional: set only when chat UI is hosted on a subpath such as /chat
               # Omit this setting when hosting at /
               - name: VITE_BASE_PATH
                 value: "/chat"
   
               # Connected deployments only (Microsoft Entra ID)
               # Omit both values for disconnected or air-gapped deployments
               - name: VITE_AUTH_CLIENT_ID
                 value: "<entra-client-id>"
               - name: VITE_AUTH_TENANT_ID
                 value: "<entra-tenant-id>"
   
 
   apiVersion: v1
   kind: Service
   metadata:
     name: chat-ui-frontend
     namespace: arc-rag
   spec:
     selector:
       app: chat-ui-frontend
     ports:
       - port: 80
         targetPort: 80
         protocol: TCP
   

   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: arc-rag-ingress
     namespace: arc-rag
   spec:
     rules:
       - host: <your-chat-ui-domain>
         http:
           paths:
             # Chat UI route
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: chat-ui-frontend
                   port:
                     number: 80
   
             # Runtime route used by the chat UI
             - path: /runtime
               pathType: Prefix
               backend:
                 service:
                   name: agents-runtime
                   port:
                     number: 8081
   ```

1. Apply the manifest.

   ```bash
   kubectl apply -f chat-ui-manifest.yaml
   ```

---

## Validate deployment

Run the following checks:

1. Confirm the chat UI deployment is ready:

   ```bash
   kubectl -n arc-rag rollout status deployment/chat-ui-frontend
   kubectl -n arc-rag get deployment chat-ui-frontend
   kubectl -n arc-rag get service chat-ui-frontend
   kubectl -n arc-rag get ingress arc-rag-ingress
   ```

1. Check the health endpoint:

   ```bash
   curl -i https://<your-chat-ui-domain>/health
   ```

   Confirm the response includes `200 OK`.

1. Open chat UI in a browser:

   - If you host at the root path, go to `https://<your-chat-ui-domain>/`.
   - If you host on a subpath, go to `https://<your-chat-ui-domain>/chat`.
   - For connected deployments, sign in with a user who has the required role assignment. For more information, see [Test the chat solution for Agents and Tools with Foundry Local](test-end-user-app.md).

1. Confirm runtime connectivity:

   - In the browser, open developer tools and select the **Network** tab.
   - Refresh the page.
   - Confirm that requests to `/runtime/threads` succeed.

1. Confirm streaming and conversation behavior:

   - Send a prompt in the chat UI.
   - Confirm the assistant response streams incrementally.
   - Confirm the conversation appears in the sidebar.
   - Confirm the thread title appears after the first response.

## Troubleshoot validation failures

Use these checks to identify and resolve common validation problems.

- If the chat UI loads but no data appears:

  - Run `kubectl -n arc-rag get ingress arc-rag-ingress` and confirm the ingress exposes both the chat UI path and `/runtime`.
  - In the browser, open developer tools and select the **Network** tab. Refresh the page and confirm requests to `/runtime/threads` are sent to the expected host and path.
  - Run `kubectl -n arc-rag set env deployment/chat-ui-frontend --list` and confirm `VITE_API_URL=/runtime`.

- If deep links fail on subpath hosting:

  - Run `kubectl -n arc-rag set env deployment/chat-ui-frontend --list` and confirm `VITE_BASE_PATH` matches your ingress path, such as `/chat`.
  - Open `https://<your-chat-ui-domain>/chat` directly in the browser. If the page loads but refresh or direct navigation fails, update either `VITE_BASE_PATH` or the ingress path so both use the same subpath.

- If sign-in fails in connected mode:

  - Run `kubectl -n arc-rag set env deployment/chat-ui-frontend --list` and confirm `VITE_AUTH_CLIENT_ID` and `VITE_AUTH_TENANT_ID` match your Microsoft Entra app registration.
  - Verify that the app registration includes the exact redirect URI used by your deployment host and path.
  - Retry sign-in with a user who has the required role assignment. For more information, see [Test the chat solution for Agents and Tools with Foundry Local](test-end-user-app.md).

## Related content

- [Chat UI in Agents and Tools in Foundry Local](chat-experience.md)
- [Chat UI configuration and API reference](chat-experience-reference.md)
- [Deploy Agents and Tools with Foundry Local](deploy.md)
- [Quickstart: Query your data with Agents and Tools with Foundry Local](quickstart-create-agent.md)
- [Configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md)