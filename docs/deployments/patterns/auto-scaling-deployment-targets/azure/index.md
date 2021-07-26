---
title: Azure Virtual Machine Scale Set Deployment Targets
description: How to use Octopus Deploy with Azure Virtual Machine Scale Sets containing deployment targets.
position: 10
hideInThisSection: true
---

[Azure Virtual Machine Scale Sets](https://azure.microsoft.com/en-us/services/virtual-machine-scale-sets/) (VMSS) are an auto-scaling technology you can use with Octopus Deploy.  Unlike technologies such as [Azure Web Apps](https://azure.microsoft.com/en-us/services/app-service/web/) or [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/#overview), each instance in a VMSS has to be registered with Octopus Deploy.  This guide will walk you through how to use VMSS with Octopus Deploy, some common pitfalls, as well as a [step template](https://library.octopus.com/step-templates/e04c5cd8-0982-44b8-9cae-0a4b43676adc/actiontemplate-check-vmss-provision-status) in the library to make it easier to manage scale out and scale in events.

# Design considerations

When you create your first Virtual Machine Scale Set you will be presented with a number of small decisions.  We have a general [recommendation guide](/docs/deployments/patterns/auto-scaling-deployment-targets/auto-scaling-recommendations.md) that cover:

- Auto-scaling or manual scaling
- Using a provided image or building your own
- How to handle scale-in events

Those recommendations are applicable to any auto-scaling technology.  Below are recommendations specific to Azure Virtual Machine Scale Sets.

## Registering Tentacles from VMs in a Virtual Machine Scale Set

A tentacle must be installed and configured in order to be able to deploy/run runbooks on VMs managed by a Virtual Machine Scale Set.  You have two options with Virtual Machine Scale Sets, a custom image or bootstrap script.  No matter which option you choose, you should become familar with how to automate tentacle installation.

- [Windows](/docs/infrastructure/deployment-targets/windows-targets/automating-tentacle-installation.md)
- [Linux](/docs/infrastructure/deployment-targets/linux/tentacle/index.md#automation-scripts)

### Include the tentacle in a custom image

VMSS allow you to create VMs from custom images.  A custom image has all the necessary software (IIS, Java, NGINX, etc.) pre-installed.  If you going down the route of a custom image, we'd recommend adding the Octopus Tentacle to that list.

:::warning
It is common to see over a dozen new versions of the Octopus Tentacle in a single quarter for bug fixes, performance improvements and security fixes.  You will need a process to keep the image up to date.
:::

### Using a custom script extension

Azure provides an extension to let you run a custom script on machine startup.  

- [Windows Custom Script Extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows)
- [Linux Custom Script Extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-linux)

Scripts should be written as idempotent.  The script will run each time a machine starts up.  For example, attempting to install the same tentacle instance multiple times will cause it to fail.  We have sample scripts you can use as a basis for your scripts.

- [Listening Tentacles](https://github.com/OctopusSamples/IaC/blob/master/bootstrap/Windows/BootstrapListeningTentacle.ps1)
- [Polling Tentacles](https://github.com/OctopusSamples/IaC/blob/master/bootstrap/Windows/BootstrapPollingTentacle.ps1)

## Overprovisioning

Azure Virtual Machine Scale Sets provide a feature called [overprovisioning](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview#overprovisioning).  In a nutshell, if you scale out to 5 VMs, 8 VMs will be created and the first 5 that are successfully provisioned will be kept, and the remaining 3 will be deleted.  Overprovisioning is designed to help scale out success rates.

Because of the bootstrap script, it is common to see all 8 VMs registed in Octopus Deploy.  This is normal behavior.  Please see our page on [reconciling the VMSS with Octopus Deploy](/docs/deployments/patterns/auto-scaling-deployment-targets/azure/reconcile-vmss-with-octopus.md).

:::hint
Overprovisioning is turned on by default when you create a Virtual Machine Scale Set.
:::

## Project Structure


