---
title: Creating the external feed
description: Create the external feed that has the container definintion to be used in the ECS steps
position: 20
hideInThisSectionHeader: true
---

In this part of the guide we will create an external feed to our [AWS ECR](https://aws.amazon.com/ecr/).

To create a new Octopus Feed, go to ( {{Library, External Feeds}} ) and click **ADD FEED**.

![External Feed](images/adding-new-feed.png "width=500")


Change the feed type to `AWS Elastic Container Registry` and fill in the of the paramaters.

![External Feed](images/adding-new-aws-ecr-feed.png "width=500")


You will need to provide the credentials configured above, as well as the region at which the registry was created. In AWS you are able to maintain separate repositories in each region.

<span><a class="btn btn-secondary" href="/docs/deployments/ecs/guides">Previous</a></span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span><a class="btn btn-success" href="/docs/deployments/ecs/guides/creating-a-new-project">Next</a></span>