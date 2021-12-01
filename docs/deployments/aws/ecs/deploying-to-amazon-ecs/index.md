---
title: Deploy Amazon ECS Service
description: Deploy a service to an Amazon ECS cluster.
position: 10
---

This is a getting started guide, will show you how to deploy an application to an Amazon ECS cluster in Octopus. In this guide, we will be deploying an application called **OctoPetShop**. It's a .NET Core application made up of a number of container definations and I have these container images already stored in a public [AWS ECR](https://aws.amazon.com/ecr/);


|  Defination Name | Description  |
|:-:|:-:|
| octopetshop-web  |  Appication front end |
| octopetshop-productservice | Application product API  |
| octopetshop-shoppingcartservice  | Application shopping cart API  |
| sqlserver  | SQL server container image  |
| octopetshop-database | Application database creation | 

The following resources have been preconfigured in AWS

* An Amazon ECS cluster using fargate.
* A public AWS ECR that can be found here that contains 

The following resources have been preconfigured in Octopus 

* Four environments: Development, Test, and Production.
* Am Amazon Account used for authentication to the ECS Cluster. To create an AWS account in Octopus you can follow [this](https://octopus.com/docs/infrastructure/accounts/aws) to help set this up. 
* The guides deploys to Amazon ECS clusters and these have already been pre configured in Amazon. To create some ECS Cluster resources you can follow [this](https://octopus.com/docs/infrastructure/deployment-targets/amazon-ecs-cluster-target
) to help get you started.

<span><a class="btn btn-success" href="/docs/deployments/ecs/guides/creating-new-extneral-feed">Get Started</a></span>




