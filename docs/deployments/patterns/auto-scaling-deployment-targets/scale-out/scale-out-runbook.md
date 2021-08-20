---
title: Scale out using a runbook
description: How to configure a runbook to manually trigger a scale out event.
position: 40
hideInThisSection: true
---

This guide will create and configure a runbook to manually scale out (and in) virtual machines in an auto-scaling group.  It will use one of the new check provision status step templates to wait for the provisioning to finish as well as reconcile the list of virtual machines with Octopus Deploy.  

:::hint
Use a runbook to handle scaling when you plan on scaling out and in on a schedule or you want to give that ability to your users without giving direct access to your cloud provider.  Do not use this option if you plan on scaling out via a metric, such as CPU usage, or memory usage.
:::

# Project Configuration

Unlike other options, the only additional configuration you need to do is add in project variables for the check provision step templates to use.  For example, if you were using the [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status-(deployment-targets)) you'd want to configure.

- Project.Azure.Account: Variable for the Azure Account used to invoke the Azure PowerShell commands
- Project.CanContinue.Value: `#{unless Octopus.Deployment.Error}#{Octopus.Action[Check VMSS Provision Status].Output.VMSSHasServersToDeployTo}#{/unless}` (The output variable indicating there are new deployment targets to deploy to)
- Project.Machine.Ids: `#{Octopus.Action[Check VMSS Provision Status].Output.VMSSDeploymentTargetIds}` (The output variable containing all the deployment targets to deploy to)
- Project.Octopus.Api.Key: API Key of a service account that has Environment Manager, Project Viewer, and Deployment Creator roles assigned to it.
- Project.Octopus.Roles: The list of roles, for example `azure-todo-web-server,windows-web-server,vmss-scale-set-todo-web` that uniquely identifies a set of deployment targets in Octopus assigned to the VMSS (the more roles the lower the chance of a false positive).
- Project.Octopus.Server.Url: `#{Server.Base.Uri}`
- - Project.VMSS.NumberOfVms: Prompted variable with a default value of 2.
- Project.VMSS.ResourceGroup.Name: The name of the resource group the VMSS is assigned to.
- Project.VMSS.ScaleSetName: The name of the Virtual Machine Scale Set.

# Runbook

First, create a new runbook in the desired project.  Next, you will need to configure the runbook process to have, at a minimum, these three steps.

1. A script step to invoke a CLI command to set the number of virtual machines in the auto scaling group.
2. Check provision status step template to wait until provisioning is done and reconciles list of virtual machines with Octopus Deploy.
3. A script step to invoke either the Octopus CLI or the [Deploy Child Octopus Deploy Project step template](https://library.octopus.com/step-templates/0dac2fe6-91d5-4c05-bdfb-1b97adf1e12e/actiontemplate-deploy-child-octopus-deploy-project).

## Script Step To Update Auto Scaling Group

The script step to update the auto scaling group is not complex.  Provide the details of the auto scaling group and the desired number of virtual machines.

### Azure

This step uses the Azure Az PowerShell module.  

```PowerShell
$vmssScaleSetName = $OctopusParameters["Project.VMSS.ScaleSetName"]
$vmssScaleSetResourceGroup = $OctopusParameters["Project.VMSS.ResourceGroup.Name"]
$vmssScaleSetInstanceCount = $OctopusParameters["Project.VMSS.NumberOfVms"]

Write-Host "Attempting to get the VMSS"
try
{
	$vmssInfo = Get-AzVmss -ResourceGroupName $vmssScaleSetResourceGroup -VMScaleSetName $vmssScaleSetName
    Write-Host "VMSS was successfully found, moving on to changing the capacity"
}
catch
{
	Write-Highlight "Unable to access the scale set $vmssScaleSetName.  Exiting step."
	Write-Host $_.Exception
	exit 0
}

Write-Highlight "Updating $vmssScaleSetName capacity to $vmssScaleSetInstanceCount"
Update-AzVmss -ResourceGroupName $vmssScaleSetResourceGroup -VMScaleSetName $vmssScaleSetName -SkuCapacity $vmssScaleSetInstanceCount
Write-Highlight "Successfully updated $vmssScaleSetName capacity to $vmssScaleSetInstanceCount"
```

## Check Provision Status

The next step will use the [Check VMSS Provsion Status step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status-(deployment-targets)).  Provide values for the following parameters (leave all the others as is):

- VMSS Name: `#{Project.VMSS.ScaleSetName}`
- VMSS Resource Group Name: `#{Project.VMSS.ResourceGroup.Name}`
- Azure Account: Project.Azure.Account
- Deployment Target Roles: `#{Project.Octopus.Roles}`
- Octopus Url: `#{Project.Octopus.Server.Url}`
- Octopus API Key: `#{Project.Octopus.Api.Key}`
- Exclude Pre-Existing Servers from Output: Yes

## Trigger a deployment

The final step will use the [Deploy Child Octopus Deploy Project step template](https://library.octopus.com/step-templates/0dac2fe6-91d5-4c05-bdfb-1b97adf1e12e/actiontemplate-deploy-child-octopus-deploy-project).  Provide values for the following parameters (leave all the others as is):

- Octopus Base Url: `#{Project.Octopus.Server.Url}`
- Octopus API Key: `#{Project.Octopus.Api.Key}` 
- Child Project Name: The name of the project you wish to deploy
- Deployment Mode: Redeploy
- Specific Deployment Target: `#{Project.Machines.Ids}`
- Wait for Deployment: No

The number of virtual machines is a prompted variable, meaning it could scale out or scale in.  The final step in the process should only run when there are new virtual machines.  Set the run condition to the variable `#{Project.CanContinue.Value}`