---
ms.service: azure-arc
ms.subservice: azure-arc-container-storage
ms.topic: include
ms.date: 09/26/2025
author: asergaz
ms.author: sergaz
---

If you wish to use Workload Identity with Azure Container Storage Enabled by Azure Arc, follow the instructions in [Configure Workload Identity for Cloud subvolumes](../howto-configure-workload-identity.md).

#### [Azure portal](#tab/portal)

#### Azure portal

1. Navigate to your Arc-enabled cluster.
1. Select **Extensions**.
1. Select your Azure Container Storage enabled by Azure Arc extension.
1. Note the Principal ID under **Cluster Extension Details**.
  
#### [Azure CLI](#tab/cli)

#### Azure CLI

In Azure CLI, enter your values for the exports (`CLUSTER_NAME`, `RESOURCE_GROUP`) and run the following command:

```bash
export CLUSTER_NAME = <your-cluster-name-here>
export RESOURCE_GROUP = <your-resource-group-here>
export EXTENSION_TYPE=${1:-"microsoft.arc.containerstorage"}
az k8s-extension list --cluster-name ${CLUSTER_NAME} --resource-group ${RESOURCE_GROUP} --cluster-type connectedClusters | jq --arg extType ${EXTENSION_TYPE} 'map(select(.extensionType == $extType)) | .[] | .identity.principalId' -r
```

---

### Configure blob storage account for Extension Identity

#### Add Extension Identity permissions to a storage account

##### [Azure portal](#tab/portal)

1. Navigate to storage account in the Azure portal.
1. Select **Access Control (IAM)**.
1. Select **Add+ -> Add role assignment**.
1. Select **Storage Blob Data Owner**, then select **Next**.
1. Select **+Select Members**.
1. To add your principal ID to the **Selected Members:** list, paste the ID and select **+** next to the identity.
1. Click **Select**.
1. To review and assign permissions, select **Next**, then select **Review + Assign**.

##### [Azure CLI](#tab/cli)

In Azure CLI, enter your values for the variables (`STORAGE_ACCOUNT_NAME`, `RESOURCE_GROUP`, `PRINCIPAL_ID`) and run the following command:

1. **Set your storage account variables:**  
   ```sh
   STORAGE_ACCOUNT_NAME=<your-storage-account-name>
   RESOURCE_GROUP=<your-resource-group>
   PRINCIPAL_ID=<your-extension-identity-principal-ID-from-the-previous-section>
   SUBSCRIPTION_ID=<your-subscription-id>
   ```

2. **Assign the `Storage Blob Data Owner` role to your Extension Identity:**  
   ```sh
   az role assignment create --assignee $PRINCIPAL_ID --role "Storage Blob Data Owner" --scope /subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Storage/storageAccounts/$STORAGE_ACCOUNT_NAME
   ```

   This command assigns the `Storage Blob Data Owner` role to the specified identity at the scope of your storage account.

---