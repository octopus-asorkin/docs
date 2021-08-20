---
title: Scale In Event
description: Recommendations how to handle a scale in event from an auto scaling group in Octopus Deploy.
position: 60
hideInThisSection: true
---

A scale in event is when virtual machines are removed from an auto-scaling group.  Just like a scale out event, a scale in event can be triggered automatically via a configured rule, or manually by a person.  When a scale in event occurs, the removed virtual machines will need to be removed from Octopus Deploy.

A scale in event will cause one of these scenarios.

- A server is registered with Octopus Deploy but no longer exists in the auto scaling group.  
- A server is deleted (manually or via a rule) in the middle of a deployment or runbook run.

# Guided Failure

A deployment will fail when a deployment target is removed while running a step on it.  That will happen during a scale-in event.  The chances of a scale-in event occuring during a deployment while deploying to a specific machine is small, but it is better to be prepared.  The recommendation is to configure [guided failure mode](docs/releases/guided-failures.md).  Any errors will pause the deployment and ask for user intervention.  From there you can exclude the deployment target or stop the deployment.

Guided failure mode can be configured at the environment level.
![guided failure environment](images/environment-guided-failure-mode.png)

Or, at the project level.
![project guided failure mode](images/project-guided-failure-mode.png)

# Processing Scale In Events

As mentioned earlier, a scale-in event results in Octopus Deploy having a deployment target registered that no longer exists.  A scheduled job will need to reconcile the list of virtual machines in the auto-scaling group with Octopus Deploy.  There are two options for scheduled jobs.

1. Machine Policy
2. Reconcilation Runbook

## Machine Policy

The recommendation is to create a separate machine policy for virtual machines managed by auto scaling groups.  If you followed that recommendation, then update the following settings in the machine policy:

- Health Check Schedule: Once every 15 minutes
- Health Check Type: Connection only test
- Clean up Unavailable Deployment Targets: Automatically delete unavailable machines after 20 minutes

## Reconcilation Runbook

A reconcilation runbook will use one of the new check provision status step templates, such as [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status-(deployment-targets)).  The step templates will:

- Wait for the auto-scaling group to finish provisioning (in the unlikely event a scale-out/scale-in event starts at the same time as the runbook).
- Reconcile the list of virtual machines in the auto-scaling group with Octopus Deploy.  Any virtual machines not in the auto-scaling group will be removed from Octopus Deploy.

To create this process, first you'll need to create some variables.  This guide assumes you are using an Azure Virtual Machine Scale Set; the same core concepts apply for any auto-scaling technology.

- Project.Azure.Account: Variable for the Azure Account used to invoke the Azure PowerShell commands
- Project.Octopus.Api.Key: API Key of a service account that has Environment Manager, Project Viewer, and Deployment Creator roles assigned to it.
- Project.Octopus.Roles: The list of roles, for example `azure-todo-web-server,windows-web-server,vmss-scale-set-todo-web` that uniquely identifies a set of deployment targets in Octopus assigned to the VMSS (the more roles the lower the chance of a false positive).
- Project.Octopus.Server.Url: `#{Server.Base.Uri}`
- Project.VMSS.ResourceGroup.Name: The name of the resource group the VMSS is assigned to.
- Project.VMSS.ScaleSetName: The name of the Virtual Machine Scale Set.

Then create a runbook and add the [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status-(deployment-targets)) step template.  

Set the following values:

- VMSS Name: `#{Project.VMSS.ScaleSetName}`
- VMSS Resource Group Name: `#{Project.VMSS.ResourceGroup.Name}`
- Azure Account: Project.Azure.Account
- Deployment Target Roles: `#{Project.Octopus.Roles}`
- Octopus Url: `#{Project.Octopus.Server.Url}`
- Octopus API Key: `#{Project.Octopus.Api.Key}`

:::hint
We recommend this option, as a deployment target could accidentally removed via the machine policy if it was shut down or the tentacle stopped responding.  This runbook process only compares the list of virtual machines with what is in Octopus Deploy.
:::