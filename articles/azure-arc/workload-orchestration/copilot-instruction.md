Workload and configuration management with Workload Orchestration service 

 

The purpose of this document is to assess the capabilities of the Workload Orchestration service, focusing on Kraft Heinz workload and configuration management use-cases.  

Layout 

Environments: 

Dev/Stage: One AKS for all locations/sites 

Prod: One AKS per location 

Locations are split into rings  

Applications: 

Control-tower: same app for all sites. Standard K8s deployment.  

Overfill. Specific implementation/configuration for each location and line. Spark application. 

All applications use busybox as an image. The focus is on the configuration composition. 

Subscription: Kraft 

GitHub repos: personal private (easy to provision/manage; easy to share with Microsoft engineers) 

Use-cases 

Onboard a new application to the fleet 

Promote a new application change through environments/rings 

Update/add application configuration in an environment 

Update/add platform configuration in an environment 

Promote a specific data pipeline version to a specific location 

Onboard a new cluster 

Deployment Observability 

Goal to identify product’s 

Value 

Frictions / pain-points 

Blockers 

 

Current Capabilities  

Capability 

Comments 

Workload assignment 

Workload assignment is based on capability matching, so that workloads and deployment targets are marked with capabilities/tags and Kalypso Scheduler performs automatic assignment 

Configuration composition 

Platform engineers define a configuration graph  and Kalypso Scheduler composes configuration values for each application assignment based on capability matching 

Namespace as a service 

Platform engineers define a namespace structure for workloads assigned to clusters. Each namespace can include resources necessary for the workload, such as service accounts, ConfigMaps, secret provider classes, as well as resources that enforce limits and quotas. 

Multi-cluster scalability 

Kalypso Scheduler assigns and configures workloads at the cluster type level, rather than at the cluster level. A cluster type represents a group of similar clusters in terms of applications and configurations. Each cluster type might be backed up by any number of clusters that consume Kalypso's output and deploy applications and configurations with GitOps. The Deployment Observability Hub component operates at the cluster level and it stores the deployment state data for each cluster in a database, capable of handling any reasonable number of clusters. 

Promotion flow 

There is an explicit definition of environments. By the means of GitHub action workflows, Kalypso Scheduler enforces global configurations to be promoted across a chain of environments. 

Deployment observability 

Deployment observability hub provides out-of-the-box Grafana dashboards to monitor what workload versions are deployed on what clusters 

Reconciler agnostic 

Kalypso Scheduler generates workload assignments and configuration compositions using templates. Every cluster type may have different reconciler (e.g. FLux, ArgoCD, Ansible, etc.) and the scheduler will use different templates while assigning applications to them. 

Platform as code 

Platform engineers define the deployment platform with CRD objects such as environment, cluster types, workloads, etc. It's very common to keep and maintain these definitions in a Git repository and deliver them to the Kalypso management cluster with GitOps. 

Bootstrapping capability 

Automated way to onboard a new application 

Automated way to onboard a new cluster 

 

Personas 

Most commonly, there are two personas involved in the scenarios considered: the application team and the platform team.  Sometimes, especially at the early stages, these roles are played by the very same people, just wearing different hats. However, as things scale out, this separation of concerns becomes more explicit. 

 

Separation of concerns 

Application behavior on a deployment target is determined by configuration values. However, configuration values are not all the same. These values are provided by different personas at different points in the application lifecycle and have different scopes. Generally, there are application and platform configurations. 

Application configurations provided by the application team are abstracted away from deployment target details. For example, logging level, working mode, new feature flag, etc. In a way, these configurations can be considered as a part of the source code. They are normally provided at the “packaging” time when an application package is being prepared to be deployed across multiple deployment targets. 

Platform configurations are provided by the platform team. They configure the runtime behavior of the same application package on different deployment targets. Those are normally endpoints, paths, subscriptions, key vaults, equipment parameters, etc. 

Both application team and platform team author their work in GitHub repositories. GitHub Actions Workflows process the repositories content and communicate to Workload Orchestration service with az cli commands to deploy application and configurations to the clusters. 

 

 

Application Team 

The application team is responsible for the software development lifecycle (SDLC) of their applications. They're responsible for owning CI/CD pipelines that create container images and Kubernetes manifests and promote deployable artifacts across environments. 

Typically, the application team has no knowledge of the clusters that they are deploying to. They aren't aware of the structure of the multi-cluster environment, platform configurations, or tasks performed by other teams. The application team primarily understands the success of their application rollout as defined by the success of the pipeline stages. 

Key responsibilities of the application team are: 

Develop, build, deploy, test, promote, release, and support their applications. 

Maintain and contribute to source and manifests repositories of their applications. 

Communicate to platform team, requesting configured compute resources for successful SDLC operations 

 

SDLS of each application is run in a system of three GitHub repositories: 

Application Source Code repository contains the application source code along with a Docker file and manifest templates, such as Helm charts. It also contains application configuration schema, configuration template and solution spec. 

Application Config repository contains environment-specific application configuration  values. These configurations are application-centric, such as logging levels, number of replicas, feature flags, localizations, etc. 

Application GitOps repository contains composed application configuration values and projected application manifests, so that the application team can review what is going to be deployed to the cluster. 

 

Platform Team 

The platform team is a shared service, consumed by numerous application teams. The platform team is responsible for managing the clusters that host applications produced by application teams. 

Key responsibilities of the platform team are: 

Assign applications to the clusters across environments (Dev, Stage, Prod) 

Provide platform/infra configurations for the applications on the clusters 

Maintain platform configuration and services on the clusters 

Onboard clusters to the fleet and maintain their distribution across environments 

 

The system of GitHub repositories for the platform team is symmetric to the application repositories setup. There are three repositories: 

Platform Control Plane repository contains Helm Chart templates for the config maps, namespace-as-a-service resources such as, service account, quotas, limits, secret provider class, etc. It also contains application platform config schemas and config templates. 

Platform Config Repository contains platform configuration values.  

Platform GitOps repository contains composed platform configuration values and projected manifests with the platform configmap and namespace-as-a-service resources, so that the platform team can review what is going to be deployed to the cluster from the platform team perspective. 

 

The Platform team deploys platform configurations (configMap) and namespace resources (service account, limits, quotas, etc.) in a form of a separate solution: 

Platform team doesn’t want to interfere in the application lifecycle and they want to keep things separated. 

Platform team wants to know what is required but they don’t want to block application deployment if not all required configs are provided. Application and platform config deployments must be asynchronous. 

New platform configs should be available for the application on the clusters immediately without waiting for the next application deployment cycle. Depending on the application and environment the application may consume these new values right away or not. 

 

 

Use-cases 

1. Onboard control-tower application to the fleet  

An application team bootstrapped control-tower application. They configured application Git repositories and established an application CI/CD pipeline. They want their application to be deployed/promoted to Dev, Stage and Prod. On the dev environment application team defines two deployment targets (two instances/flavors): func-test-dev for functional testing and perf-test-dev for performance testing. 

Persona: Platform Team 

Flow: 

Define platform config schema and platform config template for the control-tower application. 

Create platform config schema with create-schema script 

Decide (manually assign) what clusters are going to host what workload deployment targets (e.g. func-test-dev->dev-aks; perf-test-dev->dev-aks; func-test-stage->stage-aks, etc.) 

For each assigned cluster (dev-aks), for each workload deployment target, hosted on the cluster create a cluster target (`az workload-orchestration target create`). Use create-target script: 

create-target.sh dev-aks dev-aks-ct-func-test-dev ct-func-test-dev '["edge","data-collection","functional-test"]' 

create-target.sh dev-aks dev-aks-ct-perf-test-dev ct-perf-test-dev '["edge","data-collection","performance-test"]' 

Provide application dev team with the created targets, so that they can specify the clusterTargets in the values.yaml in the application config repo. 

Provide platform configuration values for the onboarded workload for each deployment target in the platform config repo 

Use add-workload-to-hub script to add each workload deployment target to the observability hub 

 

(See the instructions in the platform control-plane repo) 

 

Concerns: 

There is no actual scheduling/assigning. Capabilities matching is used only in the workload orchestration UI to show relevant app versions for the line operator. Platform team must manually assign applications to the cluster. It’s a scale issue. 

Wea have to manually create a target (‘az workload-orchestration target create’). It’s a scale issue if we deploy and application to a million clusters. 

The GitHub Action workflow has the “render the universe” problem. A tiny little change to a config value will force the workflow to process every application on every cluster as it is hard for the workflow to detect what has been affected.   

Is there any API/SDK for the service, so that we can implement more sophisticated automation? 

2. Promote a new control-tower application version across environments/rings.  

An application developer introduces a change to the source code (e.g. bugfix/feature), which produces a new application version. This version is promoted by the application team across environments.  

Persona: Application Team 

Flow: 

Application developer submits a change to the source repository. 

CI workflow builds/pushes artifacts (e.g. images) required for deployment.  

CI workflow packages and pushes to OCI a new application helm chart version 

Prepare-pr workflow iterates over all folders in the dev branch of the application config repo, composes configuration values, generates manifests out of application Helm Chart and creates a PR to the application GitOps repo. 

Application team reviews and merges the PR 

Deploy workflow creates a new solution version for each deployment target (functional testing, performance testing), sets composed application configuration values for the solutions, resolves, publishes and installs the solutions on the targets specified in values.yaml. 

If the deployment is successful (no errors from workload orchestration service),  bullets 4-6 are repeated for the next environment/ring. 

Concerns: 

Yaml.K8s content type is not supported? Only Helm.v3?  

There is no drift detection and reconciliation like with true GitOps. If I delete deployment on the cluster it is not recovered. It’s an old issue of Symphony. 

Solution template name length <=24. It’s pretty low. 

Capabilities are defined globally at the context level, so the app developers don’t have flexibility to define their own capabilities. Every time we add a new capability we have to recreate/update the context. It would be great if application (custom capabilities) could be mapped with some rules/policies to the platform (context) capabilities 

Versions pattern (X.Y.Z) does not follow the SemVer format in terms of extensions. E.g. 1.0.1-alpha, 1.0.1-alpha.1, 1.0.2, etc. 

App developers can’t use `az workload-orchestration configuration set` to configure global application level (platform/location/cluster independent) config values until there is at least one defined target (assigned cluster) with matching capabilities. The latter is done by the platform team. Which means platform team block application SDLC. 

Is there a way to start deployment to multiple clusters in parallel asynchronously, instead of one-by-one? Can we then ask the service if the whole deployment was successful?  

 

3. Update a control-tower application configuration value in Dev environment. 

An application developer needs to decrease logging level from Debug to Info for the performance testing application instance in Dev environment. 

Persona: Application Team 

Flow: 

An application developer submits a PR to reduce the logging level in the application config repo 

Once the PR is merged, the Prepare-pr workflow iterates over all folders in the dev branch of the application config repo, composes configuration values, generates manifests out of application Helm Chart and creates a PR to the application GitOps repo. 

Application team reviews and merges the PR 

Deploy workflow creates a new solution revision, sets composed application configuration values for the solution, resolves, publishes and installs the solution on the targets specified in values.yaml. 

 

4. Update platform configuration value in Dev environment. 

The platform team needs to update maximum storage limit to 64g for all hosted applications on dev-aks cluster.  

Persona: Platform Team 

Flow: 

A platform team engineer submits a PR to update maximum storage limit in the platform config repository. 

Once the PR is merged, the Prepare-pr workflow iterates over all folders in the dev branch of the platform config repo, composes configuration values, generates manifests out of the platform Helm Chart and creates a PR to the platform GitOps repo. 

Platform team reviews and merges the PR 

Deploy workflow creates a new platform solution revision for each application deployment target (functional testing, performance testing) hosted on the dev-aks cluster, sets composed platform config values for the solutions, resolves, publishes and installs the solutions on the clustrer. 

 

5. Promote a specific data pipeline version to a specific location  

TBD 

6. Onboard a new cluster 

TBD 

7. Deployment observability 

The application dev team and the platform team have the following deployment observability objectives: 

Monitor what workload versions are deployed to clusters across the environments 

compare environments and see deployment discrepancy (e.g. how my "stage" environment is different from "prod") 

3. track deployment history per environment, per application, per workload 

4. compare desired deployment state to the reality and see deployment drift 

 

The Workload Orchestration portal offers some monitoring capabilities.  

It is possible to see a list of solutions/workloads and determine what versions are currently successfully deployed, see deployments in progress, deployment failures and previously deployed versions.  

 

Workload deployment targets (e.g. ct-func-test-dev)  contain environment name, so it’s possible to determine to which environment the deployment belongs.  

For every deployed version It is possible to see at what cluster targets it is actually deployed. 

 

 

From the perspective of the deployment target it is possible to monitor what  workload versions ae deployed on it. 

 

 

At low scale as in this PoC, the provided dashboards are enough to accomplish the deployment observability objectives.  

 

Concerns: 

At high scale (100s of clusters) it is hard to say if the new workload version has been successfully deployed to all clusters in stage environment where it was supposed to be deployed. E.g. this application should go to 73 of 100 clusters. 

There is no explicit concept of environments, it is all derived from the target names. At high scale it is difficult to compare environments with each other. 

How can one query the deployment observability data to build their own dashboards and alerts? 

Alternatives: 

It is possible to use the Deployment Observability Hub, which is currently used at Kraft. The hub is integrated with the WO service. It retrieves the desired deployment state from the GitOps repositories and deployment data from the azure resources.  

 

 

The hub provides a set of deployment observability dashboards, that Kraft engineers are used to: 

 

  

 

   What is GOOD about the setup 

Automation. All daily operations for app and platform teams are automated with GitHub actions workflows. Noone runs az cli commands or clicks buttons manually. 

GitOps experience.  App and platform teams keep working following the “as-code” paradigm with the Git repository, PR reviews, commit history, etc. The only difference is that the deployment/delivery/reconciliation is not performed by a GitOps operator, but by Toolchain Orchestrator. 

   What is BAD about the setup (major PAINPOINTS) 

No Scheduling/Assigning. Platform team has to manually decide and assign applications to the clusters. Even with a handful of applications and a few clusters across environments it becomes error prone. 

No bulk deployments. You can’t deploy an application to a million of clusters with a single command. 

 

 

 

 