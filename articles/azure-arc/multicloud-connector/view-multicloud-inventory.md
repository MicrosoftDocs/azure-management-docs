---
title: View multicloud inventory with the multicloud connector enabled by Azure Arc
description: View multicloud inventory with the multicloud connector enabled by Azure Arc
ms.topic: how-to
ms.date: 10/22/2025
# Customer intent: "As a cloud architect, I want to view my AWS resources within Azure, so that I can maintain an organized and unified multicloud inventory for efficient resource governance."
---

# View multicloud inventory with the multicloud connector enabled by Azure Arc

The **Inventory** solution of the multicloud connector shows an up-to-date view of your resources from other public clouds in Azure, providing you with a single place to see all your cloud resources.

Currently, the multicloud connector provides support for connecting resources from these public clouds:

- Amazon Web Services (AWS)
- Google Cloud Platform (GCP) (preview)

After you enable the **Inventory** solution, metadata from the assets in the source cloud is included with the asset representations in Azure. You can also apply Azure tags or Azure policies to these resources. This solution allows you to query for all your cloud resources through Azure Resource Graph, such as querying to find all Azure, AWS, and GCP resources with a specific tag.

The **Inventory** solution scans your source cloud regularly to update the view represented in Azure. You can specify the interval to query when you [connect your public cloud](add-public-cloud.md) and configure the **Inventory** solution.

## Supported services

Today, resources associated with the following AWS and GCP services are scanned and represented in Azure. When you [create the **Inventory** solution](add-public-cloud.md#add-your-public-cloud-in-the-azure-portal), all available services are selected by default, but you can optionally include any services.

### [AWS](#tab/aws)

The following table shows the AWS services that are scanned, the resource types associated with each service, and the Azure namespace that corresponds to each resource type.

| AWS service  | AWS resource type  | Azure namespace |
|--------------|--------------------|--------------------------------------|
| Access Analyzer | `accessAnalyzerAnalyzers`     | `Microsoft.AwsConnector/accessAnalyzerAnalyzers`|
| API Gateway   | `apiGatewayRestApis`     | `Microsoft.AwsConnector/apiGatewayRestApis`|
| API Gateway   | `apiGatewayStages`     | `Microsoft.AwsConnector/apiGatewayStages`|
| App Sync   | `appSyncGraphQLApis`     | `Microsoft.AwsConnector/appSyncGraphQLApis`|
| Autoscaling  | `autoScalingAutoScalingGroups`     | `Microsoft.AwsConnector/autoScalingAutoScalingGroups`|
| Cloud Formation   | `cloudFormationStacks`     | `Microsoft.AwsConnector/cloudFormationStacks`|
| Cloud Formation   | `cloudFormationStackSets`     | `Microsoft.AwsConnector/cloudFormationStackSets`|
| Cloud Front   | `cloudFront`     | `Microsoft.AwsConnector/cloudFrontDistributions`|
| Cloud Trail   | `cloudTrailTrails`     | `Microsoft.AwsConnector/cloudTrailTrails`|
| Cloud Watch   | `cloudWatchAlarms`     | `Microsoft.AwsConnector/cloudWatchAlarms`|
| Code Build   | `codeBuildProjects`     | `Microsoft.AwsConnector/codeBuildProjects`|
| Code Build   | `codeBuildSourceCredentialsInfos`     | `Microsoft.AwsConnector/codeBuildSourceCredentialsInfos`|
| Config  | `configServiceConfigurationRecorders`     | `Microsoft.AwsConnector/configServiceConfigurationRecorders`|
| Config   | `configServiceConfigurationRecorderStatuses`     | `Microsoft.AwsConnector/configServiceConfigurationRecorderStatuses`|
| Config  | `configServiceDeliveryChannels`     | `Microsoft.AwsConnector/configServiceDeliveryChannels`|
| DAX  | `daxClusters`     | `Microsoft.AwsConnector/daxClusters`|
| DMS  | `databaseMigrationServiceReplicationInstances`     | `Microsoft.AwsConnector/databaseMigrationServiceReplicationInstances`|
| Dynamo DB    | `dynamoDBContinuousBackupsDescriptions`     | `Microsoft.AwsConnector/dynamoDBContinuousBackupsDescriptions`|
| Dynamo DB    | `dynamoDBTables`     | `Microsoft.AwsConnector/dynamoDBTables`|
| EC2    | `ec2Instances`    | `Microsoft.HybridCompute/machines/EC2InstanceId/providers/Microsoft.AwsConnector/Ec2Instances`|
| EC2    | `ec2AccountAttributes`    | `Microsoft.AwsConnector/ec2AccountAttributes`|
| EC2    | `ec2Addresses`    | `Microsoft.AwsConnector/ec2Addresses`|
| EC2    | `ec2FlowLogs`    | `Microsoft.AwsConnector/ec2FlowLogs`|
| EC2    | `ec2Images`    | `Microsoft.AwsConnector/ec2Images`|
| EC2    | `ec2Ipams`    | `Microsoft.AwsConnector/ec2Ipams`|
| EC2    | `ec2KeyPairs`    | `Microsoft.AwsConnector/ec2KeyPairs`|
| EC2    | `ec2Subnets`    | `Microsoft.AwsConnector/ec2Subnets`|
| EC2    | `ec2Volumes`   | `Microsoft.AwsConnector/ec2Volumes`|
| EC2    | `ec2VPCs`  | `Microsoft.AwsConnector/ec2VPCs`|
| EC2    | `ec2NetworkAcls`  | `Microsoft.AwsConnector/ec2NetworkAcls`|
| EC2    | `ec2NetworkInterfaces`| `Microsoft.AwsConnector/ec2NetworkInterfaces`|
| EC2    | `ec2RouteTables` | `Microsoft.AwsConnector/ec2RouteTables`|
| EC2    | `ec2VPCEndpoints` | `Microsoft.AwsConnector/ec2VPCEndpoints`|
| EC2    | `ec2VPCPeeringConnections` | `Microsoft.AwsConnector/ec2VPCPeeringConnections`|
| EC2    | `ec2InstanceStatuses` | `Microsoft.AwsConnector/ec2InstanceStatuses`|
| EC2    | `ec2SecurityGroups` | `Microsoft.AwsConnector/ec2SecurityGroups`|
| EC2    | `ec2Snapshots` | `Microsoft.AwsConnector/ec2Snapshots`|
| ECR   | `ecrImageDetails`     | `Microsoft.AwsConnector/ecrImageDetails`|
| ECR   | `ecrRepositories`     | `Microsoft.AwsConnector/ecrRepositories`|
| ECS   | `ecsClusters`     | `Microsoft.AwsConnector/ecsClusters`|
| ECS   | `ecsServices`     | `Microsoft.AwsConnector/ecsServices`|
| ECS   | `ecsTaskDefinitions`     | `Microsoft.AwsConnector/ecsTaskDefinitions`|
| EFS   | `efsFileSystems`     | `Microsoft.AwsConnector/efsFileSystems`|
| EFS   | `efsMountTargets`     | `Microsoft.AwsConnector/efsMountTargets`|
| EKS   | `eksClusters`     | `Microsoft.Kubernetes/connectedclusters/clusterName_region/providers/Microsoft.AwsConnector/eksClusters`|
| EKS   | `eksNodegroups`     | `Microsoft.AwsConnector/eksNodegroups`|
| Elastic Beanstalk   | `elasticBeanstalkApplications`     | `Microsoft.AwsConnector/elasticBeanstalkApplications`|
| Elastic Beanstalk   | `elasticBeanstalkConfigurationTemplates`     | `Microsoft.AwsConnector/elasticBeanstalkConfigurationTemplates`|
| Elastic Beanstalk   | `elasticBeanstalkEnvironments`     | `Microsoft.AwsConnector/elasticBeanstalkEnvironments`|
| Elastic Load Balancer V2    | `elasticLoadBalancingV2LoadBalancers`| `Microsoft.AwsConnector/elasticLoadBalancingV2LoadBalancers`|
| Elastic Load Balancer V2    | `elasticLoadBalancingV2Listeners`| `Microsoft.AwsConnector/elasticLoadBalancingV2Listeners`|
| Elastic Load Balancer V2    | `elasticLoadBalancingV2TargetGroups`| `Microsoft.AwsConnector/elasticLoadBalancingV2TargetGroups`|
| Elastic Load Balancer V2    | `elasticLoadBalancingV2TargetHealthDescriptions`| `Microsoft.AwsConnector/elasticLoadBalancingV2TargetHealthDescriptions`|
| EMR   | `emrClusters` | `Microsoft.AwsConnector/emrClusters`|
| GuardDuty   | `guardDutyDetectors` | `Microsoft.AwsConnector/guardDutyDetectors`|
| IAM   | `iamAccessKeyLastUseds` | `Microsoft.AwsConnector/iamAccessKeyLastUseds`|
| IAM   | `iamAccessKeyMetaData` | `Microsoft.AwsConnector/iamAccessKeyMetaData`|
| IAM   | `iamMFADevices` | `Microsoft.AwsConnector/iamMFADevices`|
| IAM   | `iamPasswordPolicies` | `Microsoft.AwsConnector/iamPasswordPolicies`|
| IAM   | `iamPolicyVersions` | `Microsoft.AwsConnector/iamPolicyVersions`|
| IAM   | `iamRoles` | `Microsoft.AwsConnector/iamRoles`|
| IAM   | `iamManagedPolicies` | `Microsoft.AwsConnector/iamManagedPolicies`|
| IAM   | `iamServerCertificates` | `Microsoft.AwsConnector/iamServerCertificates`|
| IAM   | `iamUserPolicies` | `Microsoft.AwsConnector/iamUserPolicies`|
| IAM   | `iamVirtualMFADevices` | `Microsoft.AwsConnector/iamVirtualMFADevices`|
| KMS   | `kmsKeys` | `Microsoft.AwsConnector/kmsKeys`|
| Lambda   | `lambdaFunctions` | `Microsoft.AwsConnector/lambdaFunctions`|
| Lightsail   | `lightsailInstances` | `Microsoft.AwsConnector/lightsailInstances`|
| Lightsail   | `lightsailBuckets`| `Microsoft.AwsConnector/lightsailBuckets`|
| Logs   | `logsLogGroups` | `Microsoft.AwsConnector/logsLogGroups`|
| Logs   | `logsLogStreams` | `Microsoft.AwsConnector/logsLogStreams`|
| Logs   | `logsMetricFilters` | `Microsoft.AwsConnector/logsMetricFilters`|
| Logs   | `logsSubscriptionFilters` | `Microsoft.AwsConnector/logsSubscriptionFilters`|
| Macie   | `macieAllowLists` | `Microsoft.AwsConnector/macieAllowLists`|
| Macie2   | `macie2JobSummaries` | `Microsoft.AwsConnector/macie2JobSummaries`|
| Network Firewalls   | `networkFirewallFirewalls` | `Microsoft.AwsConnector/networkFirewallFirewalls`|
| Network Firewalls   | `networkFirewallFirewallPolicies` | `Microsoft.AwsConnector/networkFirewallFirewallPolicies`|
| Network Firewalls   | `networkFirewallRuleGroups` | `Microsoft.AwsConnector/networkFirewallRuleGroups`|
| Open Search Service   | `openSearchDomainStatuses` | `Microsoft.AwsConnector/openSearchDomainStatuses`|
| Organization   | `organizationsAccounts` | `Microsoft.AwsConnector/organizationsAccounts`|
| Organization   | `organizationsOrganizations` | `Microsoft.AwsConnector/organizationsOrganizations`|
| RDS   | `rdsDBInstances` | `Microsoft.AwsConnector/rdsDBInstances`|
| RDS   | `rdsDBClusters` | `Microsoft.AwsConnector/rdsDBClusters`|
| RDS   | `rdsEventSubscriptions` | `Microsoft.AwsConnector/rdsEventSubscriptions`|
| RDS   | `rdsDBSnapshots` | `Microsoft.AwsConnector/rdsDBSnapshots`|
| RDS   | `rdsDBSnapshotAttributesResults` | `Microsoft.AwsConnector/rdsDBSnapshotAttributesResults`|
| RDS   | `rdsEventSubscriptions` | `Microsoft.AwsConnector/rdsEventSubscriptions`|
| Redshift   | `redshiftClusters` | `Microsoft.AwsConnector/redshiftClusters`|
| Redshift   | `redshiftClusterParameterGroups` | `Microsoft.AwsConnector/redshiftClusterParameterGroups`|
| Route 53   | `route53DomainsDomainSummaries` | `Microsoft.AwsConnector/route53DomainsDomainSummaries`|
| Route 53   | `route53HostedZones` | `Microsoft.AwsConnector/route53HostedZones`|
| SageMaker  | `sageMakerApps` | `Microsoft.AwsConnector/sageMakerApps`|
| SageMaker   | `sageMakerDevices` | `Microsoft.AwsConnector/sageMakerDevices`|
| SageMaker   | `sageMakerImages` | `Microsoft.AwsConnector/sageMakerImages`|
| SageMaker   | `sageMakerNotebookInstanceSummaries` | `Microsoft.AwsConnector/sageMakerNotebookInstanceSummaries`|
| Secrets Manager   | `secretsManagerResourcePolicies` | `Microsoft.AwsConnector/secretsManagerResourcePolicies`|
| Secrets Manager   | `secretsManagerSecrets` | `Microsoft.AwsConnector/secretsManagerSecrets`|
| Secrets Manager   | `secretsManagerSecrets` | `Microsoft.AwsConnector/secretsManagerSecrets`|
| S3   | `s3Buckets` | `Microsoft.AwsConnector/s3Buckets`|
| S3   | `s3AccessControlPolicies` | `Microsoft.AwsConnector/s3AccessControlPolicies`|
| S3   | `s3ControlMultiRegionAccessPointPolicyDocuments` | `Microsoft.AwsConnector/s3ControlMultiRegionAccessPointPolicyDocuments`|
| S3   | `s3BucketPolicies` | `Microsoft.AwsConnector/s3BucketPolicies`|
| S3   | `s3AccessPoints` | `Microsoft.AwsConnector/s3AccessPoints`|
| SNS   | `snsTopics` | `Microsoft.AwsConnector/snsTopics`|
| SNS   | `snsSubscriptions` | `Microsoft.AwsConnector/snsSubscriptions`|
| SQS   | `sqsQueues` | `Microsoft.AwsConnector/sqsQueues`|
| SSM   | `ssmInstanceInformations` | `Microsoft.AwsConnector/ssmInstanceInformations`|
| SSM   | `ssmParameters` | `Microsoft.AwsConnector/ssmParameters`|
| SSM   | `ssmResourceComplianceSummaryItems` | `Microsoft.AwsConnector/ssmResourceComplianceSummaryItems`|
| WAF   | `wafWebACLSummaries` | `Microsoft.AwsConnector/wafWebACLSummaries`|
| WAFv2   | `wafv2LoggingConfigurations` | `Microsoft.AwsConnector/wafv2LoggingConfigurations`|

### [GCP](#tab/gcp)

The following table shows the GCP services that are scanned, the resource types associated with each service, and the Azure namespace that corresponds to each resource type.

| GCP service | GCP resource type | Azure namespace |
|-------------|-------------------|-----------------|
| BigQuery | `bigQueryDatasets` | `Microsoft.GcpConnector/bigQueryDatasets` |
| CloudFunctions | `cloudFunctions` | `Microsoft.GcpConnector/cloudFunctions` |
| Compute | `computeInstances` | `Microsoft.GcpConnector/computeInstances` |
| Container | `containerclusters` | `Microsoft.GcpConnector/containerclusters` |
| Storage | `storageBuckets` | `Microsoft.GcpConnector/storageBuckets` |
| SQL Admin | `sqlAdminInstances` | `Microsoft.GcpConnector/sqlAdminInstances` |

---

## Resource representation in Azure

After you connect your cloud and enable the **Inventory** solution, the multicloud connector creates a new resource group using the naming convention `<PublicCloud>_<AccountID>`. Azure representations of your resources are created in this resource group, using the `AwsConnector` or `GcpConnector` namespace values described in the previous section. You can apply Azure tags and policies to these resources.

Resources that are discovered and projected in Azure are placed in Azure regions, using a [standard mapping scheme](resource-representation.md#region-mapping).

> [!NOTE]
> If you have EC2 instances or GCP VMs that have previously been [connected to Azure Arc](/azure/azure-arc/servers/deployment-options), the connector will create the related inventory resource as child resource of the Microsoft.HybridCompute/machines if the [prerequisites](add-public-cloud.md#azure-prerequisites) have been met in the subscription where the Arc machine resides. Otherwise, the Inventory resource will not be created.

## Permission options

1. **Global Read**: Provides read only access to all resources in the AWS account or the GCP Organization/Project. When new services are introduced, the connector can scan for those resources without requiring an updated CloudFormation template in AWS or Terraform template in GCP.

1. **Least Privilege Access**: Provides read access to only the resources under the selected services. If you choose to scan for more resources in the future, you must upload a new template.

## Periodic sync options

The periodic sync time that you select when configuring the **Inventory** solution determines how often your source cloud (AWS or GCP) is scanned and synced to Azure. By enabling periodic sync, changes to your source cloud resources are reflected in Azure. For instance, if a resource is deleted in your source cloud, that resource is also deleted in Azure.

If you prefer, you can turn periodic sync off when configuring this solution. If you do so, your Azure representation may become out of sync with your source cloud resources, as Azure won't be able to rescan and detect any changes.

## Querying for resources in Azure Resource Graph

[Azure Resource Graph](/azure/governance/resource-graph/overview) is an Azure service designed to extend Azure Resource Management by providing efficient and performant resource exploration. Running queries at scale across a given set of subscriptions helps you effectively govern your environment.

You can run queries using [Resource Graph Explorer](/azure/governance/resource-graph/first-query-portal) in the Azure portal. Some example queries for common scenarios are shown here.

### Query all onboarded multicloud asset inventories

```kusto
resources
| where subscriptionId == "<subscription ID>"
| where id contains "microsoft.awsconnector" 
| union (awsresources | where type == "microsoft.awsconnector/ec2instances" and subscriptionId =="<subscription ID>")
| extend awsTags= properties.awsTags, azureTags = ['tags']
| project subscriptionId, resourceGroup, type, id, awsTags, azureTags, properties 
```

### Query for all resources under a specific connector

```kusto
resources
| extend connectorId = tolower(tostring(properties.publicCloudConnectorsResourceId)), resourcesId=tolower(id)
| join kind=leftouter (
    awsresources
    | extend pccId = tolower(tostring(properties.publicCloudConnectorsResourceId)), awsresourcesId=tolower(id)
    | extend parentId = substring(awsresourcesId, 0, strlen(awsresourcesId) - strlen("/providers/microsoft.awsconnector/ec2instances/default"))
) on $left.resourcesId == $right.parentId
| where connectorId =~ "yourConnectorId" or pccId =~ "yourConnectorId"
| extend resourceType = tostring(split(iif (type =~ "microsoft.hybridcompute/machines", type1, type), "/")[1])
```

### Query for all virtual machines in Azure and AWS, along with their instance size

```kusto
resources 
| where (['type'] == "microsoft.compute/virtualmachines") 
| union (awsresources | where type == "microsoft.awsconnector/ec2instances")
| extend cloud=iff(type contains "ec2", "AWS", "Azure")
| extend awsTags=iff(type contains "microsoft.awsconnector", properties.awsTags, ""), azureTags=tags
| extend size=iff(type contains "microsoft.compute", properties.hardwareProfile.vmSize, properties.awsProperties.instanceType.value)
| project subscriptionId, cloud, resourceGroup, id, size, azureTags, awsTags, properties
```

### Query for all functions across Azure and AWS

```kusto
resources
| where (type == 'microsoft.web/sites' and ['kind'] contains 'functionapp') or type == "microsoft.awsconnector/lambdafunctionconfigurations"
| extend cloud=iff(type contains "awsconnector", "AWS", "Azure")
| extend functionName=iff(cloud=="Azure", properties.name,properties.awsProperties.functionName), state=iff(cloud=="Azure", properties.state, properties.awsProperties.state), lastModifiedTime=iff(cloud=="Azure", properties.lastModifiedTimeUtc,properties.awsProperties.lastModified), location=iff(cloud=="Azure", location,properties.awsRegion),  tags=iff(cloud=="Azure", tags, properties.awsTags)
| project cloud, functionName, lastModifiedTime, location, tags
```

### Query for all resources with a certain tag

```kusto
resources 
| extend awsTags=iff(type contains "microsoft.awsconnector", properties.awsTags, ""), azureTags=tags 
| where awsTags contains "<yourTagValue>" or azureTags contains "<yourTagValue>" 
| project subscriptionId, resourceGroup, name, azureTags, awsTags
```

## Next steps

- Learn about the [multicloud connector **Arc Onboarding** solution](onboard-multicloud-vms-arc.md).
- Learn more about [Azure Resource Graph](/azure/governance/resource-graph/overview).

