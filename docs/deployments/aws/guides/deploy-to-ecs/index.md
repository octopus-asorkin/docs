---
title: Deploy to Amazon ECS
description: A guide showing you how to deploy an application to an Amazon ECS cluster using Octopus Deploy. 
position: 10
---

This guide will show you how to deploy an application to an Amazon ECS cluster using Octopus. In this guide, we will be deploying an application called **OctoPetShop**. It's a .NET Core application made up of a number of container images and they are available in a public [AWS ECR](https://aws.amazon.com/ecr/) feed.


|  Container Image | Description  |
|:-:|:-:|
| `octopetshop-web`  |  Appication front end |
| `octopetshop-productservice` | Application product API  |
| `octopetshop-shoppingcartservice`  | Application shopping cart API  |
| `sqlserver`  | SQL server container image  |
| `octopetshop-database` | Application database creation | 

The following resources have been preconfigured in **AWS**:

* An Amazon ECS cluster using Fargate.
* A public AWS ECR feed.

The following resources have been preconfigured in **Octopus**: 

* Four environments: Development, Test, Staging and Production.
* An [Amazon Account](/docs/infrastructure/accounts/aws/index.md) used for authentication to the ECS Cluster. 
* An [Amazon ECS Cluster](/docs/infrastructure/deployment-targets/amazon-ecs-cluster-target.md) deployment target. 

<span><a class="btn btn-success" href="/docs/deployments/aws/guides/deploy-to-ecs/creating-new-aws-external-feed">Get Started</a></span>