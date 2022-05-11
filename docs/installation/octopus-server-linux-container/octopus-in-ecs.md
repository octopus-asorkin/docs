---
title: Octopus Server in ECS
description: Octopus can be installed AWS ECS and this document explain how to do this and leverage HA.
position: 41
---

One of the driving forces behind creating the Octopus Server Linux Container was so Octopus could run in a container in Kubernetes for [Octopus Cloud](/docs/octopus-cloud/index.md). With the release of the Octopus Server Linux Container image in **2020.6**, this option is available for those who want to host Octopus in their own ECS clusters.

This page describes how to run Octopus Server in ECS using Fargate.

Since [Octopus High Availability](/docs/administration/high-availability/index.md) (HA) and ECS go hand in hand, this guide will show how to support scaling Octopus Server instances with multiple HA nodes. It assumes a working knowledge of ECS concepts, such as Task definiations, Task Services, AWS Service Integrations.

## Getting started {#getting-started}

Whether you are running Octopus in a Container using Docker or ECS, or running it on Windows Server, there are a number of items to consider when creating an Octopus High Availability cluster:

- A Highly available [SQL Server database](/docs/installation/sql-server-database.md)
- A shared file system for [Artifacts, Packages, and Task Logs](/docs/administration/managing-infrastructure/server-configuration-and-file-storage/index.md#ServerconfigurationandFilestorage-FileStorageFilestorage)
- A [Load balancer](/docs/administration/high-availability/load-balancing/index.md) for traffic to the Octopus Web Portal 
- Access to each Octopus Server node for [Polling Tentacles](/docs/administration/high-availability/maintain/polling-tentacles-with-ha.md)
- Creating each Octopus Server node, including the Startup and upgrade processes that may result in database schema upgrades

The following sections describe these in more detail by creating an Octopus High Availability cluster with two Octopus Server nodes being served by a load balancer on port `80`.

:::hint
The YAML provided in this guide is designed to provide you with a starting point to help you get Octopus Server running in a container in Kubernetes. We recommend taking the time to configure your Octopus instance to meet your own organization's requirements.
:::

### SQL Server Database {#sql-database}

Each Octopus Server node stores project, environment, and deployment-related data in a shared Microsoft SQL Server Database. Since this database is shared, it's important that the database server is also highly available. 

If you don't have a SQL cluster in AWS then AWS provides a fully managed and highly available SQL database as a service called [Amazon RDS for SQL Server](https://aws.amazon.com/rds/sqlserver/).

Choosing a SQL edition is an important decision, and will depend on your organization requirements and Octopus usage. 

:::hint
It's not possible to change the edition of a SQL Server RDS instance by modifying it without taking a snapshot and restoring that to a new instance. 
:::

For a highly available SQL Server in AWS RDS, we recommend SQL Server Standard or higher.

Once you've settled on an edition, the great thing about using AWS RDS is that you can start small and scale the size of your instance on demand as your Octopus usage grows. AWS provide a [list of the instance sizes](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SQLServer.html#SQLServer.Concepts.General.InstanceClasses) for each SQL Server edition and versions.

!include <high-availability-database-recommendations>
- [Amazon RDS for SQL Server](https://aws.amazon.com/rds/sqlserver/)

!include <high-availability-db-logshipping-mirroring-note>

### Shared storage

To share common files between the Octopus Server nodes, we need to use [Amazon EFS](https://aws.amazon.com/efs/?c=s&sec=srv). EFS allows a fleet of ECS tasks and services to read and write from a EFS file system at the same time.

### Load balancer {#load-balancer}

A Load balancer is required to direct traffic to the Octopus Web Portal and optionally a way to access each of the Octopus Server nodes in an Octopus High Availability cluster may be required if you're using [Polling Tentacles](/docs/administration/high-availability/maintain/polling-tentacles-with-ha.md).


To distribute traffic to the Octopus web portal on multiple ECS nodes (Tasks), you need to use a HTTP load balancer. AWS provides a solution to distribute HTTP/HTTPS traffic to EC2 instances, Elastic Load Balancing is a highly available, secure, and elastic load balancer. There are three implementations of ELB;

* [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
* [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)
* [Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html)

If you are *only* using [Listening Tentacles](/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication.md#listening-tentacles-recommended), we recommend using the Application Load Balancer.

However, [Polling Tentacles](/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication.md#polling-tentacles) don't work well with the Application Load Balancer, so instead, we recommend using the Network Load Balancer. To setup a Network Load Balancer for Octopus High Availability with Polling Tentacles it gets a little complicated with ECS and it's discussed further down the documentation.


## Configuring Octopus in ECS

Before starting, it's assumed you have already created a VPC and subnets that allow inbound access on port 80/443 and 10933, Database and EFS file system. This guides also assumed this in a new installtion of Octopus deploy but if you are migrating from running Windows on to AWS ECS we have a migration guide you can check out [here](/docs/administration/high-availability/migrate/index.md).  

:::hint
The examples and guide are using AWS Fargate but running Octopus in ECS on EC2 is also supported.
:::

### Load balancer configuration 

In this part of the guide it's assumed your Octopus instance isn't using any polling tentacles. Given we are creating a HA conifguration of Octopus deploy there will be multiple task replicas in our ECS service running Octopus Deploy. We need to be able to distribute HTTP/HTTPS traffic to all our tasks and ensure if tasks fail or new tasks are started the loadbalancer picks these up. 

We first need to create an application loadbalancer that listens for traffic on HTTP/HTTPs and assosiate a target group that distributes the same traffic to IP address's in the same VPC as the ECS cluster hosting Octopus. For more information on managing loadblancing with ECS tasks you can find it [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html)

The YAML below is an example loadblancer ready to listen to traffic on HTTP to forward to our target group;

:::hint
As this is an example I've used HTTP but configuring SSL with Octopus in ECS is easy and can be conifigured in the same but using HTTPS. If you can find out more on how to do this [here](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html)
:::

```yaml

LoadBalancers:
- AvailabilityZones:
  - LoadBalancerAddresses: []
    SubnetId: subnet-04f96d0ec0213d656
    ZoneName: eu-west-2a
  - LoadBalancerAddresses: []
    SubnetId: subnet-052a7f6b013fbf77d
    ZoneName: eu-west-2c
  - LoadBalancerAddresses: []
    SubnetId: subnet-0bfedc22f4dbb714f
    ZoneName: eu-west-2b
  CanonicalHostedZoneId: ZHURV8PSTC4K8
  CreatedTime: '2022-05-11T02:39:02.200000+00:00'
  DNSName: solutions-default-ecs-alb-1519316669.eu-west-2.elb.amazonaws.com
  IpAddressType: ipv4
  LoadBalancerArn: arn:aws:elasticloadbalancing:eu-west-2:381713788115:loadbalancer/app/solutions-default-ecs-alb/9bc47c40495208f0
  LoadBalancerName: solutions-default-ecs-alb
  Scheme: internet-facing
  SecurityGroups:
  - sg-0412e4dbe923b7435
  State:
    Code: active
  Type: application
  VpcId: vpc-044610533a0ea44f4

```

The YAML below is an example target group ready to distribute traffic to ECS service tasks in an ECS cluster via HTTP from our loadbalancer;



```yaml

TargetGroups:
- HealthCheckEnabled: true
  HealthCheckIntervalSeconds: 30
  HealthCheckPath: /
  HealthCheckPort: traffic-port
  HealthCheckProtocol: HTTP
  HealthCheckTimeoutSeconds: 5
  HealthyThresholdCount: 5
  IpAddressType: ipv4
  LoadBalancerArns:
  - arn:aws:elasticloadbalancing:eu-west-2:381713788115:loadbalancer/app/solutions-default-ecs-alb/9bc47c40495208f0
  Matcher:
    HttpCode: '200'
  Port: 80
  Protocol: HTTP
  ProtocolVersion: HTTP1
  TargetGroupArn: arn:aws:elasticloadbalancing:eu-west-2:381713788115:targetgroup/solutions-default-ecs-web-tg/df21872e51294908
  TargetGroupName: solutions-default-ecs-web-tg
  TargetType: ip
  UnhealthyThresholdCount: 2
  VpcId: vpc-044610533a0ea44f4

```


### Cluster Configuration


```yaml

clusters:
- activeServicesCount: 1
  capacityProviders:
  - FARGATE
  - FARGATE_SPOT
  clusterArn: arn:aws:ecs:eu-west-2:381713788115:cluster/solutions-default-ecs-cluster
  clusterName: solutions-default-ecs-cluster
  defaultCapacityProviderStrategy: []
  pendingTasksCount: 0
  registeredContainerInstancesCount: 0
  runningTasksCount: 2
  settings: []
  statistics: []
  status: ACTIVE
  tags: []
failures: []

```




### Task Defintion Configuration

```yaml

---
ipcMode:
executionRoleArn: arn:aws:iam::381713788115:role/ecsTaskExecutionRole
containerDefinitions:
- dnsSearchDomains:
  environmentFiles:
  logConfiguration:
    logDriver: awslogs
    secretOptions:
    options:
      awslogs-group: "/ecs/octopus-deploy-server"
      awslogs-region: eu-west-2
      awslogs-stream-prefix: ecs
  entryPoint:
  portMappings:
  - hostPort: 80
    protocol: tcp
    containerPort: 80
  - hostPort: 10933
    protocol: tcp
    containerPort: 10933
  command:
  linuxParameters:
  cpu: 0
  environment:
  - name: ACCEPT_EULA
    value: Y
  - name: ADMIN_EMAIL
    value: adam.close@octopus.com
  - name: ADMIN_PASSWORD
    value: MyPassw0rd1!
  - name: ADMIN_USERNAME
    value: OctoAdmin
  - name: DB_CONNECTION_STRING
    value: Server=solutions-default-rds-sql.caj2bomv8w0z.eu-west-2.rds.amazonaws.com,1433;Database=Octopus;User=Octopus;Password=1QBSv2C5dp9B
  - name: MASTER_KEY
    value: SVylngjyupJG0lZwUMEBHA==
  resourceRequirements:
  ulimits:
  dnsServers:
  mountPoints:
  - readOnly:
    containerPath: "/repository"
    sourceVolume: octopus-efs-mount
  - readOnly:
    containerPath: "/artifacts"
    sourceVolume: octopus-efs-mount
  - readOnly:
    containerPath: "/taskLogs"
    sourceVolume: octopus-efs-mount
  - readOnly:
    containerPath: "/cache"
    sourceVolume: octopus-efs-mount
  - readOnly:
    containerPath: "/import"
    sourceVolume: octopus-efs-mount
  workingDirectory:
  secrets:
  dockerSecurityOptions:
  memory:
  memoryReservation:
  volumesFrom: []
  stopTimeout:
  image: octopusdeploy/octopusdeploy
  startTimeout:
  firelensConfiguration:
  dependsOn:
  disableNetworking:
  interactive:
  healthCheck:
  essential: true
  links:
  hostname:
  extraHosts:
  pseudoTerminal:
  user:
  readonlyRootFilesystem: false
  dockerLabels:
  systemControls:
  privileged:
  name: octopus-deploy-server
placementConstraints: []
memory: '1024'
taskRoleArn: arn:aws:iam::381713788115:role/ecsTaskExecutionRole
compatibilities:
- EC2
- FARGATE
taskDefinitionArn: arn:aws:ecs:eu-west-2:381713788115:task-definition/octopus-deploy-server:4
family: octopus-deploy-server
requiresAttributes:
- targetId:
  targetType:
  value:
  name: com.amazonaws.ecs.capability.logging-driver.awslogs
- targetId:
  targetType:
  value:
  name: ecs.capability.execution-role-awslogs
- targetId:
  targetType:
  value:
  name: ecs.capability.efsAuth
- targetId:
  targetType:
  value:
  name: com.amazonaws.ecs.capability.docker-remote-api.1.19
- targetId:
  targetType:
  value:
  name: ecs.capability.efs
- targetId:
  targetType:
  value:
  name: com.amazonaws.ecs.capability.task-iam-role
- targetId:
  targetType:
  value:
  name: com.amazonaws.ecs.capability.docker-remote-api.1.25
- targetId:
  targetType:
  value:
  name: com.amazonaws.ecs.capability.docker-remote-api.1.18
- targetId:
  targetType:
  value:
  name: ecs.capability.task-eni
pidMode:
requiresCompatibilities:
- FARGATE
networkMode: awsvpc
runtimePlatform:
  operatingSystemFamily: LINUX
  cpuArchitecture:
cpu: '512'
revision: 4
status: ACTIVE
inferenceAccelerators:
proxyConfiguration:
volumes:
- fsxWindowsFileServerVolumeConfiguration:
  efsVolumeConfiguration:
    transitEncryptionPort:
    fileSystemId: fs-075b29a2a107ed8f1
    authorizationConfig:
      iam: DISABLED
      accessPointId:
    transitEncryption: DISABLED
    rootDirectory: "/"
  name: octopus-efs-mount
  host:
  dockerVolumeConfiguration:

```


### Cluster Service Configuration 

```yaml

services:
- clusterArn: arn:aws:ecs:eu-west-2:381713788115:cluster/solutions-default-ecs-cluster
  createdAt: '2022-05-11T04:57:25.365000+01:00'
  createdBy: arn:aws:iam::381713788115:role/aws-reserved/sso.amazonaws.com/ap-southeast-2/AWSReservedSSO_DeveloperAccess_645f9848983dec35
  deploymentConfiguration:
    deploymentCircuitBreaker:
      enable: false
      rollback: false
    maximumPercent: 200
    minimumHealthyPercent: 100
  deployments:
  - createdAt: '2022-05-11T05:39:30.317000+01:00'
    desiredCount: 2
    failedTasks: 20
    id: ecs-svc/2182780950350705823
    launchType: FARGATE
    networkConfiguration:
      awsvpcConfiguration:
        assignPublicIp: ENABLED
        securityGroups:
        - sg-0412e4dbe923b7435
        subnets:
        - subnet-0bfedc22f4dbb714f
        - subnet-052a7f6b013fbf77d
        - subnet-04f96d0ec0213d656
    pendingCount: 0
    platformFamily: Linux
    platformVersion: 1.4.0
    rolloutState: IN_PROGRESS
    rolloutStateReason: ECS deployment ecs-svc/2182780950350705823 in progress.
    runningCount: 2
    status: PRIMARY
    taskDefinition: arn:aws:ecs:eu-west-2:381713788115:task-definition/octopus-deploy-server:4
    updatedAt: '2022-05-11T05:39:30.317000+01:00'
  desiredCount: 2
  enableECSManagedTags: true
  enableExecuteCommand: false

```

### Upgrading Octopus in ECS

Starting the ECS service for the first time works exactly as Octopus requires; one task at a time is successfully started before the next. This gives the first task a chance to update the SQL schema with any required changes, and all other task start-up and share the already configured database.

If the task definiation is updated with an new docker imagine, and the service tasks are redeployed, by default the [rolling update strategy]https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html) is used. A rolling update creates new task, deltes the old one, updates the loadblancer target group with the new task and repeats for each task. This means that during the update there will be a mix of old and new versions of Octopus. This  will cause problems, when the first new task starts it will upgrade the database schema meaning old octopus instances that havn't been upgraded yet will be using the wrong database schema and this could lead to unpredactible results for anyone using Octopus.









