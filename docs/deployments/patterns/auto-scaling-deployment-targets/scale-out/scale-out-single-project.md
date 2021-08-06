---
title: Scale out using a single project
description: How to add the necessary steps to an existing deployment process to handle a scale out event.
position: 20
hideInThisSection: true
---

This option will add the [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status) into your existing project.  Once that finishes running, you can run a [health check](/docs/projects/built-in-step-templates/health-check.md) to add the new machines.

Pros: All the new VMs in the VMSS are deployed to at the same time.  
Cons: You will need to change an existing deployment process.  Preventing existing machines from getting deployed to will require a variable run condition.

Recommendations: Use this option if you plan on scaling out by more than 5 VMs at a time.

### Update Project Configuration

You will want to ensure your project can handle scale out and scale in events.

1. On your Octopus instance click on **{{Project, Add Project}}**.
2. Enter in a **Project Name** and click **Save**
3. From the project overview screen go to **{{Deployments, Settings}}**.
4. Configure the following settings:
    - Deployment Targets Required: Change to `Allow deployments to be created when there are no deployment targets`.
    - Transient Deployment Targets - Unavailable Deployment Targets: Change to `Skip and Continue`.  Add the role assigned to VMs in the virtual machine scale set.
    - Transient Deployment Targets - Unhealthy Deployment Targets: Change to `Exclude`.
5. Click **SAVE** to save the changes.

![Orchestration Project Deployment Settings](images/orchestration-project-deployment-settings.png)

These settings will enable you to create a release the deployment target triggers to use to on a scale out event.

### Configure Deployment Target Triggers

Next, configure deployment target triggers to trigger a deployment anytime a new deployment target is found.  

1. From the project overview screen go to **{{Deployments, Triggers}}**.
2. In the top right cornger click on **{{Add Trigger, Deployment target trigger}}**.
3. Configure the following settings:
    - Name: Provide a name for the trigger.
    - Event Categories: select `Machine Created`.
    - Environments: enter the environments the VMSS exists in.
    - Target roles: select the target role of the VMs in the VMSS.
4. Click **SAVE** to save the new trigger.

![Orchestration Project Deployment Target Trigger](images/orchestration-project-deployment-target-trigger.png)

### Configure Variables

The [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status) has some parameters without defaults.  It will also set a couple of output variables.

Please add the following variables to your project.

- Project.Azure.Account: Variable for the Azure Account used to invoke the Azure PowerShell commands
- Project.CanContinue.Value: `#{unless Octopus.Deployment.Error}#{Octopus.Action[Check VMSS Provision Status].Output.VMSSHasServersToDeployTo}#{/unless}` (The output variable indicating there are new deployment targets to deploy to)
- Project.Machine.Ids: `#{Octopus.Action[Check VMSS Provision Status].Output.VMSSDeploymentTargetIds}` (The output variable containing all the deployment targets to deploy to)
- Project.Octopus.Api.Key: API Key of a service account that has Environment Manager, Project Viewer, and Deployment Creator roles assigned to it.
- Project.Octopus.Roles: The list of roles, for example `azure-todo-web-server,windows-web-server,vmss-scale-set-todo-web` that uniquely identifies a set of deployment targets in Octopus assigned to the VMSS (the more roles the lower the chance of a false positive).
- Project.Octopus.Server.Url: `#{Server.Base.Uri}`
- Project.VMSS.ResourceGroup.Name: The name of the resource group the VMSS is assigned to.
- Project.VMSS.ScaleSetName: The name of the Virtual Machine Scale Set.

![Orchestration Project Variables](images/release-orchestration-variables.png)