---
title: Auto Scaling Recommendations
description: General recommendations when using any auto-scaling technology with Octopus Deploy.
position: 10
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

Most tools give the option to use provided OS images or a custom built image for the servers in the auto scaling group.  There are advantages and disadvantages to either option.  As we don't know your use cases and context, it would be impossible for us to provide a recommendation.

No matter which option you choose, you will need to bootstrap the install of the tentacle.  This could be done as a script using a [custom script extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows) for VMSS or included as part of the custom image.

Please see our documentation on automating tentacle installations:

- [Windows](/docs/infrastructure/deployment-targets/windows-targets/automating-tentacle-installation.md)
- [Linux](/docs/infrastructure/deployment-targets/linux/tentacle/index.md#automation-scripts)

For example scripts, please see our [Infrastructure as Code Sample GitHub Repo](https://github.com/OctopusSamples/IaC/tree/master/bootstrap).

:::hint
Unlike static servers, which are treated more like pets where you handcraft and maintain them, servers created via auto-scaling should be treated as cattle.  At any given point a server might go down and you'll need to replace it.  Assume whatever could go wrong can go wrong.  For the bootstrap script assume:

1. It will be run multiple times (perhaps after each restart)
2. Something might have happened to the registration in Octopus Deploy (delete the wrong record), so force the registration
:::

## Polling or Listening Tentacles

Octopus Deploy Tentacles support two communication modes [polling or listening](/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication.md).  When a tentacle is in listening mode, the Octopus Server pushes commands to the tentacle.  When the tentacle is in polling mode, it will pull commands from the Octopus Server.  

Generally, listening tentacles are more efficent and are easier to configure when used with Octopus' [High Availability](/docs/administration/high-availability/index.md) feature.  However, they require the Octopus Server to know the hostname or IP address of the server running the tentacle.  In addition, they will require a custom security group and firewall setting to accept inbound connections.  

In our testing, polling tentacles tended to have an easier time registering with an Octopus Instance than listening tentacles.  

# Scaling In

Eventually you are going run into the following scenarios:

- A server is registered with Octopus Deploy but no longer exists in the auto scaling group.  
- A server is deleted (manually or via a rule) in the middle of a deployment or runbook run.

How the server is registered, as a deployment target or worker, affects the mitigation for those scenarios.

## Custom Machine Policy

Create a custom machine policy that runs regularly (10 minutes is a good starting point).  Only assign servers in auto scaling groups to this machine policy.  

Change the following settings from the default machine policy:

- Interval: 10 minutes
- During Health Checks: Unavailable deployment targets will not cause health checks to fail
- Calamari Updates: Always keep Calamari up to date
- Clean Up Unavailable Deployment Targets: Automatically delete unavailable machines after 15 minutes

![auto scaling custom machine policy](images/auto-scaling-machine-policy.png)

## Project Configuration

To account for a variety of use cases, any project that will deploy to servers in auto scaling groups should be configured to:

- Deployment Targets Required: Allow deployments to be created when there are no deployment targets
- Unavailable Deployment Targets: Skip and Continue (specify a role)
- Unhealthy Deployment Targets: Exclude (in the event the machine policy hasn't removed the server yet)

![auto scaling projects](images/project-settings-with-auto-scaling.png)

By default, a deployment will fail when a deployment target is removed while running a step on it.  The recommendation is to configure [guided failure mode](docs/releases/guided-failures.md).  Any errors will pause the deployment and ask for user intervention.

Guided failure mode can be configured at the environment level.
![guided failure environment](images/environment-guided-failure-mode.png)

Or, at the project level.
![project guided failure mode](images/project-guided-failure-mode.png)