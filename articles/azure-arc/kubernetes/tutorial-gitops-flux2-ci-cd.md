---
title: "Tutorial: Implement CI/CD with GitOps (Flux v2)"
description: "This tutorial walks through setting up a CI/CD solution using GitOps (Flux v2) in Azure Arc-enabled Kubernetes or Azure Kubernetes Service clusters."
author: eedorenko
ms.author: iefedore
ms.topic: tutorial
ms.date: 08/25/2025
ms.custom: template-tutorial, devx-track-azurecli
# Customer intent: "As a DevOps engineer, I want to implement a CI/CD workflow using GitOps with Flux v2 in Azure Kubernetes Service, so that I can automate the deployment and management of applications across development and staging environments."
---
# Tutorial: Implement CI/CD with GitOps (Flux v2)

In this tutorial, you set up a CI/CD solution using [GitOps with Flux v2](conceptual-gitops-flux2.md) and Azure Arc-enabled Kubernetes or Azure Kubernetes Service (AKS) clusters. Using the sample Azure Vote app, you can:

> [!div class="checklist"]
> * Connect your application and GitOps repositories to Azure Devops (Azure Repos) or GitHub.
> * Implement CI/CD flow with either Azure Pipelines or GitHub.
> * Connect your Azure Container Registry to Azure DevOps and Kubernetes.
> * Create environment variable groups or secrets.
> * Deploy the `dev` and `stage` environments.
> * Test the application environments.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites

* Complete the [previous tutorial](./tutorial-use-gitops-flux2.md) to learn how to deploy GitOps for your CI/CD environment.
* Understand the [benefits and architecture](./conceptual-gitops-flux2.md) of this feature.
* Verify you have:
  * A [connected Azure Arc-enabled Kubernetes cluster](./quickstart-connect-cluster.md#connect-an-existing-kubernetes-cluster) named **arc-cicd-cluster**.
  * A connected Azure Container Registry with either [AKS integration](/azure/aks/cluster-container-registry-integration) or [non-AKS cluster authentication](../../container-registry/container-registry-auth-kubernetes.md).
* Install the latest versions of these Azure Arc-enabled Kubernetes and Kubernetes Configuration CLI extensions:

  ```azurecli
  az extension add --name connectedk8s
  az extension add --name k8s-configuration
  ```

* Or to update these extensions to the latest version, run the following commands:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-configuration
    ```

### Connect Azure Container Registry to Kubernetes

Enable your Kubernetes cluster to pull images from your Azure Container Registry. If it's private, authentication is required.

#### Connect Azure Container Registry to existing AKS clusters

Integrate an existing Azure Container Registry with existing AKS clusters using the following command:

```azurecli
az aks update -n arc-cicd-cluster -g myResourceGroup --attach-acr arc-demo-acr
```

#### Create an image pull secret

To connect non-AKS and local clusters to your Azure Container Registry, create an image pull secret. Kubernetes uses image pull secrets to store information needed to authenticate your registry.

Create an image pull secret with the following `kubectl` command. Repeat for both the `dev` and `stage` namespaces.

```console
kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```

To avoid having to set an imagePullSecret for every Pod, consider adding the imagePullSecret to the Service account in the `dev` and `stage` namespaces. For more information, see the [Kubernetes tutorial](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account).

Depending on the CI/CD orchestrator you prefer, you can proceed with instructions either for Azure DevOps or for GitHub.

## Implement CI/CD with Azure DevOps

This tutorial assumes familiarity with Azure DevOps, Azure Repos and Pipelines, and Azure CLI.

Make sure to complete the following steps first:

* Sign into [Azure DevOps Services](https://dev.azure.com/).
* Verify you have "Build Admin" and "Project Admin" permissions for [Azure Repos](/azure/devops/repos/get-started/what-is-repos) and [Azure Pipelines](/azure/devops/pipelines/get-started/pipelines-get-started).

### Import application and GitOps repositories into Azure Repos

Import an [application repository](conceptual-gitops-ci-cd.md#application-code-repository) and a [GitOps repository](conceptual-gitops-ci-cd.md#gitops-repository) into Azure Repos. For this tutorial, use the following example repositories:

* **arc-cicd-demo-src** application repository
  * URL: https://github.com/Azure/arc-cicd-demo-src
  * Contains the example Azure Vote App to deploy using GitOps.
  * Import the repository with name `arc-cicd-demo-src`

* **arc-cicd-demo-gitops** GitOps repository
  * URL: https://github.com/Azure/arc-cicd-demo-gitops
  * Works as a base for your cluster resources that house the Azure Vote App.
  * Import the repository with name `arc-cicd-demo-gitops`

Learn more about [importing Git repositories](/azure/devops/repos/git/import-git-repository).

>[!NOTE]
> Importing and using two separate repositories for application and GitOps repositories can improve security and simplicity. The application and GitOps repositories' permissions and visibility can be tuned individually.
> For example, the cluster administrator may not find the changes in application code relevant to the desired state of the cluster. Conversely, an application developer doesn't need to know the specific parameters for each environment - a set of test values that provide coverage for the parameters may be sufficient.

### Connect the GitOps repository

To continuously deploy your app, connect the application repository to your cluster using GitOps. Your **arc-cicd-demo-gitops** GitOps repository contains the basic resources to get your app up and running on your **arc-cicd-cluster** cluster.

The initial GitOps repository contains only a [manifest](https://github.com/Azure/arc-cicd-demo-gitops/blob/master/arc-cicd-cluster/manifests/namespaces.yml) that creates the **dev** and **stage** namespaces corresponding to the deployment environments.

The GitOps connection that you create automatically syncs the manifests in the manifest directory and updates the cluster state.

The CI/CD workflow populates the manifest directory with extra manifests to deploy the app.

1. [Create a new GitOps connection](./tutorial-use-gitops-flux2.md) to your newly imported **arc-cicd-demo-gitops** repository in Azure Repos.

   ```azurecli
   az k8s-configuration flux create \
      --name cluster-config \
      --cluster-name arc-cicd-cluster \
      --namespace flux-system \
      --resource-group myResourceGroup \
      -u https://dev.azure.com/<Your organization>/<Your project>/_git/arc-cicd-demo-gitops \
      --https-user <Azure Repos username> \
      --https-key <Azure Repos PAT token> \
      --scope cluster \
      --cluster-type connectedClusters \
      --branch master \
      --kustomization name=cluster-config prune=true path=arc-cicd-cluster/manifests
   ```

   > [!TIP]
   > For an AKS cluster (rather than an Arc-enabled cluster), use `-cluster-type managedClusters`.

1. Check the state of the deployment in Azure portal.
   * If successful, you see both `dev` and `stage` namespaces created in your cluster.
   * You can also confirm that on the Azure portal page of your cluster, a configuration `cluster-config` is created on the `GitOps` tab.
      
### Import the CI/CD pipelines

Now that you synced a GitOps connection, you need to import the CI/CD pipelines that create the manifests.

The application repository contains a `.pipeline` folder with pipelines used for PRs, CI, and CD. Import and rename the three pipelines provided in the sample repository:

| Pipeline file name | Description |
| ------------- | ------------- |
| [`.pipelines/az-vote-pr-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-pr-pipeline.yaml) | The application PR pipeline, named **arc-cicd-demo-src PR** |
| [`.pipelines/az-vote-ci-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-ci-pipeline.yaml) | The application CI pipeline, named **arc-cicd-demo-src CI** |
| [`.pipelines/az-vote-cd-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-cd-pipeline.yaml) | The application CD pipeline, named **arc-cicd-demo-src CD** |

### Connect Azure Container Registry to Azure DevOps

During the CI process, you deploy your application containers to a registry. Start by creating an [Azure service connection](/azure/devops/pipelines/library/service-endpoints):

1. In Azure DevOps, open the **Service connections** page from the project settings page. In TFS, open the **Services** page from the **settings** icon in the top menu bar.
2. Choose **+ New service connection** and select the type of service connection you need.
3. Fill in the parameters for the service connection. For this tutorial:
   * Name the service connection **arc-demo-acr**.
   * Select **myResourceGroup** as the resource group.
4. Select the **Grant access permission to all pipelines**.
   * This option authorizes YAML pipeline files for service connections.
5. Choose **Save** to create the connection.

### Configure PR service connection

The CD pipeline manipulates pull requests (PRs) in the GitOps repository, which requires a service connection. To configure this connection:

1. In Azure DevOps, open the **Service connections** page from the project settings page. In TFS, open the **Services** page from the **settings** icon in the top menu bar.
2. Choose **+ New service connection** and select `Generic` type.
3. Fill in the parameters for the service connection. For this tutorial:
   * Server URL `https://dev.azure.com/<Your organization>/<Your project>/_apis/git/repositories/arc-cicd-demo-gitops`
   * Leave Username and Password blank.
   * Name the service connection **azdo-pr-connection**.
4. Select the **Grant access permission to all pipelines**.
   * This option authorizes YAML pipeline files for service connections.
5. Choose **Save** to create the connection.

### Install GitOps Connector

1. Add GitOps Connector repository to Helm repositories:

   ```console
      helm repo add gitops-connector https://azure.github.io/gitops-connector/
   ```

1. Install the connector to the cluster:

   ```console
      helm upgrade -i gitops-connector gitops-connector/gitops-connector \
         --namespace flux-system \
         --set gitRepositoryType=AZDO \
         --set ciCdOrchestratorType=AZDO \
         --set gitOpsOperatorType=FLUX \
         --set azdoGitOpsRepoName=arc-cicd-demo-gitops \
         --set azdoOrgUrl=https://dev.azure.com/<Your organization>/<Your project> \
         --set gitOpsAppURL=https://dev.azure.com/<Your organization>/<Your project>/_git/arc-cicd-demo-gitops \
         --set orchestratorPAT=<Azure Repos PAT token>
   ```

   > [!NOTE]
   > `Azure Repos PAT token` should have `Build: Read & execute` and `Code: Full` permissions.

1. Configure Flux to send notifications to GitOps connector:

   ```console
   cat <<EOF | kubectl apply -f -
   apiVersion: notification.toolkit.fluxcd.io/v1beta2
   kind: Alert
   metadata:
     name: gitops-connector
     namespace: flux-system
   spec:
     eventSeverity: info
     eventSources:
     - kind: GitRepository
       name: cluster-config
     - kind: Kustomization
       name: cluster-config-cluster-config 
     providerRef:
       name: gitops-connector
   ---
   apiVersion: notification.toolkit.fluxcd.io/v1beta2
   kind: Provider
   metadata:
     name: gitops-connector
     namespace: flux-system
   spec:
     type: generic
     address: http://gitops-connector:8080/gitopsphase
   EOF
   ```

For details about installation, see the [GitOps Connector](https://github.com/microsoft/gitops-connector#installation) repository.

### Create environment variable groups

#### App repository variable group

Create a [variable group](/azure/devops/pipelines/library/variable-groups) named **az-vote-app-dev**. Set the following values:

| Variable | Value |
| -------- | ----- |
| `AZURE_SUBSCRIPTION` | (your Azure Service Connection, which should be **arc-demo-acr** from earlier in the tutorial) |
| `AZ_ACR_NAME` | Azure ACR name, for example arc-demo-acr |
| `ENVIRONMENT_NAME` | Dev |
| `MANIFESTS_BRANCH` | `master` |
| `MANIFESTS_REPO` | `arc-cicd-demo-gitops` |
| `ORGANIZATION_NAME` | Name of Azure DevOps organization |
| `PROJECT_NAME` | Name of GitOps project in Azure DevOps |
| `REPO_URL` | Full URL for GitOps repository |
| `SRC_FOLDER` | `azure-vote` |
| `TARGET_CLUSTER` | `arc-cicd-cluster` |
| `TARGET_NAMESPACE` | `dev` |
| `VOTE_APP_TITLE` | Voting Application |
| `AKS_RESOURCE_GROUP` | AKS Resource group. Needed for automated testing. |
| `AKS_NAME` | AKS Name. Needed for automated testing. |

#### Stage environment variable group

1. Clone the **az-vote-app-dev** variable group.
1. Change the name to **az-vote-app-stage**.
1. Ensure the following values for the corresponding variables:

   | Variable | Value |
   | -------- | ----- |
   | `ENVIRONMENT_NAME` | Stage |
   | `TARGET_NAMESPACE` | `stage` |

You're now ready to deploy to the `dev` and `stage` environments.

#### Create environments

In your Azure DevOps project, create `Dev` and `Stage` environments. For details, see [Create and target environments](/azure/devops/pipelines/process/environments).

### Give more permissions to the build service

The CD pipeline uses the security token of the running build to authenticate to the GitOps repository. More permissions are needed for the pipeline to create a new branch, push changes, and create PRs. To enable these permissions:

1. In Azure DevOps, open **Project settings**.
1. Under **Repositories**, select **Repos**.
1. Select **Security**.
1. Find `<Project Name> Build Service (<Organization Name>)` and `Project Collection Build Service (<Organization Name>)` (use search if you don't see them), and allow **Contribute**, **Contribute to pull requests**, and **Create branch**.
1. Under **Pipelines**, select **Settings**.
1. Turn off the **Protect access to repositories in YAML pipelines** option.

For more information, see [Grant version control permissions to the build service](/azure/devops/pipelines/scripts/git-commands?preserve-view=true&tabs=yaml&view=azure-devops#version-control) and [Manage build service account permissions](/azure/devops/pipelines/process/access-tokens?preserve-view=true&tabs=yaml&view=azure-devops#manage-build-service-account-permissions).

### Deploy the dev environment for the first time

With the CI and CD pipelines created, run the CI pipeline to deploy the app for the first time.

#### CI pipeline

During the initial CI pipeline run, if you see a resource authorization error in reading the service connection name, do the following:

1. Verify the variable being accessed is AZURE_SUBSCRIPTION.
1. Authorize the use.
1. Rerun the pipeline.

The CI pipeline:

* Ensures the application change passes all automated quality checks for deployment.
* Does any extra validation that couldn't be completed in the PR pipeline. Specific to GitOps, the pipeline also publishes the artifacts for the commit that will be deployed by the CD pipeline.
* Verifies the Docker image has changed and the new image is pushed.

#### CD pipeline

During the initial CD pipeline run, you need to give the pipeline access to the GitOps repository. Select **View** when prompted that the pipeline needs permission to access a resource. Then, select **Permit** to grant permission to use the GitOps repository for the current and future runs of the pipeline.

The successful CI pipeline run triggers the CD pipeline to complete the deployment process. You deploy to each environment incrementally.

> [!TIP]
> If the CD pipeline does not automatically trigger:
>
> 1. Verify the name matches the branch trigger in [`.pipelines/az-vote-cd-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-cd-pipeline.yaml)
>    * It should be `arc-cicd-demo-src CI`.
> 1. Rerun the CI pipeline.

Once the template and manifest changes to the GitOps repository are generated, the CD pipeline creates a commit, pushes it, and creates a PR for approval.

1. Find the PR created by the pipeline to the GitOps repository.
1. Verify the changes to the GitOps repository. You should see:
   * High-level Helm template changes.
   * Low-level Kubernetes manifests that show the underlying changes to the desired state. Flux deploys these manifests.
1. If everything looks good, approve and complete the PR.
1. After a few minutes, Flux picks up the change and starts the deployment.
1. Monitor the `git commit` status on the **Commit history** tab. Once it is `succeeded`, the CD pipeline starts automated testing.
1. Forward the port locally using `kubectl` and ensure the app works correctly using:

    ```console
   kubectl port-forward -n dev svc/azure-vote-front 8080:80
    ```

1. View the Azure Vote app in your browser at `http://localhost:8080/`.
1. Vote for your favorites and get ready to make some changes to the app.

### Set up environment approvals

Upon app deployment, you can make changes to the code or templates, but you can also unintentionally put the cluster into a bad state.

If the dev environment reveals a break after deployment, enabling environment approvals helps keep the problem from later environments.

1. In your Azure DevOps project, go to the environment that needs to be protected.
1. Navigate to **Approvals and Checks** for the resource.
1. Select **Create**.
1. Provide the approvers and an optional message.
1. Select **Create** again to complete the addition of the manual approval check.

For more information, see [Define approval and checks](/azure/devops/pipelines/process/approvals).

Next time the CD pipeline runs, the pipeline will pause after the GitOps PR creation. Verify that the change is properly synced and passes basic functionality. Approve the check from the pipeline to let the change flow to the next environment.

### Make an application change

With this baseline set of templates and manifests representing the state on the cluster, you make a small change to the app.

1. In the **arc-cicd-demo-src** repository, edit [`azure-vote/src/azure-vote-front/config_file.cfg`](https://github.com/Azure/arc-cicd-demo-src/blob/master/azure-vote/src/azure-vote-front/config_file.cfg) file.

2. Since "Cats vs Dogs" isn't getting enough votes, change it to "Tabs vs Spaces" to drive up the vote count.

3. Commit the change in a new branch, push it, and create a pull request. This sequence of steps is the typical developer flow that starts the CI/CD lifecycle.

### PR validation pipeline

The PR pipeline is the first line of defense against a faulty change. Usual application code quality checks include linting and static analysis. From a GitOps perspective, you also need to assure the same quality for the resulting infrastructure to be deployed.

The application's Dockerfile and Helm charts can use linting in a similar way to the application.

Errors found during linting range from incorrectly formatted YAML files, to best practice suggestions, such as setting CPU and Memory limits for your application.

> [!NOTE]
> To get the best coverage from Helm linting in a real application, substitute values that are reasonably similar to values that would be used in a real environment.

Errors found during pipeline execution appear in the test results section of the run. From here, you can:

* Track the useful statistics on the error types.
* Find the first commit on which they were detected.
* Stack trace style links to the code sections that caused the error.

The pipeline run finishes, confirming the quality of the application code and the template that deploys it. You can now approve and complete the PR. The CI runs again, regenerating the templates and manifests, before triggering the CD pipeline.

> [!TIP]
> In a real environment, be sure to set branch policies to ensure the PR passes your quality checks. For more information, see [Branch policies and settings](/azure/devops/repos/git/branch-policies).

### CD process approvals

A successful CI pipeline run triggers the CD pipeline to complete the deployment process. This time, the pipeline requires you to approve each deployment environment.

1. Approve the deployment to the `dev` environment.
1. Once the template and manifest changes to the GitOps repository have been generated, the CD pipeline creates a commit, pushes it, and creates a PR for approval.
1. Verify the changes to the GitOps repository. You should see:
   * High-level Helm template changes.
   * Low-level Kubernetes manifests that show the underlying changes to the desired state.
1. If everything looks good, approve and complete the PR.
1. Wait for the deployment to complete.
1. As a basic smoke test, navigate to the application page and verify the voting app now displays **Tabs vs Spaces**.
   * Forward the port locally using `kubectl` and ensure the app works correctly using:
   `kubectl port-forward -n dev svc/azure-vote-front 8080:80`
   * View the Azure Vote app in your browser at `http://localhost:8080/` and verify the voting choices have changed to Tabs vs Spaces.
1. Repeat steps 1-7 for the `stage` environment.

The deployment is now complete.

For a detailed overview of all the steps and techniques implemented in the CI/CD workflows used in this tutorial, see the [Azure DevOps GitOps Flow diagram](https://github.com/Azure/arc-cicd-demo-src/blob/master/docs/azdo-gitops.md).

## Implement CI/CD with GitHub

This tutorial assumes familiarity with GitHub, GitHub Actions.

### Fork application and GitOps repositories

Fork an [application repository](./conceptual-gitops-ci-cd.md#application-repo) and a [GitOps repository](./conceptual-gitops-ci-cd.md#gitops-repo). For this tutorial, use the following example repositories:

* **arc-cicd-demo-src** application repository
  * URL: https://github.com/Azure/arc-cicd-demo-src
  * Contains the example Azure Vote App to deploy using GitOps.

* **arc-cicd-demo-gitops** GitOps repository
  * URL: https://github.com/Azure/arc-cicd-demo-gitops
  * Works as a base for your cluster resources that house the Azure Vote App.

### Connect the GitOps repository

To continuously deploy your app, connect the application repository to your cluster using GitOps. Your **arc-cicd-demo-gitops** GitOps repository contains the basic resources to get your app up and running on your **arc-cicd-cluster** cluster.

The initial GitOps repository contains only a [manifest](https://github.com/Azure/arc-cicd-demo-gitops/blob/master/arc-cicd-cluster/manifests/namespaces.yml) that creates the **dev** and **stage** namespaces corresponding to the deployment environments.

The GitOps connection that you create automatically:

* Sync the manifests in the manifest directory.
* Update the cluster state.

The CI/CD workflow populates the manifest directory with extra manifests to deploy the app.

1. [Create a new GitOps connection](./tutorial-use-gitops-flux2.md) to your newly forked **arc-cicd-demo-gitops** repository in GitHub.

   ```azurecli
   az k8s-configuration flux create \
      --name cluster-config \
      --cluster-name arc-cicd-cluster \
      --namespace cluster-config \
      --resource-group myResourceGroup \
      -u  https://github.com/<Your organization>/arc-cicd-demo-gitops.git \
      --https-user <Azure Repos username> \
      --https-key <Azure Repos PAT token> \
      --scope cluster \
      --cluster-type connectedClusters \
      --branch master \
      --kustomization name=cluster-config prune=true path=arc-cicd-cluster/manifests
   ```

1. Check the state of the deployment in Azure portal.
   * If successful, you see both `dev` and `stage` namespaces created in your cluster.

### Install GitOps Connector

1. Add GitOps Connector repository to Helm repositories:

   ```console
      helm repo add gitops-connector https://azure.github.io/gitops-connector/
   ```

1. Install the connector to the cluster:

   ```console
      helm upgrade -i gitops-connector gitops-connector/gitops-connector \
         --namespace flux-system \
         --set gitRepositoryType=GITHUB \
         --set ciCdOrchestratorType=GITHUB \
         --set gitOpsOperatorType=FLUX \
         --set gitHubGitOpsRepoName=arc-cicd-demo-src \
         --set gitHubGitOpsManifestsRepoName=arc-cicd-demo-gitops \
         --set gitHubOrgUrl=https://api.github.com/repos/<Your organization> \
         --set gitOpsAppURL=https://github.com/<Your organization>/arc-cicd-demo-gitops/commit \
         --set orchestratorPAT=<GitHub PAT token>
   ```

1. Configure Flux to send notifications to GitOps connector:

   ```console
   cat <<EOF | kubectl apply -f -
   apiVersion: notification.toolkit.fluxcd.io/v1beta2
   kind: Alert
   metadata:
     name: gitops-connector
     namespace: flux-system
   spec:
     eventSeverity: info
     eventSources:
     - kind: GitRepository
       name: cluster-config
     - kind: Kustomization
       name: cluster-config-cluster-config
     providerRef:
       name: gitops-connector
   ---
   apiVersion: notification.toolkit.fluxcd.io/v1beta2
   kind: Provider
   metadata:
     name: gitops-connector
     namespace: flux-system
   spec:
     type: generic
     address: http://gitops-connector:8080/gitopsphase
   EOF
   ```

For the details on installation, refer to the [GitOps Connector](https://github.com/microsoft/gitops-connector#installation) repository.

### Create GitHub secrets

The next step is to create GitHub repository and environment secrets.

#### Create GitHub repository secrets

Use the following values for your GitHub repository secrets:

| Secret | Value |
| -------- | ----- |
| `AZURE_CREDENTIALS `| Credentials for Azure in the following format {"clientId":"GUID","clientSecret":"GUID","subscriptionId":"GUID","tenantId":"GUID"} |
| `AZ_ACR_NAME` | Azure ACR name, for example arc-demo-acr |
| `MANIFESTS_BRANCH` | `master` |
| `MANIFESTS_FOLDER` | `arc-cicd-cluster` |
| `MANIFESTS_REPO` | `https://github.com/your-organization/arc-cicd-demo-gitops` |
| `VOTE_APP_TITLE` | Voting Application |
| `AKS_RESOURCE_GROUP` | AKS Resource group. Needed for automated testing. |
| `AKS_NAME` | AKS Name. Needed for automated testing. |
| `PAT` | GitHub PAT token with the permission to PR to the GitOps repository |

#### Create GitHub environment secrets

1. Create `az-vote-app-dev` environment with the following secrets:

| Secret | Value |
| -------- | ----- |
| `ENVIRONMENT_NAME` | Dev |
| `TARGET_NAMESPACE` | `dev` |

1. Create `az-vote-app-stage` environment with the following secrets:

| Secret | Value |
| -------- | ----- |
| `ENVIRONMENT_NAME` | Stage |
| `TARGET_NAMESPACE` | `stage` |

You're now ready to deploy to the `dev` and `stage` environments.

#### CI/CD Dev workflow

To start the CI/CD Dev workflow, change the source code. In the application repository, update values in `.azure-vote/src/azure-vote-front/config_file.cfg` file and push the changes to the repository.

The CI/CD Dev workflow:

* Ensures the application change passes all automated quality checks for deployment.
* Does any extra validation that couldn't be completed in the PR pipeline.
* Verifies the Docker image has changed and the new image is pushed.
* Publishes the artifacts (Docker image tags, Manifest templates, Utils) that are used by the following CD stages.
* Deploys the application to Dev environment.
  * Generates manifests to the GitOps repository.
  * Creates a PR to the GitOps repository for approval.

Once these steps are completed:

1. Find the PR created by the pipeline to the GitOps repository.
1. Verify the changes to the GitOps repository. You should see:
   * High-level Helm template changes.
   * Low-level Kubernetes manifests that show the underlying changes to the desired state. Flux deploys these manifests.
1. If everything looks good, approve and complete the PR.
1. After a few minutes, Flux picks up the change and starts the deployment.
1. Monitor the `git commit` status on the Commit history tab. Once it is `succeeded`, the `CD Stage` workflow starts.
1. Forward the port locally using `kubectl` and ensure the app works correctly using:

    ```console
   kubectl port-forward -n dev svc/azure-vote-front 8080:80
    ```

1. View the Azure Vote app in your browser at `http://localhost:8080/`.
1. Vote for your favorites and get ready to make some changes to the app.

#### CD Stage workflow

The CD Stage workflow starts automatically once Flux successfully deploys the application to dev environment and notifies GitHub actions via GitOps Connector.

The CD Stage workflow:

* Runs application smoke tests against Dev environment
* Deploys the application to Stage environment.
  * Generates manifests to the GitOps repository
  * Creates a PR to the GitOps repository for approval

Once the manifests PR to the Stage environment is merged and Flux successfully applies all the changes, the Git commit status is updated in the GitOps repository. The deployment is now complete.

For a detailed overview of all the steps and techniques implemented in the CI/CD workflows used in this tutorial, see the [GitHub GitOps Flow diagram](https://github.com/Azure/arc-cicd-demo-src/blob/master/docs/azdo-gitops-githubfluxv2.md).

## Clean up resources

If you're not going to continue to use this application, delete any resources with the following steps:

1. Delete the Azure Arc GitOps configuration connection:

   ```azurecli
   az k8s-configuration flux delete \
         --name cluster-config \
         --cluster-name arc-cicd-cluster \
         --resource-group myResourceGroup \
         -t connectedClusters --yes
   ```

1. Delete GitOps Connector:

   ```console
   helm uninstall gitops-connector -n flux-system
   kubectl delete alerts.notification.toolkit.fluxcd.io gitops-connector -n flux-system
   kubectl delete providers.notification.toolkit.fluxcd.io  gitops-connector -n flux-system
   ```

## Next steps

In this tutorial, you set up a full CI/CD workflow that implements DevOps from application development through deployment. Changes to the app automatically trigger validation and deployment, gated by manual approvals.

Advance to our conceptual article to learn more about GitOps and configurations with Azure Arc-enabled Kubernetes.

> [!div class="nextstepaction"]
> [Concept: CD process with GitOps](https://github.com/microsoft/kalypso/blob/main/docs/cd-concept.md)
> [Sample implementation: Explore CI/CD flow with GitOps](https://github.com/microsoft/kalypso/blob/main/cicd/tutorial/cicd-tutorial.md)
