---
author: cwatson-cat
ms.author: cwatson
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ms.topic: include
ms.date: 11/08/2026
ai-usage: ai-assisted
---

### Set up your environment

The following examples use the cluster address and an access token from Microsoft Entra ID. Replace `<cluster-domain>` with the domain of your deployment, such as `arcrag.contoso.com`, and `<app-registration-client-id>` with the application (client) ID of the app registration for Agents and Tools with Foundry Local.

# [curl](#tab/curl)

Sign in with the Azure CLI, and save the address and a token in variables:

```bash
az login --tenant <your-tenant-id> --output none

BASE="https://<cluster-domain>"
TOKEN=$(az account get-access-token \
  --resource "api://<app-registration-client-id>" \
  --query accessToken -o tsv)
```

# [Python](#tab/python)

Install the packages:

```bash
pip install azure-identity requests openai
```

Sign in with `az login`, and then create the clients. Use `requests` for management APIs, and the OpenAI Python library for the Responses and Conversations APIs:

```python
import requests
from azure.identity import AzureCliCredential
from openai import OpenAI

BASE = "https://<cluster-domain>"
scope = "api://<app-registration-client-id>/.default"
token = AzureCliCredential().get_token(scope).token

api = requests.Session()
api.headers["Authorization"] = f"Bearer {token}"

client = OpenAI(base_url=f"{BASE}/openai/v1", api_key=token)
```

Tokens expire after about an hour. Get a new token when a request returns `401 Unauthorized`.

---
