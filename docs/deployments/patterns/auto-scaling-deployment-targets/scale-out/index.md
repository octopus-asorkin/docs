---
title: Scale Out Event
description: Recommendations how to handle a scale out event from an auto scaling group in Octopus Deploy.
position: 50
hideInThisSection: true
---

A scale out event is when virtual machines are added to an auto-scaling group.  It can be triggered automatically via a configured rule, or manually by a person.  When a scale out event occurs, the new virtual machines will need to be added to Octopus Deploy.  This guide will provide you the various options on how to do this.

# Octopus Deploy Prep Work

Octopus Deploy will need to be configured to handle auto-scaling virtual machines.  That configuration includes:

- Custom Machine Policy
- Updating project settings to handle missing or non-existant deployment targets
- Machine roles assigned to deployment targets.

Please see the [Scale out Octopus Deploy prep work](/docs/deployments/patterns/auto-scaling-deployment-targets/scale-out/scale-out-prep-work.md) for more details.

# Deployment Target Triggers

Use [deployment target triggers](/docs/projects/project-triggers/deployment-target-triggers.md) tied to a `Machine Created` event.  It is the easiest way to handle scale out events.  When a new virtual machine registers itself with Octopus Deploy, the trigger will fire and deploy to those newly created machines.

To configure a deployment target trigger you will want to:

1. From the project overview screen go to **{{Deployments, Triggers}}**.
2. In the top right cornger click on **{{Add Trigger, Deployment target trigger}}**.
3. Configure the following settings:
    - Name: Provide a name for the trigger.
    - Event Categories: select `Machine Created`.
    - Environments: enter the environments the VMSS exists in.
    - Target roles: select the target role of the VMs in the VMSS.
4. Click **SAVE** to save the new trigger.

![Orchestration Project Deployment Target Trigger](images/orchestration-project-deployment-target-trigger.png)

# Processing Scale Out Events

There are two factors you need to consider when configuring Octopus Deploy to process scale out events.

1. How many virtual machines will be added in a scale out event.
2. How long it takes to configure and deploy to a fresh virtual machine.

In our testing, we've found scaling out to more than five virtual machines AND having a deployment take longer than five minutes can make a scale out event take double or triple the amount of time than it should.

- The number of instances in a VMSS is increased from 1 to 10.
- Octopus Deploy sees 4 of the 9 new VMs have come online and trigger a deployment.
- After that deployment is finished, Octopus Deploy will see the remaining 5 VMs and trigger a new deployment.

![Deployment target triggers with auto scaling groups](images/auto-scaling-with-deployment-target-triggers.png)

This is due to the scattershot nature of virtual machine provisioning.  Sometimes all virtual machines finish provisioning within a few seconds of one another.  Other times, they finish provisioning 60 or 90 seconds apart.

There are three auto-scaling configuration options for Octopus Deploy.  They are:

1. Wait for deployment target triggers to run on all virtual machines
2. Pause a deployment and wait for all virtual machines to finish provisioning
3. Configure Octopus Deploy Runbooks to manually scale out

## Wait for deployment target triggers to run on all new virtual machines

With this option, you let Octopus Deploy work as-is.  If the scale out event is adding 15 new virtual machines, then Octopus Deploy will process the first batch that comes online (generally 5-8), then once that deployment is finished it will process the remaining machines.  

We recommend this option for a few reasons:

 - There is nothing additional to configure outside of the deployment target trigger, project settings, and a new machine policy.    
 - You don't have to modify your deployment process.
 - Most scale out events we've encountered are less than ten virtual machines per project.  Typically we see two to five virtual machines added.

:::hint
Start with this option first and adjust as needed.  Unless you know for certain you plan on scaling out by more than five virtual machines AND each deployment takes more than five minutes.
:::

## Pause the deployment and wait for all virtual machines to finish provisioning

If each scale out event is more than five virtual machines AND each deployment takes more than 10 minutes to finish, then we recommend leveraging the new step templates to pause the deployment.

:::hint
It is not uncommon for a deployment to a brand new virtual machine to take longer than 10 minutes.  A lot of our customers also install and configure dependent software (IIS, Java, .NET, NGINX) to ensure each server is in a "known state."  
:::

The new step templates we've added cover the popular cloud providers.

- [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status-(deployment-targets))
- [Check ASG Provsion Status step template](TBD)

While they target different technologies, the basically work the same.

- When a scale out event occurs, Octopus Deploy will wait until all machines are provisioned then do the deployment.
- They will detect when a duplicate trigger run occurs and give you the option to cancel the deployment or proceed.
- They will set output variables containing the names and ids of the newly created virtual machines.
- They will reconcile the list in the auto scaling group with what is stored in Octopus Deploy.  Any Virtual Machines in Octopus Deploy not in the auto scaling group will be removed from Octopus Deploy.

You can add this step template to an existing project, or create a new orchestration project.

- [Guide for Single Process](/docs/deployments/patterns/auto-scaling-deployment-targets/scale-out/scale-out-single-project.md)
- [Guide for Orchestration Project](/docs/deployments/patterns/auto-scaling-deployment-targets/scale-out/scale-out-orchestration-project.md)

:::hint
We only recommend this option when each scale out event adds more than five virtual machines and each deployment takes more than five minutes.  If you are adding 20 virtual machines each morning and it takes 15 minutes to finish configuring and deploying to each one, this is the option for you.
:::

## Use Octopus Deploy Runbooks to scale out

The final option is not use triggers and instead create an [Octopus Deploy Runbook](/docs/runbooks/index.md) to manually scale out.  The runbook process will be:

1. Run a CLI command on the cloud provider to scale out to a set number of machines.
2. Use the new step templates to wait until the scale out event finishes.
3. Use the Octopus CLI or the [Deploy Child Octopus Deploy Project](https://library.octopus.com/step-templates/0dac2fe6-91d5-4c05-bdfb-1b97adf1e12e/actiontemplate-deploy-child-octopus-deploy-project) step template to redeploy the latest release to the new virtual machines.

[Guide for Scale Out runbook](/docs/deployments/patterns/auto-scaling-deployment-targets/scale-out/scale-out-runbook.md)

This option has the following benefits:
- No unpredictable trigger behavior, the runbook is the one responsible for triggering a deployment to the new virtual machines.
- You can empower others to scale out virtual machines without providing them credentials to your cloud provider.
- Improved visibility as to the status of a scale out event.  Anyone can check the logs without having to login to the cloud provider.

:::hint
We recommend this option if you plan on only scaling out and scaling in via a schedule.  If you choose this option, it will be harder to manage metric based scale out events as you'll have two tools - Octopus Deploy and the Cloud Provider - handling a scale out event.  
:::