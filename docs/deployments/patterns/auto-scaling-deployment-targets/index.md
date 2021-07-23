---
title: Auto Scaling Virtual Machines
description: How to implement auto scaling (Azure Virtual Machine Scale Sets or AWS Auto Scaling) with Octopus Deploy.
position: 10
hideInThisSection: true
---

One of the main advantages of migrating to the cloud is the ability to scale up and down infrastructure.  A lot of the platform as a service (PaaS) offerings (Azure Web Apps, AWS Lambda, Azure Containers, etc.) have scalability built-in.  However, not every application can (or should) be migrated to run on PaaS.  For a variety of reasons, applications have to continue to run on virtual machines.  Thankfully, you can still take advantage of scalability provided by migrating to the cloud.  Cloud providers provide the ability to auto-scale Virtual Machines.  Two of the more common auto-scaling offerings are [Azure Virtual Machine Scale Sets](https://azure.microsoft.com/en-us/services/virtual-machine-scale-sets/) (VMSS) and [AWS Auto Scaling Groups](https://aws.amazon.com/autoscaling/) (ASG).  

This section will walk you through how to configure Octopus Deploy to best work with Auto Scaling Virtual Machines.

# Recommendations

When creating your first VMSS or ASG you will be presented with a series of decisions to make.  We have created a page to help guide you through those questions.

[Auto Scaling Recommendations](/docs/deployments/patterns/auto-scaling-virtual-machines/auto-scaling-recommendations.md)

# Configuring Octopus Deploy

We have written guides covering the two more popular auto-scaling tools, Azure Virtual Machine Scale Sets and AWS Auto Scaling Groups.

- [Azure Virtual Machine Scale Set Deployment Targets](/docs/deployments/patterns/auto-scaling-virtual-machines/azure/index.md)

