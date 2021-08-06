---
title: Auto Scaling Design Considerations
description: Design considerations when leveraging auto scaling technologies with Octopus Deploy.
position: 20
hideInThisSection: true
---

When leveraging auto-scaling technology for the first time, for example [Azure Virtual Machine Scale Sets](https://azure.microsoft.com/en-us/services/virtual-machine-scale-sets/) (VMSS) or [AWS Auto Scaling](https://aws.amazon.com/autoscaling/) (ASG) you will be presented with a series of decisions to make.  This section will walk you through the ones that pertain specifically to Octopus Deploy.

# Manual Scaling or Auto Scaling

Despite being called "Auto Scaling," most tools allow you to manually scale up and down your infrastructure.  That is when you logon to your tool and manually increase or decrease the number of servers.  

Auto Scaling is when you configure a set of rules in the tool.  For example, you can scale on a schedule; scale to 20 servers in the morning and down to 2 servers at night.  Or, based on a metric, the network traffic reaches a threshold then add 3 servers.

A lot of our users opt for manual scaling over auto scaling when an increase in traffic is known ahead of time.  For example, a online retail store will increase the number of virtual machines days or weeks prior to a major sale to perform load tests.  Other customers prefer to scale up and scale down on a schedule.  A bank might scale up their web servers at 5 AM and down at 9 PM.  

Behind the scenes, Octopus will treat both an auto-scale event and a manual-scale event the same.  You need to add 1 to N servers and get the latest software deployed.  The guides in our documentation will account for both scenarios.

:::hint
If you opt for manual scaling or are only going to scale machines on a schedule, we recommend taking a look at [the runbooks feature](/docs/runbooks/index.md) in Octopus Deploy.  This will allow you to put certain guardrails, such as the max number of servers a person can request, in place while empowering teams to scale up and down as they see fit.  It is also a great way of providing this functionality without giving everyone access to your tool's portal.
:::

# Using provided images vs. building your own

Most tools give the option to use provided OS images or a custom built image for the servers in the auto scaling group.  A custom image is one you build with all the required software (IIS, Java, NGINX, etc.) pre-installed.  If you are going to create a custom image our recommendation is to:

- Have the tentacle pre-installed on the custom image.
- Use the custom script extension to configure the tentacle.

Configuring the tentacle will require you to provide an Octopus URL, API Key, Thumbprint, Server Roles, Environments, and Tenants.  With this approach you can have a single custom image per operating system, rather than a custom image per application.

:::hint
Use the provided images plus the custom script extension to download, install, and configure the tentacle when starting out.  Creating a custom image puts the reponsibility on you to keep the image up to date with the latest patches and bug fixes. 
:::

# Polling or Listening Tentacles

Octopus Deploy Tentacles support two communication modes [polling or listening](/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication.md).  When a tentacle is in listening mode, the Octopus Server pushes commands to the tentacle.  When the tentacle is in polling mode, it will pull commands from the Octopus Server.  

Generally, listening tentacles are more efficent and are easier to configure when used with Octopus' [High Availability](/docs/administration/high-availability/index.md) feature.  However, they require the Octopus Server to know the hostname or IP address of the server running the tentacle.  In addition, they will require a custom security group and firewall setting to accept inbound connections.  