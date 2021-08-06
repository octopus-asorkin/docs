---
title: Auto Registering New Virtual Machines with Octopus Deploy
description: Recommendations on how to auto-register new virtual machines as Deployment Targets in Octopus Deploy.
position: 30
hideInThisSection: true
---

Most auto-scaling technologies give the option to use provided OS images or a custom built image for the servers in the auto scaling group.  No matter which option you choose, you will need to bootstrap the install of the tentacle.  This could be done as a script using a [custom script extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows) for VMSS or included as part of the custom image.

Please see our documentation on automating tentacle installations:

- [Windows](/docs/infrastructure/deployment-targets/windows-targets/automating-tentacle-installation.md)
- [Linux](/docs/infrastructure/deployment-targets/linux/tentacle/index.md#automation-scripts)

For example scripts, please see our [Infrastructure as Code Sample GitHub Repo](https://github.com/OctopusSamples/IaC/tree/master/bootstrap).

:::hint
Unlike static servers, which are treated more like pets where you handcraft and maintain them, servers created via auto-scaling should be treated as cattle.  At any given point a server might go down and you'll need to replace it.  Assume whatever could go wrong can go wrong.  For the bootstrap script assume:

1. It will be run multiple times (perhaps after each restart).
2. You can't assume the machine registration will be in Octopus Deploy, so force the registration.
3. Use the hostname to uniquely identify the machine in Octopus to avoid duplication.
:::